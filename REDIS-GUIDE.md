# Rival Rumble - Redis Integration Guide

This guide provides detailed information about the Redis integration in Rival Rumble, which enables cross-server requirement validation for players.

## Why Redis?

In a multi-server Minecraft network running Rival Rumble, players need to meet certain requirements (minimum party level, no blacklisted Pokémon, sufficient balance) to join. When players move between servers, their data might be out of sync or not immediately available on the destination server.

Redis solves this by:
1. Allowing servers to communicate in real-time
2. Checking player requirements on the source server where the player is currently active
3. Ensuring consistent enforcement of rules across your network

## How It Works

When a player attempts to join a server with Rival Rumble:

1. The destination server sends a request via Redis to all servers
2. Any server with the player online checks their requirements locally
3. The source server responds with the check results and player data
4. The destination server makes a join decision based on this data
5. If applicable, entry costs are deducted from the player's balance on the source server

This ensures that players cannot bypass requirements by server-hopping, and that all validation uses the most up-to-date player data.

## Redis Configuration

The Redis configuration file is located at `config/rival-rumble/redis.json`:

```json
{
  "url": "redis://localhost:6379",
  "enableDebugLogs": false,
  "checkTimeoutMillis": 5000,
  "batchInterval": 50,
  "maxRetryAttempts": 3
}
```

### Configuration Options Explained

| Option | Description | Recommended Setting |
|--------|-------------|---------------------|
| `url` | Redis connection string | `redis://[:password@]host[:port]` |
| `enableDebugLogs` | Toggle detailed Redis logs | `false` in production, `true` for troubleshooting |
| `checkTimeoutMillis` | How long to wait for a response | `5000` (5 seconds) |
| `batchInterval` | Interval for batching Redis messages | `50` ms (lower for faster response) |
| `maxRetryAttempts` | Retries for failed operations | `3` (increase for unstable connections) |

## Setting Up a Redis Server

### Option 1: Dedicated Redis Server (Recommended for Production)

#### Linux:
```bash
sudo apt update
sudo apt install redis-server
sudo systemctl start redis-server
sudo systemctl enable redis-server
```

Edit `/etc/redis/redis.conf`:
```
bind 0.0.0.0  # To allow remote connections
requirepass YourStrongPassword
```

Restart Redis:
```bash
sudo systemctl restart redis-server
```

#### Windows:
1. Download Redis for Windows from https://github.com/microsoftarchive/redis/releases
2. Run the installer
3. Open `redis.windows.conf` and set:
   ```
   bind 0.0.0.0
   requirepass YourStrongPassword
   ```
4. Restart the Redis service

### Option 2: Docker (Great for Testing or Small Setups)

```bash
docker run --name rival-rumble-redis -p 6379:6379 \
  -v redis-data:/data \
  -d redis:latest redis-server --requirepass "YourStrongPassword"
```

### Option 3: Redis Cloud Services

For large networks, consider using managed Redis services:
- Redis Cloud
- AWS ElastiCache
- Azure Cache for Redis
- Google Cloud Memorystore

## Connecting Your Servers

1. Make sure all Minecraft servers can reach the Redis server
2. Update the `redis.json` on each server with the correct URL:
   ```json
   {
     "url": "redis://:YourStrongPassword@redis-server-ip:6379"
   }
   ```
3. Ensure firewall rules allow TCP traffic on port 6379 between all servers

## Monitoring and Debugging

### Enabling Debug Logs

Set `enableDebugLogs` to `true` in `redis.json` to get detailed logs about Redis operations.

### Checking Redis Connection

Use Redis CLI to verify connectivity:
```bash
redis-cli -h your-redis-host -p 6379 -a YourStrongPassword ping
```

You should receive `PONG` as a response.

### Common Issues and Solutions

| Issue | Possible Cause | Solution |
|-------|---------------|----------|
| Connection timeout | Network configuration | Check firewall settings |
| Authentication failed | Incorrect password | Verify the password in URL |
| No response from other servers | Redis subscriber not running | Check server logs for Redis initialization |
| Slow response times | Network latency | Reduce `checkTimeoutMillis` or optimize network |
| High memory usage | Message accumulation | Adjust `batchInterval` to process messages faster |

## Performance Optimization

For large networks with many players:

1. **Connection Pool Tuning**:
   - Increase the connection pool size if you have many concurrent operations

2. **Message Batching**:
   - Adjust `batchInterval` based on your network performance:
     - Lower values: More responsive but higher Redis load
     - Higher values: More efficient but slightly delayed responses

3. **Timeout Settings**:
   - Set `checkTimeoutMillis` based on your network latency:
     - For low-latency networks: 2000-3000ms
     - For higher-latency networks: 5000-8000ms

## Advanced Configuration

### Handling Network Partitions

If your network spans multiple regions, consider:
1. Setting up Redis replicas in each region
2. Using Redis Sentinel for high availability
3. Implementing fallback logic in case Redis becomes unavailable

### Security Best Practices

1. **Network Security**:
   - Use a private network for Redis traffic
   - Implement IP-based restrictions

2. **Authentication**:
   - Use a strong password
   - Rotate passwords regularly

3. **Encryption**:
   - Enable TLS for Redis connections if available

## Technical Details

### Redis Message Format

Rival Rumble uses these Redis message patterns:

1. Requirement check request:
   ```
   checkRequirements:<playerUuid>:<requestId>
   ```

2. Requirement check response:
   ```
   requirementResponse:<requestId>:<meetsRequirements>:<failReason>:<partyLevel>:<balance>:<blacklistedPokemon>:<failCode>:<minPartyLevel>:<entryCost>
   ```

3. Entry cost deduction:
   ```
   deductEntryCost:<playerUuid>:<amount>
   ```

### Fallback Behavior

If Redis is unavailable or no response is received:
1. The system falls back to checking requirements locally
2. This may use cached/outdated data but ensures the server remains functional

## Conclusion

The Redis integration in Rival Rumble provides a robust solution for multi-server networks, ensuring consistent enforcement of requirements across all servers. By properly configuring your Redis setup, you can provide a seamless experience for players moving between servers in your network.

---

*For general Rival Rumble setup instructions, see the [main setup guide](SETUP.md).* 