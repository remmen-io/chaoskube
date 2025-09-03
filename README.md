# chaoskube
[![GitHub release](https://img.shields.io/github/release/remmen-io/chaoskube.svg)](https://github.com/remmen-io/chaoskube/releases)
[![go-doc](https://godoc.org/github.com/remmen-io/chaoskube/chaoskube?status.svg)](https://godoc.org/github.com/remmen-io/chaoskube/chaoskube)

`chaoskube` periodically kills random pods in your Kubernetes cluster.

<p align="center"><img src="chaoskube.png" width="40%" align="center" alt="chaoskube"></p>

## About This Fork

Enhanced fork of [linki/chaoskube](https://github.com/linki/chaoskube) with:
- **Dynamic intervals** - automatically adjusts timing based on cluster size
- **Static pod protection** - automatically excludes static/mirror pods
- **Updated dependencies** - latest security patches

## Why

Test how your system behaves under arbitrary pod failures. Chaoskube randomly kills pods to help you build resilient applications.

## Main Usage

```console
# Start in safe mode (shows what would be killed)
$ chaoskube --dry-run

# Enable actual pod termination every 10 minutes
$ chaoskube --no-dry-run --interval=10m

# Target only specific namespaces
$ chaoskube --no-dry-run --namespaces='staging,testing'

# Use dynamic intervals that scale with cluster size
$ chaoskube --no-dry-run --dynamic-interval --dynamic-factor=1.5
```

## New Features

### Dynamic Intervals

The dynamic interval feature automatically adjusts the time between pod terminations based on the number of candidate pods in your cluster. This helps ensure appropriate chaos levels in both small and large environments.

**How it works:**
With dynamic interval enabled, chaoskube calculates the interval between pod terminations using:

```
interval = totalWorkingMinutes / (podCount × factor)
```

Where:
- `totalWorkingMinutes` = 10 days × 8 hours × 60 minutes = 4800 minutes (assumes all pods should be killed during 2 work weeks)
- `factor` is the configurable dynamic interval factor

The dynamic interval factor lets you control the aggressiveness of the terminations:
- With `factor = 1.0`: Standard interval calculation
- With `factor > 1.0`: More aggressive terminations (shorter intervals)
- With `factor < 1.0`: Less aggressive terminations (longer intervals)

**Example scenarios:**
- Small cluster (100 pods, factor 1.0): interval = 48 minutes
- Small cluster (100 pods, factor 1.5): interval = 32 minutes
- Small cluster (100 pods, factor 2.0): interval = 24 minutes
- Large cluster (1500 pods, factor 1.0): interval = 3.2 minutes

**Usage:**
```console
$ chaoskube --dynamic-interval --dynamic-factor=1.5 --no-dry-run
```

### Static Pod Protection

Chaoskube automatically excludes static pods (mirror pods) from being terminated. Static pods are managed directly by the kubelet on a node rather than by the API server, and they are identified by the presence of the `kubernetes.io/config.mirror` annotation. This protection ensures that critical system components remain stable during chaos testing.

## Quick Start

**Helm:**
```console
$ helm repo add chaoskube-dynamic https://remmen-io.github.io/chaoskube/
$ helm install chaoskube chaoskube-dynamic/chaoskube --create-namespace -n chaoskube
```

**Basic usage:**
```console
$ chaoskube --dry-run  # Safe mode - shows what would be killed
$ chaoskube --no-dry-run --interval=5m  # Kill every 5 minutes
```

## Configuration

### Key Flags
| Flag | Description | Default |
|------|-------------|---------|
| `--dry-run` | Don't kill pods, just log | `true` |
| `--interval` | Time between kills | `10m` |
| `--dynamic-interval` | Enable smart scaling | `false` |
| `--dynamic-factor` | Aggressiveness multiplier | `1.0` |
| `--namespaces` | Target namespaces | all |
| `--labels` | Label selector | all |

**Note:** Static pods (mirror pods) are automatically excluded from termination regardless of filters.

### Filtering Examples
```console
# Only kill in specific namespaces
$ chaoskube --namespaces 'staging,testing'

# Only kill pods with specific labels
$ chaoskube --labels 'app=myapp,env!=prod'

# Exclude system pods
$ chaoskube --namespaces '!kube-system'
```

### Time Restrictions
```console
# Skip weekends and nights
$ chaoskube --excluded-weekdays=Sat,Sun --excluded-times-of-day=22:00-08:00
```

## Health Check

Chaoskube exposes a health endpoint on port 8080 for liveness probes.

## Contributing

Issues and pull requests welcome! This fork maintains compatibility with the original while adding intelligent scaling.

## Acknowledgments

This project is built upon the excellent [chaoskube](https://github.com/linki/chaoskube) project by [@linki](https://github.com/linki) and its contributors. We're grateful for their foundational work that made this enhanced version possible.
