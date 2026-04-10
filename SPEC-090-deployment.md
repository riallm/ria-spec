# SPEC-090: Deployment Specification

**Version**: 1.0  
**Status**: Approved  
**Last Updated**: 2025-04-09  
**Category**: Deployment

## 1. Overview

This specification defines deployment procedures for RIA models via riallm.

## 2. Deployment Targets

### 2.1 Supported Environments

| Environment | Use Case | Model Tiers |
|------------|----------|-------------|
| **Local workstation** | Development | 1B, 8B |
| **Single GPU server** | Team usage | 8B, 64B |
| **Multi-GPU cluster** | Enterprise | 64B, 128B |
| **Cloud VM** | Scalable | All tiers |
| **Edge device** | Mobile/offline | 1B |

### 2.2 Deployment Modes

| Mode | Description | Latency | Throughput |
|------|-------------|---------|-----------|
| **Interactive** | Single user, low latency | <100ms | 1 req/s |
| **Batch** | Multiple requests queued | <1s | 10 req/s |
| **Streaming** | Token-by-token response | <50ms first | Continuous |
| **Offline** | No network required | Varies | Varies |

## 3. Local Deployment

### 3.1 Quick Start

```bash
# Install riallm
cargo install riallm

# Download model
riallm download riallm/ria-8b-q4_k_m.gguf

# Start server
riallm serve --model ria-8b-q4_k_m.gguf --port 8080
```

### 3.2 System Requirements

| Component | Minimum | Recommended |
|----------|---------|-------------|
| **CPU** | 4 cores | 8+ cores |
| **RAM** | 16 GB | 32 GB |
| **Storage** | 10 GB SSD | 50 GB NVMe |
| **GPU** | 8 GB VRAM | 12+ GB VRAM |
| **OS** | Ubuntu 20.04 | Ubuntu 22.04 |

## 4. Server Deployment

### 4.1 Docker Deployment

```dockerfile
FROM rust:1.80 as builder
WORKDIR /app
COPY . .
RUN cargo build --release

FROM ubuntu:22.04
COPY --from=builder /app/target/release/riallm /usr/local/bin/
COPY model.gguf /models/model.gguf

EXPOSE 8080
CMD ["riallm", "serve", "--model", "/models/model.gguf", "--port", "8080"]
```

### 4.2 Docker Compose

```yaml
version: '3.8'

services:
  riallm:
    build: .
    ports:
      - "8080:8080"
    volumes:
      - ./models:/models
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: 1
              capabilities: [gpu]
    environment:
      - RIALLM_DEVICE=cuda
      - RIALLM_CACHE_SIZE=4GB
```

### 4.3 Kubernetes Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: riallm
spec:
  replicas: 3
  selector:
    matchLabels:
      app: riallm
  template:
    metadata:
      labels:
        app: riallm
    spec:
      containers:
      - name: riallm
        image: riallm/riallm:latest
        ports:
        - containerPort: 8080
        resources:
          limits:
            nvidia.com/gpu: 1
            memory: 32Gi
          requests:
            nvidia.com/gpu: 1
            memory: 16Gi
        env:
        - name: RIALLM_DEVICE
          value: "cuda"
```

## 5. Cloud Deployment

### 5.1 AWS Deployment

```bash
# Launch EC2 instance with GPU
aws ec2 run-instances \
  --image-id ami-0abcdef1234567890 \
  --instance-type g5.2xlarge \
  --key-name my-key \
  --security-group-ids sg-12345678 \
  --user-data file://startup.sh

# Install and start riallm
cat startup.sh
#!/bin/bash
curl -sSL https://riallm.org/install.sh | bash
riallm download riallm/ria-8b-q4_k_m.gguf
riallm serve --model ria-8b-q4_k_m.gguf --host 0.0.0.0
```

### 5.2 Instance Recommendations

| Provider | Instance | GPU | vRAM | Hourly Cost |
|----------|---------|-----|------|-------------|
| **AWS** | g5.2xlarge | A10G | 24GB | $1.21 |
| **GCP** | a2-highgpu-1g | A100 | 40GB | $3.69 |
| **Azure** | NC6s v3 | V100 | 16GB | $3.06 |
| **Lambda** | gpu_1x_a100 | A100 | 80GB | $1.10 |

## 6. Configuration Management

### 6.1 Environment-Specific Configs

```yaml
# development.yaml
model:
  path: "ria-8b-q4_k_m.gguf"
  device: "auto"
  
server:
  port: 8080
  workers: 1
  
logging:
  level: "debug"

# production.yaml
model:
  path: "ria-64b-q4_k_m.gguf"
  device: "cuda:0"
  
server:
  port: 8080
  workers: 4
  
logging:
  level: "info"
```

### 6.2 Secrets Management

```bash
# Using environment variables
export RIALM_API_KEY="your-secret-key"

# Using secret file
echo "your-secret-key" > /run/secrets/api_key

# Using Kubernetes secrets
kubectl create secret generic riallm-api-key --from-literal=key=your-secret-key
```

## 7. Monitoring

### 7.1 Health Checks

```yaml
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost:8080/health"]
  interval: 30s
  timeout: 10s
  retries: 3
  start_period: 40s
```

### 7.2 Metrics

| Metric | Type | Description |
|--------|------|-------------|
| `request_count` | Counter | Total requests |
| `request_duration_seconds` | Histogram | Request latency |
| `tokens_generated_total` | Counter | Total tokens |
| `gpu_memory_usage_bytes` | Gauge | GPU memory |
| `model_load_duration_seconds` | Histogram | Model loading time |

### 7.3 Prometheus Configuration

```yaml
scrape_configs:
  - job_name: 'riallm'
    static_configs:
      - targets: ['riallm:8080']
    metrics_path: '/metrics'
```

## 8. Scaling

### 8.1 Horizontal Scaling

| Strategy | Description | Use Case |
|----------|-------------|----------|
| **Load balancer** | Multiple instances | High availability |
| **Model parallel** | Split model across GPUs | Large models |
| **Data parallel** | Multiple models | High throughput |

### 8.2 Vertical Scaling

| Resource | Scale Up When | Action |
|----------|--------------|--------|
| **GPU memory** | >80% usage | Larger GPU |
| **RAM** | >90% usage | More RAM |
| **CPU** | >70% usage | More cores |
| **Disk I/O** | High latency | Faster disk |

## 9. Backup and Recovery

### 9.1 Backup Strategy

| Component | Frequency | Retention |
|----------|----------|-----------|
| **Model files** | On update | Keep last 3 |
| **Configuration** | On change | 90 days |
| **Logs** | Daily | 1 year |
| **Metrics** | Hourly aggregates | 1 year |

### 9.2 Disaster Recovery

| Scenario | Recovery Time | Procedure |
|----------|--------------|-----------|
| **Server failure** | <5 minutes | Auto-restart |
| **GPU failure** | <30 minutes | Failover to backup |
| **Data corruption** | <1 hour | Restore from backup |
| **Complete outage** | <4 hours | Rebuild from scratch |

## 10. References

- [SPEC-091: Infrastructure](./SPEC-091-infrastructure.md)
- [SPEC-092: Scaling](./SPEC-092-scaling.md)
- [SPEC-040: riallm Integration](./SPEC-040-riallm-integration.md)
