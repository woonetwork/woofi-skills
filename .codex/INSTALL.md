# Installing WOOFi Skills for Codex

Enable WOOFi skills in Codex via native skill discovery. Just clone, symlink.

## Prerequisites

- Git
- No API credentials required — WOOFi Swap API is public

## Installation

1. **Clone the repository:**

   ```bash
   git clone https://github.com/woonetwork/woofi-skills ~/.codex/woofi-skills
   ```

2. **Create the skills symlink:**

   ```bash
   mkdir -p ~/.agents/skills
   ln -s ~/.codex/woofi-skills/skills ~/.agents/skills/woofi-skills
   ```

   **Windows (PowerShell):**

   ```powershell
   New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.agents\skills"
   cmd /c mklink /J "$env:USERPROFILE\.agents\skills\woofi-skills" "$env:USERPROFILE\.codex\woofi-skills\skills"
   ```

3. **Restart Codex** (quit and relaunch the CLI) to discover the skills.

## Verify

```bash
ls -la ~/.agents/skills/woofi-skills
```

You should see the four skill directories: `woofi-trading-stats`, `woofi-volume-by-source`, `woofi-earn-tvl`, `woofi-woo-staking`.

## Available Skills

| Skill | When to Use |
|-------|-------------|
| `woofi-trading-stats` | Trading volume, trader counts, transaction counts by period and network |
| `woofi-volume-by-source` | Volume breakdown by integrator or aggregator source |
| `woofi-earn-tvl` | Earn vault TVL, APY, and yield farming metrics |
| `woofi-woo-staking` | WOO token staking APR and total staked amount |

## Updating

```bash
cd ~/.codex/woofi-skills && git pull
```

Skills update instantly through the symlink.

## Uninstalling

```bash
rm ~/.agents/skills/woofi-skills
```

Optionally delete the clone: `rm -rf ~/.codex/woofi-skills`.
