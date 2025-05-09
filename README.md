# Rival Rumble

A Fabric mod for Minecraft that extends Cobblemon with competitive PvP gameplay, rewards, and cross-server functionality.

## Features

- **Player Battles**: Challenge other players to Pokémon battles
- **Rewards System**: Win money and potentially steal Pokémon from opponents
- **Multi-Server Support**: Players can move between servers with consistent requirements validation
- **Entry Requirements**: Configurable minimum party level and entry costs
- **Cooldown System**: Prevents battle spam with configurable cooldowns for winners and losers
- **Blacklist System**: Prevent specific Pokémon from being used on your server
- **Leaderboards**: Track player performance and rankings

## Requirements

- Minecraft 1.21.x
- Fabric Server
- Cobblemon 1.6.1+
- Fabric Language Kotlin
- Redis (for multi-server setups)

## Quick Start

1. Install the mod in your server's `mods` folder
2. Start the server once to generate configuration files
3. Configure settings in `config/rival-rumble/`
4. Restart the server

## Configuration

Detailed configuration options can be found in the [Setup Guide](SETUP.md).

## Commands

| Command | Description |
|---------|-------------|
| `/rivalrumble reload` | Reload all configuration files |
| `/rivalrumble blacklist add <species>` | Add a Pokémon species to the blacklist |
| `/rivalrumble blacklist remove <species>` | Remove a Pokémon species from the blacklist |
| `/rivalrumble leaderboard` | Show the battle leaderboard |
| `/rivalrumble stats` | Show your battle statistics |

## Multi-Server Setup

For multi-server setups, Rival Rumble uses Redis to synchronize player data and validate requirements across servers. This ensures players meet the entry requirements (minimum party level, no blacklisted Pokémon, sufficient balance) before joining any server in your network.

See the [Setup Guide](SETUP.md) for detailed instructions on configuring Redis.
