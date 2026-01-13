# Hytale Server Deployment

This directory contains Kubernetes manifests for deploying a Hytale server using Kustomize.

## Overview

Hytale is a blocky sandbox game developed by Hypixel Studios. This deployment uses the `proxyfil/docker-hytale-server` Docker image to run a dedicated Hytale server on Kubernetes.

## Prerequisites

- Kubernetes cluster (1.19+)
- kubectl configured
- PersistentVolume provisioner (or use hostPath volumes)
- Docker image: `proxyfil/docker-hytale-server:latest`

## Quick Start

### Deploy the Server

```bash
kubectl apply -k .
```

### View Logs & Authenticate

On first launch, you need to authenticate your server:

```bash
# Get pod name
kubectl get pods -n hytale-server

# View logs
kubectl logs -f <pod-name> -n hytale-server

# Follow the instructions to visit https://accounts.hytale.com/device
# and enter the provided code
```

### Check Status

```bash
# Check pod status
kubectl get pods -n hytale-server

# Check service
kubectl get svc -n hytale-server

# Get external access port
kubectl get svc hytale-server -n hytale-server
```

## Configuration

### Server Settings

The server is configured with the following default settings:

- **Memory**: 4GB initial, 8GB maximum
- **Port**: 5520 (UDP)
- **Auth Mode**: authenticated (requires Hytale account)
- **Backup**: Enabled (every 30 minutes)
- **Protocol**: QUIC over UDP

### Environment Variables

Key environment variables can be modified in [hytale-stateful.yaml](hytale-stateful.yaml):

| Variable | Default | Description |
|----------|---------|-------------|
| `JAVA_OPTS` | `-Xms4G -Xmx8G -XX:+UseG1GC -XX:MaxGCPauseMillis=200` | JVM memory allocation |
| `SERVER_PORT` | `5520` | UDP port for the server |
| `BIND_ADDRESS` | `0.0.0.0:5520` | Address to bind to |
| `AUTH_MODE` | `authenticated` | Authentication mode (authenticated/offline) |
| `ENABLE_BACKUP` | `true` | Enable automatic backups |
| `BACKUP_FREQUENCY` | `30` | Backup interval in minutes |
| `DISABLE_SENTRY` | `false` | Disable crash reporting |
| `ALLOW_OP` | `false` | Allow operator commands |

### Mods

The server includes the Nitrado Web Server mod (`nitrado-webserver-1.0.0.jar`). To add more mods:

1. Edit [mods.txt](mods.txt)
2. Add mod filenames or URLs (one per line)
3. Apply the changes:

```bash
kubectl apply -k .
kubectl rollout restart statefulset/hytale-server -n hytale-server
```

## Storage

The deployment uses hostPath volumes for data persistence:

- **Universe**: `/opt/hytale/universe` - Game worlds and player data
- **Mods**: `/opt/hytale/mods` - Server mods and plugins
- **Logs**: `/opt/hytale/logs` - Server logs
- **Cache**: `/opt/hytale/cache` - AOT cache and temporary files

> **Note**: For production, consider using PersistentVolumeClaims instead of hostPath volumes.

## Accessing the Server

### NodePort (Default)

The server is exposed via NodePort on port 30520:

```bash
# Get node IP
kubectl get nodes -o wide

# Connect to: <node-ip>:30520
```

### LoadBalancer (Cloud Environments)

To use a LoadBalancer instead of NodePort:

1. Edit [hytale-svc.yaml](hytale-svc.yaml)
2. Change `type: NodePort` to `type: LoadBalancer`
3. Remove the `nodePort` line
4. Apply changes:

```bash
kubectl apply -k .

# Get external IP
kubectl get svc hytale-server -n hytale-server
```

### Cloud-Specific Annotations

#### AWS (EKS)
```yaml
annotations:
  service.beta.kubernetes.io/aws-load-balancer-type: "nlb"
```

#### GCP (GKE)
```yaml
annotations:
  cloud.google.com/load-balancer-type: "External"
```

#### Azure (AKS)
```yaml
annotations:
  service.beta.kubernetes.io/azure-load-balancer-internal: "false"
```

## Operations

### View Server Logs

```bash
kubectl logs -f <pod-name> -n hytale-server
```

### Access Server Console

```bash
kubectl exec -it <pod-name> -n hytale-server -- /bin/bash
```

