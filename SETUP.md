# Rival Rumble - Setup & Administration Guide

## Overview

Rival Rumble is a Minecraft mod for Fabric that extends the Cobblemon experience with player-vs-player battles, rewards, and item stealing mechanics. This guide covers everything server administrators need to know to set up and configure the mod properly.

## Requirements

- Minecraft 1.21.x
- Fabric Server
- Cobblemon Mod (version 1.6.1+)
- Fabric Language Kotlin
- Redis server (for multi-server setups)

## Installation

1. Place the `rival-rumble-x.x.x.jar` in your server's `mods` folder.
2. Start the server once to generate the configuration files.
3. Stop the server.
4. Configure the settings (see Configuration section).
5. Restart the server.

## Configuration

All configuration files are located in the `config/rival-rumble/` directory.

### Main Configuration (config.json)

```json
{
  "enable_gameplay": true,
  "min_party_level": 200,
  "entry_cost": 100,
  "money_steal_amount": 100,
  "winner_cooldown_seconds": 30,
  "loser_cooldown_seconds": 60,
  "challenge_timeout_seconds": 120,
  "selection_timeout_seconds": 30,
  "steal_gui_duration_seconds": 30,
  "neuter_stolen_pokemon": true,
  "store_url": "",
  "leaderboard_size": 10,
  "join_server_command": "execute as %player% run proxycommand 'server Kyogre'",
  "winner_commands": [
    "tellraw %winner% {\"text\":\"Congratulations on your victory!\",\"color\":\"green\"}",
    "give %winner% minecraft:diamond 1"
  ],
  "loser_commands": [
    "tellraw %loser% {\"text\":\"Better luck next time!\",\"color\":\"red\"}",
    "effect give %loser% minecraft:regeneration 30 1"
  ]
}
```

#### Configuration Options

| Option | Description |
|--------|-------------|
| `enable_gameplay` | Enable/disable all gameplay features |
| `min_party_level` | Minimum total level of all Pokémon in a player's party to join |
| `entry_cost` | Cost to join the server (in game currency) |
| `money_steal_amount` | Amount of money stolen when a player wins a battle |
| `winner_cooldown_seconds` | Cooldown period for winners before they can battle again |
| `loser_cooldown_seconds` | Cooldown period for losers before they can battle again |
| `challenge_timeout_seconds` | Time limit for accepting a battle challenge |
| `selection_timeout_seconds` | Time limit for selecting Pokémon for battle |
| `steal_gui_duration_seconds` | Duration that the steal GUI remains open |
| `neuter_stolen_pokemon` | Whether stolen Pokémon are neutered (unable to breed) |
| `store_url` | URL to your server's store (shown in various messages) |
| `leaderboard_size` | Number of players shown on the leaderboard |
| `join_server_command` | Command to execute when player clicks "Join Rumble Server" button (use %player% as placeholder for player name) |
| `winner_commands` | Commands to execute when a player wins a battle |
| `loser_commands` | Commands to execute when a player loses a battle |

### Redis Configuration (redis.json)

This file is important for multi-server setups to enable cross-server validation.

```json
{
  "url": "redis://localhost:6379",
  "enableDebugLogs": false,
  "checkTimeoutMillis": 5000,
  "batchInterval": 50,
  "maxRetryAttempts": 3
}
```

#### Redis Configuration Options

| Option | Description |
|--------|-------------|
| `url` | Redis connection string (format: `redis://[:password@]host[:port]`) |
| `enableDebugLogs` | Enable detailed Redis operation logs |
| `checkTimeoutMillis` | Timeout for cross-server requirement checks |
| `batchInterval` | Interval for batching Redis messages (in milliseconds) |
| `maxRetryAttempts` | Number of retry attempts for failed Redis operations |

### Blacklist Configuration (blacklist.json)

Configure which Pokémon are not allowed in the server.

```json
{
  "species": [
    "mewtwo",
    "mew",
    "lugia",
    "hooh"
  ]
}
```

### Language Configuration (lang.json)

Contains all customizable messages. You can modify this file to change the language or wording of any messages shown to players.

