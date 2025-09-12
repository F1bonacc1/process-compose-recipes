# NATS Cluster Recipe 🚀

A production-ready NATS cluster setup with JetStream support and built-in monitoring using native NATS server binaries.

## Overview

This recipe creates a 3-node NATS cluster with:
- **JetStream** enabled for stream processing and message persistence
- **Clustering** with automatic discovery and mesh topology
- **Health monitoring** with HTTP health checks
- **Real-time monitoring** using `nats-top`
- **Persistent storage** for each node's data

## Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   NATS Node 0   │    │   NATS Node 1   │    │   NATS Node 2   │
│                 │    │                 │    │                 │
│ Client: :4220   │◄──►│ Client: :4221   │◄──►│ Client: :4222   │
│ Monitor: :8220  │    │ Monitor: :8221  │    │ Monitor: :8222  │
│ Cluster: :6220  │    │ Cluster: :6221  │    │ Cluster: :6222  │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         ▲                       ▲                       ▲
         └───────────────────────┼───────────────────────┘
                                 │
                    ┌─────────────────┐
                    │   nats-top      │
                    │   (monitoring)  │
                    └─────────────────┘
```

## Prerequisites

### Required Software
- **NATS Server**: Install from [NATS.io downloads](https://nats.io/download/)
- **NATS CLI** (optional but recommended): For testing and monitoring
- **Process-Compose**: v1.73.0 or later

## Usage

### Start the Cluster
```bash
process-compose -f <recipe-path>/process-compose.yaml
```

### Verify Installation
The cluster will automatically:
1. **Sanity check** - Verify NATS server is installed
2. **Prepare directories** - Create necessary folder structure
3. **Start 3 nodes** - Launch clustered NATS servers
4. **Health monitoring** - Continuous health checks
5. **Display monitoring** - Real-time cluster stats via `nats-top`

### Connection Details

| Node | Client Port | Monitor Port | Cluster Port |
|------|-------------|--------------|--------------|
| N0   | 4220        | 8220         | 6220         |
| N1   | 4221        | 8221         | 6221         |
| N2   | 4222        | 8222         | 6222         |

## Testing the Cluster

### Basic Pub/Sub
```bash
# Subscribe to messages (terminal 1)
nats sub test.topic --server nats://127.0.0.1:4220

# Publish messages (terminal 2)
nats pub test.topic 'Hello NATS Cluster!' --server nats://127.0.0.1:4221
```

### JetStream Examples
```bash
# Create a stream
nats stream add ORDERS --subjects "orders.*" --storage file --server nats://127.0.0.1:4220

# Create consumer
nats consumer add ORDERS PROCESSOR --pull --deliver all --server nats://127.0.0.1:4220

# Add messages to stream
nats pub orders.new "Order #1234" --server nats://127.0.0.1:4221
nats pub orders.processed "Order #1234 processed" --server nats://127.0.0.1:4222

# Consume messages
nats consumer next ORDERS PROCESSOR --server nats://127.0.0.1:4220
```

### Cluster Status
```bash
# Check cluster status from any node
curl http://127.0.0.1:8220/routez

# Detailed server info
curl http://127.0.0.1:8221/varz

# JetStream info
curl http://127.0.0.1:8222/jsz
```

## Monitoring

### Real-time Monitoring
The recipe includes `nats-top` for real-time cluster monitoring:
- **Connection counts** per node
- **Message rates** (in/out)
- **JetStream statistics**
- **Cluster topology** visualization

### Health Checks
Each node includes HTTP health checks:
- **Endpoint**: `/healthz?js-server-only=true`
- **Frequency**: Every 10 seconds
- **Timeout**: 3 seconds
- **Failure threshold**: 3 consecutive failures

### Log Monitoring
Process-Compose logs are tailed in real-time:
```bash
# Logs are displayed via pc_log process
tail -f /tmp/process-compose-${USER}.log
```

## Configuration

### Directory Structure
```
nats-cluster/
data/
├── n0/          # Node 0 JetStream data
├── n1/          # Node 1 JetStream data
└── n2/          # Node 2 JetStream data