### Restart Server

```bash
kubectl rollout restart statefulset/hytale-server -n hytale-server
```

### Scale (Not Recommended)

Game servers typically run as single instances. For multiple servers, create separate deployments with different namespaces and ports.

## Backup & Recovery

### Manual Backup

```bash
# Backup universe data
kubectl cp <pod-name>:/hytale-server/universe ./universe-backup -n hytale-server

# Backup mods
kubectl cp <pod-name>:/hytale-server/mods ./mods-backup -n hytale-server
```

### Restore from Backup

```bash
# Restore universe
kubectl cp ./universe-backup <pod-name>:/hytale-server/universe/ -n hytale-server

# Restart pod
kubectl delete pod <pod-name> -n hytale-server
```

### Automated Backups

Automatic backups are enabled by default (every 30 minutes). Backups are stored within the container's volume.

## Troubleshooting

### Pod Won't Start

```bash
# Describe pod
kubectl describe pod <pod-name> -n hytale-server

# Check events
kubectl get events -n hytale-server --sort-by='.lastTimestamp'

# Check logs
kubectl logs <pod-name> -n hytale-server
```

### Authentication Issues

If you see authentication errors:

1. View the logs to get the device code
2. Visit https://accounts.hytale.com/device
3. Enter the code displayed in the logs

> **Note**: Limited to 100 servers per Hytale license. For more capacity, apply for a [Server Provider account](https://support.hytale.com/hc/en-us/articles/45328341414043).

### Service Not Accessible

```bash
# Check service
kubectl get svc hytale-server -n hytale-server
kubectl describe svc hytale-server -n hytale-server

# Check endpoints
kubectl get endpoints hytale-server -n hytale-server

# Test from within cluster (if using ClusterIP)
kubectl run -it --rm debug --image=busybox --restart=Never -n hytale-server -- sh
# nc -u hytale-server 5520
```

### Memory Issues

If you see OutOfMemoryError:

1. Increase `JAVA_OPTS` in [hytale-stateful.yaml](hytale-stateful.yaml)
2. Adjust resource limits
3. Apply changes and restart:

```bash
kubectl apply -k .
kubectl rollout restart statefulset/hytale-server -n hytale-server
```

## Performance Tuning

### For Small Servers (1-5 players)

```yaml
env:
  - name: JAVA_OPTS
    value: "-Xms1G -Xmx2G"
resources:
  limits:
    memory: "2Gi"
  requests:
    memory: "1Gi"
```

### For Large Servers (50+ players)

```yaml
env:
  - name: JAVA_OPTS
    value: "-Xms8G -Xmx16G -XX:+UseG1GC -XX:MaxGCPauseMillis=200 -XX:+ParallelRefProcEnabled"
resources:
  limits:
    memory: "16Gi"
  requests:
    memory: "8Gi"
```

## Uninstall

```bash
# Delete all resources
kubectl delete -k .

# Optionally delete data (WARNING: deletes all game data)
# Manually remove hostPath directories on nodes:
# rm -rf /opt/hytale/*
```

## System Requirements

| Resource | Minimum | Recommended |
|----------|---------|-------------|
| Java | 25 | 25 |
| RAM | 4GB | 8GB+ |
| CPU | 2 cores | 4+ cores |
| Storage | 10GB | 50GB+ |
| Protocol | UDP | UDP (QUIC) |

## Resources

- [Official Repository](https://github.com/Proxyfil/docker-hytale-server)
- [Docker Hub](https://hub.docker.com/r/proxyfil/docker-hytale-server)
- [Hytale Official Server Manual](https://support.hytale.com/hc/en-us/articles/45326769420827)
- [Hytale Support](https://support.hytale.com/)
- [Server Provider Account Application](https://support.hytale.com/hc/en-us/articles/45328341414043)

## Notes

- Hytale is currently in early access - server stability may vary
- The server uses QUIC protocol over UDP (not TCP)
- Authentication is required on first launch
- Limited to 100 servers per license
- Runs as non-root user for security

## License

This deployment configuration is based on the [proxyfil/docker-hytale-server](https://github.com/Proxyfil/docker-hytale-server) project.

> **Note**: Hytale and all related trademarks are properties of Hypixel Studios Canada Inc. This is an unofficial community deployment.