## Setting Up Redis for Multi-Server Deployments

For multi-server setups, a Redis server is required to enable cross-server validation of player requirements.

### Installing Redis

#### Linux:
```bash
sudo apt update
sudo apt install redis-server
sudo systemctl enable redis-server
```

#### Docker:
```bash
docker run --name rival-rumble-redis -p 6379:6379 -d redis redis-server --requirepass "YourSecurePassword"
```

#### Windows:
1. Download Redis for Windows from https://github.com/microsoftarchive/redis/releases
2. Install with default settings
3. Enable the service to start automatically

### Securing Redis

1. Edit the Redis configuration file (`redis.conf`):
   ```
   requirepass YourSecurePassword
   ```

2. Update your `redis.json` to use the password:
   ```json
   {
     "url": "redis://:YourSecurePassword@localhost:6379"
   }
   ```

3. Ensure your firewall allows traffic on the Redis port (default: 6379) between your server instances.

## Multi-Server Setup

The multi-server feature allows:
- Checking player requirements (Pokémon levels, blacklisted Pokémon, balance) from the source server
- Players to move between servers with consistent validation
- Automatic entry fee handling

To implement this:
1. Set up a Redis server accessible by all Minecraft servers
2. Configure each server with the same Redis connection details
3. Ensure each server has the same `min_party_level`, blacklist, and other gameplay configurations

## Post-Battle Commands

You can configure commands to be executed automatically when a battle ends using the `winner_commands` and `loser_commands` arrays in the config. These commands will be run from the server console with the following placeholders available:

| Placeholder | Description |
|-------------|-------------|
| `%winner%` | The name of the winning player |
| `%loser%` | The name of the losing player |
| `%pokemon%` | The name of the stolen Pokémon |

For example:
```json
"winner_commands": [
  "tellraw %winner% {\"text\":\"Congratulations on your victory!\",\"color\":\"green\"}",
  "give %winner% minecraft:diamond 1"
],
"loser_commands": [
  "tellraw %loser% {\"text\":\"Better luck next time!\",\"color\":\"red\"}",
  "effect give %loser% minecraft:regeneration 30 1"
]
```

Notes:
- Loser commands will not execute if the losing player has disconnected
- You can use any valid Minecraft command that the console can execute
- Commands are executed in the order they appear in the configuration
- If a command fails to execute, it will be logged and skipped, but other commands will still run

## Commands

| Command | Permission | Description |
|---------|------------|-------------|
| `/rivalrumble reload` | op | Reload all configuration files |
| `/rivalrumble blacklist add <species>` | op | Add a Pokémon species to the blacklist |
| `/rivalrumble blacklist remove <species>` | op | Remove a Pokémon species from the blacklist |
| `/rivalrumble leaderboard` | everyone | Show the battle leaderboard |
| `/rivalrumble stats` | everyone | Show your battle statistics |

## Troubleshooting

### Connection Issues
- Players can't join: Check if Redis is properly configured and running
- Redis connection errors: Verify firewall settings and Redis credentials
- Log shows "Could not find GameProfile/ClientConnection field": Incompatible Minecraft version

### Gameplay Issues
- Battle system not working: Check if `enable_gameplay` is set to true
- Players bypassing requirements: Ensure Redis is properly configured for multi-server setups
- Missing deduction of entry fee: Verify economy plugin compatibility

### Redis Troubleshooting
1. Check Redis is running:
   ```bash
   redis-cli ping
   ```
   Should return "PONG"

2. Check Redis connectivity:
   ```bash
   redis-cli -h your_redis_host -p 6379 -a YourSecurePassword ping
   ```

3. Check logs for Redis connection issues:
   ```
   grep "redis" logs/latest.log
   ```

## Performance Tuning

For servers with many players:
1. Increase your Redis connection pool size
2. Adjust `batchInterval` for message processing
3. Optimize timeout values

## Support

For further assistance, join our Discord server or open an issue on our GitHub repository.

---

*This guide is for Rival Rumble version 1.0.0 and may change with future updates.* 