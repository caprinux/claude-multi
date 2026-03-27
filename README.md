# claude-multi

Run multiple Claude Code instances with different accounts on the same machine, simultaneously.

Claude Code ties each session to a single authenticated account. `claude-multi` manages isolated profiles so you can run multiple accounts concurrently without conflicts.

## How it works

Each profile gets its own config directory at `~/.claude-profiles/<name>/` with isolated credentials. Session data (conversations, history) is **shared across all profiles** via symlinks to `~/.claude/`, so `--continue` and `--resume` work regardless of which profile you use. Settings and plugins are synced from your main `~/.claude/` config automatically.

### Shared sessions

When a profile is created or launched, `claude-multi` symlinks the following from the profile directory to `~/.claude/`:

- `projects/` — conversation history (JSONL files)
- `sessions/` — session metadata
- `history.jsonl` — command history

This means you can start a session with one profile and resume it with another. The resumed session uses whichever profile's credentials you launch with.

If a profile already has local session data (e.g. from a previous version of `claude-multi`), it is automatically migrated to the shared location when symlinks are set up.

## Install

```bash
# Clone and symlink to PATH
git clone https://github.com/caprinux/claude-multi.git
ln -s "$(pwd)/claude-multi/claude-multi" /usr/local/bin/claude-multi

# Or just copy it
curl -o /usr/local/bin/claude-multi https://raw.githubusercontent.com/caprinux/claude-multi/main/claude-multi
chmod +x /usr/local/bin/claude-multi
```

### Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI installed
- `python3` (for reading JSON config files)
- `bash` 4+

## Quick start

```bash
# 1. Login to your first account
claude-multi login work
# This opens a shell. Run `claude` and use `/login`, then type `exit`.

# 2. Login to your second account
claude-multi login personal
# Same flow — login, then exit.

# 3. Run them simultaneously in separate terminals
claude-multi run work           # terminal 1
claude-multi run personal       # terminal 2

# 4. Resume a session from any profile
claude-multi run work --continue    # picks up where you left off
```

## Commands

### Account management

| Command | Description |
|---|---|
| `login <profile>` | Create a profile and open a shell to login. Credentials are saved directly to the profile. |
| `save <profile>` | Save the currently logged-in account (from `~/.claude/`) as a named profile. |
| `logout <profile>` | Logout a profile and invalidate its tokens. |
| `status [profile]` | Show auth status for one or all profiles. |
| `list` | List all profiles with email, plan, and token status. |
| `delete <profile>` | Delete a profile and its credentials. |
| `rename <old> <new>` | Rename a profile. |

### Running Claude Code

| Command | Description |
|---|---|
| `run <profile> [args...]` | Run Claude Code using a specific profile. All Claude CLI arguments are passed through. |
| `shell <profile>` | Open a subshell where all `claude` commands use the profile. |

### Configuration

| Command | Description |
|---|---|
| `sync <profile>` | Sync settings, plugins, skills, and commands from `~/.claude/` to a profile. |
| `which <profile>` | Print the config directory path for a profile. |

## Examples

```bash
# List all profiles
claude-multi list

# Run Claude with a prompt using a specific profile
claude-multi run work -p "fix the failing tests"

# Resume the last session with a different profile
claude-multi run personal --continue

# Open a shell where `claude` is bound to a profile
claude-multi shell work
claude          # uses work account
claude -p "..."  # uses work account
exit            # back to normal shell

# Sync updated settings to a profile
claude-multi sync work

# Check which account a profile uses
claude-multi status work
```

## Environment variables

| Variable | Description |
|---|---|
| `CLAUDE_MULTI_HOME` | Override the profiles directory (default: `~/.claude-profiles`) |

## How login works

On headless/remote servers, `claude auth login` can have issues with the OAuth callback (it starts a localhost listener that your browser can't reach). `claude-multi login` works around this by:

1. Creating an isolated profile directory
2. Opening a shell with `CLAUDE_CONFIG_DIR` set to that directory
3. You run `claude` from that shell and use `/login` (the interactive TUI handles the auth code paste correctly)
4. Credentials are written directly to the profile directory

## License

MIT
