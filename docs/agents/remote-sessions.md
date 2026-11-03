# Remote agent sessions

How the owner reaches the coding agents on the test server from a phone or a desktop app. Agents there run as the `agent` user in `/home/agent/automaton`, never as root (see the operating rules in [AGENTS.md](../../AGENTS.md)).

Both remote modes stop when the server reboots. Start them again as described below.

## Claude Code: Remote Control

Remote Control lets the Claude app (mobile, desktop or claude.ai) open a Claude Code session running on the server.

Start it in tmux as `agent`, so it keeps running after you disconnect:

```bash
sudo -u agent -H tmux new-session -d -s claude-rc -c /home/agent/automaton claude remote-control
```

From inside an interactive Claude Code session, run `/remote-control` instead. The session then appears in the Claude app under the owner's account. To check it's running, run `sudo -u agent tmux ls` and look for `claude-rc`.

## Codex: remote-control daemon

`codex remote-control` (experimental in Codex 0.166) runs a Codex app-server daemon that the ChatGPT/Codex app can pair with.

```bash
# as agent, in the checkout
codex remote-control start   # start the daemon; prints the machine name
codex remote-control pair    # print a short-lived pairing code for the phone
codex remote-control stop    # stop the daemon
```

On the phone, add a remote machine in the Codex or ChatGPT app and enter the pairing code before it expires.

If `start` fails with `app server is running but is not managed by codex app-server daemon`, a Codex desktop SSH session left its own app-server running. Close that chat in Codex desktop. If the processes remain (`pgrep -af "codex.*app-server"`), stop them, then start again.

## Codex desktop over SSH

Codex desktop runs Codex on the server over SSH. Use an SSH host entry that signs in as `agent`, since Codex is installed for that user and not for root. Then set the project folder to `/home/agent/automaton`. For agents to work without approval prompts, the `agent` user's `~/.codex/config.toml` sets:

```toml
approval_policy = "never"
sandbox_mode = "danger-full-access"
```

A chat can also be switched to **Full access** in the composer.
