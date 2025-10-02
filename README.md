# CO2E Blockchain Node

A Docker Compose setup for deploying a CO2E blockchain node using the OP Stack (Optimism) architecture. This project runs a complete Layer 2 blockchain node with both execution layer (op-reth) and consensus layer (op-node) components.

## Overview

CO2E is a Layer 2 blockchain (Chain ID: 171) built on the OP Stack framework. This deployment includes:

- **op-reth**: Execution layer client (Ethereum-compatible)
- **op-node**: Consensus layer client for L2 block production and L1 data availability

## Architecture

```
┌─────────────────┐    ┌─────────────────┐
│    op-node      │    │    op-reth      │
│  (Consensus)    │◄──►│  (Execution)    │
│                 │    │                 │
│ Port: 8547      │    │ HTTP: 8545      │
│ P2P: 9223       │    │ WS: 8546        │
└─────────────────┘    │ Auth: 8551      │
         │              └─────────────────┘
         │
         ▼
┌─────────────────┐
│   L1 Network    │
│   (Optimism)    │
└─────────────────┘
```

## Prerequisites

- Docker
- Docker Compose
- OpenSSL (for JWT token generation)
- At least 4GB RAM
- 50GB+ free disk space

## Quick Start

1. **Clone and navigate to the project directory:**
   ```bash
   cd co2e-node
   ```

2. **Generate a new JWT token for secure communication between services:**
   ```bash
   openssl rand -hex 32 > config/jwt.txt
   ```

3. **Start the blockchain node:**
   ```bash
   docker-compose up -d
   ```

4. **Check the status:**
   ```bash
   docker-compose logs -f
   ```

5. **Stop the node:**
   ```bash
   docker-compose down
   ```

## Configuration

### Environment Variables

The configuration is stored in `config/.env`:

| Variable | Value | Description |
|----------|-------|-------------|
| `L1_RPC_KIND` | `basic` | L1 RPC client type |
| `L1_RPC_URL` | `https://optimism-rpc.publicnode.com` | L1 Optimism RPC endpoint |
| `CHAIN_ID` | `171` | CO2E chain identifier |
| `RETH_SEQUENCER_HTTP` | `https://rpc.co2e.cc` | CO2E sequencer RPC endpoint |

### Network Configuration

- **Chain ID**: 171
- **Block Time**: 2 seconds
- **L1 Chain**: Optimism (Chain ID: 10)
- **RPC Endpoint**: https://rpc.co2e.cc

### Key Files

- `config/genesis.json`: Genesis block configuration
- `config/rollup.json`: Rollup configuration and parameters
- `config/jwt.txt`: JWT secret for engine API authentication (generate with `openssl rand -hex 32`)
- `config/artifact.json`: Contract artifacts and addresses

## Services

### op-reth (Execution Layer)

- **Image**: `ghcr.io/paradigmxyz/op-reth`
- **Purpose**: Handles transaction execution and state management
- **Ports**:
  - 8545: HTTP RPC (commented out)
  - 8546: WebSocket RPC (commented out)
  - 30303: P2P networking (commented out)

### op-node (Consensus Layer)

- **Image**: `ghcr.io/otternode-io/ott-op-stack-1136:latest`
- **Purpose**: Manages L2 consensus and L1 data availability
- **Ports**:
  - 9223: P2P networking (commented out)
  - 8547: RPC endpoint (commented out)

## Data Persistence

The following directories are mounted for data persistence:

- `./data/op-reth`: Execution layer blockchain data
- `./data`: General node data
- `./data/op_log`: Operation logs

## Networking

### P2P Configuration

The node automatically detects its public IP address and configures P2P networking accordingly. The `op-node-entrypoint` script fetches the public IP from multiple providers:

- ifconfig.me
- api.ipify.org
- ipecho.net/plain
- v4.ident.me

### RPC Access

By default, RPC ports are commented out for security. To enable external access, uncomment the relevant port mappings in `docker-compose.yml`:

```yaml
ports:
  - 9545:8545  # HTTP RPC
  - 9546:8546  # WebSocket RPC
  - 8547:8547  # op-node RPC
```

## Monitoring

### Logs

View real-time logs:
```bash
# All services
docker-compose logs -f

# Specific service
docker-compose logs -f op-reth
docker-compose logs -f op-node
```

### Health Checks

Check if services are running:
```bash
docker-compose ps
```

## Troubleshooting

### Common Issues

1. **Port conflicts**: Ensure ports 8545, 8546, 8547, and 9223 are available
2. **Insufficient disk space**: Monitor disk usage in the `./data` directory
3. **Network connectivity**: Verify L1 RPC endpoint accessibility

### Reset Node Data

To completely reset the blockchain data:
```bash
docker-compose down
sudo rm -rf ./data
docker-compose up -d
```

### Debug Mode

Enable verbose logging by modifying the entrypoint scripts or adding debug flags.

## Security Considerations

- RPC ports are disabled by default for security
- JWT authentication is used between op-reth and op-node
- Private keys are stored securely in the config directory
- Consider using a firewall to restrict access

## Maintenance

### Updates

To update the node software:
```bash
docker-compose pull
docker-compose up -d
```

### Backup

Important files to backup:
- `config/` directory (contains keys and configuration)
- `data/` directory (contains blockchain data)

## Support

For issues related to:
- **CO2E Network**: Visit the official CO2E documentation
- **OP Stack**: Check the Optimism documentation
- **Docker**: Refer to Docker documentation

## License

This project follows the licensing terms of the underlying OP Stack components.

---

**Note**: This is a node deployment for the CO2E network. Make sure you understand the network's terms and conditions before participating.