```

### Node Configuration
Each node is configured with:
- **Name**: N0, N1, N2
- **JetStream**: Enabled with file storage
- **Cluster**: Full mesh connectivity
- **Routes**: All nodes know about each other
- **Monitoring**: HTTP monitoring enabled

### Port Allocation
Ports are dynamically assigned using replica numbers:
- **Client**: `422 + replica_num`
- **Monitor**: `822 + replica_num`  
- **Cluster**: `622 + replica_num`

## Customization

### Scaling
To change the number of nodes, modify the `replicas` value:
```yaml
nats-cluster-node:
  replicas: 5  # Scale to 5 nodes
```

### Storage Configuration
JetStream data is stored in `./data/n${PC_REPLICA_NUM}`. To change storage location:
```yaml
nats-cluster-node:
  command: >
    nats-server --name N$${PC_REPLICA_NUM} --js --sd /custom/path/n$${PC_REPLICA_NUM}
    # ... rest of command
```

### Authentication
Add authentication by modifying the server command:
```yaml
nats-cluster-node:
  command: >
    nats-server --name N$${PC_REPLICA_NUM} --js --sd ./data/n$${PC_REPLICA_NUM}
    --user myuser --pass mypassword
    # ... rest of command
```

### TLS Configuration
Enable TLS by adding certificate options:
```yaml
nats-cluster-node:
  command: >
    nats-server --name N$${PC_REPLICA_NUM} --js --sd ./data/n$${PC_REPLICA_NUM}
    --tls --tlscert server-cert.pem --tlskey server-key.pem
    # ... rest of command
```

## Troubleshooting

### Common Issues

#### 1. "nats-server: command not found"
**Solution**: Install NATS server binary

#### 2. Port conflicts
**Error**: "bind: address already in use"
**Solution**: Check for running NATS instances
```bash
# Kill existing NATS processes
pkill nats-server

# Or change ports in the recipe
```

#### 3. Cluster formation issues
**Symptoms**: Nodes not connecting to each other
**Debug**:
```bash
# Check cluster routes
curl http://127.0.0.1:8220/routez | jq '.routes'
```


#### 4. JetStream not working
**Error**: "JetStream not enabled"
**Solution**: Verify `--js` flag is present and storage directory exists
```bash
# Check JetStream status
curl http://127.0.0.1:8220/jsz
```

### Debug Commands
```bash
# Check which ports are in use
netstat -tlnp | grep :42[0-9][0-9]

# Check cluster info
nats server info --server nats://127.0.0.1:4220

# Monitor real-time
nats-top --server nats://127.0.0.1:4220
```

## Performance Tuning

### Memory Settings
For high-throughput scenarios:
```yaml
nats-cluster-node:
  command: >
    nats-server --name N$${PC_REPLICA_NUM} --js --sd ./data/n$${PC_REPLICA_NUM}
    --max_payload 8MB --max_connections 10000
    # ... rest of command
```

### JetStream Limits
Configure JetStream resource limits:
```yaml
nats-cluster-node:
  command: >
    nats-server --name N$${PC_REPLICA_NUM} --js --sd ./data/n$${PC_REPLICA_NUM}
    --jetstream_max_memory 1GB --jetstream_max_store 10GB
    # ... rest of command
```

## Resources

- [NATS Server Documentation](https://docs.nats.io/running-a-nats-service/introduction)
- [JetStream Guide](https://docs.nats.io/jetstream)
- [NATS Clustering](https://docs.nats.io/running-a-nats-service/configuration/clustering)
- [Process-Compose Documentation](https://f1bonacc1.github.io/process-compose/launcher/)

## License

This recipe is part of the Process-Compose Recipes collection, licensed under MIT License.