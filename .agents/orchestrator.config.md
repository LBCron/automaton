# Orchestrator configuration: Automaton

This is the Automaton project's own configuration for `.agents/skills/automaton-orchestrate`. Updated 2026-11-06. No secrets:
credentials live where each section says.

## Project
- Automaton, `rzvdaniel/automaton` (app store and catalog apps: `rzvdaniel/automaton-apps`). Owner: Razvan.
- `main` of `automaton` is protected by rulesets (reviewed PRs and required checks); the owner is admin with bypass and
  pushes small skill and docs changes straight to `main` when asked. `automaton-apps` `main`: ruleset "Main" (pull request
  required, no force push or deletion); the owner bypasses; a CI job commits `catalog.json` through a `catalog-bot` pull
  request (GitHub does not allow the Actions identity as a bypass actor on a personal repo). Agents use pull requests.

## Agents
- **Orchestrator**: Claude Desktop on the owner's Windows PC (Git Bash and PowerShell). The owner switches Claude accounts when
  usage runs out; the repo (this file, the skill, the `handover` issue) carries the state.
- **Server Claude**: a second Claude account on the test server, headless, user `agent`, repo clone `~/automaton`,
  config `~/.claude-second`. Start: write the task to a file, `scp` it to `automaton-runner-01-agent:claude-tasks/tN.txt`, then
  `ssh automaton-runner-01-agent 'nohup bash -c "~/bin/claude-task tN < ~/claude-tasks/tN.txt" > /dev/null 2>&1 &'`
  (never add `</dev/null` after it: the task then gets an empty prompt). Log `~/claude-tasks/tN.log`. Alive?
  `ps -eo etime,args | grep "[c]laude -p Task"`; last work: mtime of `~/.claude-second/projects/*/*.jsonl`. Model Sonnet,
  effort low; it implements issues end to end under `automaton-implement-issue`, may release to the TEST server and merge its
  own reviewed PRs, never production. Queue: a script that waits for `kill -0 <pid>` (pid from `pgrep -f "claude -p Task X" | head -1`),
  then runs the next `claude-task`; started with `nohup`. Never `pkill -f` over ssh (it matches itself).
- **Codex**: on the owner's PC; the owner starts it with one line ("Read the last comment on issue N and do it"). Give it
  docs, README, launch and `automaton-apps` work that needs no server release; it tests on the test server with `e2e-` names only.
- **Subagents** (Agent tool): model `sonnet`, effort `low`.
- Usage is tight on both Claude accounts (Pro plans): keep orchestrator turns short, one server agent at a time.

## Environments
- **Production** `ssh automaton` (user `automaton`, `sudo -n` works; `/opt/automaton-server` and `/etc/automaton` need sudo). API
  `https://api.automaton.monster` (`/health` returns `{"status":"ok","version":...}`), Studio `studio.automaton.monster`, store
  `store.automaton.monster`, website `automaton.monster`. Adopted install with `compose.override.yaml`. Server Runner "Server Machine Management".
- **Test server** `ssh automaton-runner-01` (root) and `ssh automaton-runner-01-agent` (user `agent`). Studio
  `https://studio.runner-01.automaton.monster`, API `https://api.runner-01.automaton.monster`. Credentials for agents:
  `~/.config/automaton-test/accounts.env` on the test server (CRLF; read literally, never `source`, never print); agents approve
  permissions and add users there and report it. Items named `e2e-` are throwaway (a nightly job deletes old ones).
- **Temporary fresh-install servers** (Hetzner, rebuilt by the owner on request) for the llms.txt install tests: `runner-02` at
  `ssh root@2.29.55.250`, domain `runner-02.automaton.monster`; delete after the final run.
- The owner's CLI is logged in to production. For any other server use `AUTOMATON_CONFIG_DIR=<temp dir>` and delete the folder after.
- SSH on the PC: `~/bin` wrappers around Windows OpenSSH use the Windows agent; if SSH asks for a passphrase the owner runs `ssh-add`.
- Self sign-up is disabled on purpose; users come from admin invites or creation. No workarounds.

## Boards and issues (github.com/users/rzvdaniel/projects)
1 Automaton roadmap (fields Plan, Status, Priority `P1 - next|P2 - soon|P3 - later`; engineering). 2 App catalog and ecosystem (parent
#729; Priority `P1|P2|P3`). 3 Public launch (parent #560; fields Priority, Who `Owner|Claude`; label `launch`). 4 Automaton security
audit (not ours). 5 Research (idea #749; label `research`). Filing rules: `automaton-write-issue`. Sub-issues follow their parent onto
its board. Handover: the open issue labelled `handover`. Labels: `verified`, `verified-partial`, `verification-failed`, `live`.

## Release and production
- Server release: `gh workflow run server-release.yml -R rzvdaniel/automaton -f version=X.Y.Z -f cli=auto` (one command; about 10-15 min).
  The server Claude releases to the test server (`automaton server update --wait` there).
- Production update (verified releases are pre-approved by the owner): `ssh automaton 'sudo -n bash /opt/automaton-server/update.sh --version server-vX.Y.Z'`,
  then check `/health`, Studio, store and website return 200. It needs 10 GiB free and prunes old images. Do not downgrade.
- Website (not in the server bundle), on production: never checkout or pull in `/opt/automaton`; `git fetch origin main`, `git archive origin/main` into
  a `mktemp -d`, then `AUTOMATON_PRODUCTION_BUILD_CONTEXT=<dir> AUTOMATON_PRODUCTION_IMAGE_TAG=<sha> docker compose --env-file /opt/automaton/.env -p automaton-prod -f docker-compose.prod.yml build -q website-green`
  and `up -d --no-deps --no-build --wait website-green`.
- Production Runner after a Runner image change: `POST api/runners/{id}/upgrade` with the CLI key, or `automaton runners upgrade --wait`.
- Production apps: publish and delete with the owner's CLI from a temp folder made with `git archive origin/main <folder>` of `automaton-apps`
  (never the owner's working copy; never publish an app without a valid `automaton.json`; use `automaton listing` for cover and text).
- Fresh-server tests (#561): the owner rebuilds a temporary server; a subagent follows only `https://automaton.monster/llms.txt`.
- Pre-approved by the owner: releases and production updates of verified work; production app publish/delete when he names the app.
  Always ask first: deleting data volumes, backups or the database; loosening protections.

## Standing decisions (2026-11)
Each issue atomic, no batching of releases. No claiming labels. One server agent plus the orchestrator (usage). Self sign-up stays
disabled. Only high-quality apps and tools are published (Untranslatable is the bar). No extra servers or paid monitors. The owner pushes
straight to `main` of `automaton-apps`; agents use PRs. `verified-partial` closes only with the owner's written acceptance. Installing
tools on the owner's PC is fine; no inbound firewall rules left behind. Never print or commit credentials.

## Local quirks and traps
- Long inline Python or heredocs through Bash break on quotes: write a script file (Write tool) and run it.
- `gh` GraphQL gives sporadic 5xx; REST works; post bodies with `gh api ... --input file.json`. Board 1 option names are `P1 - next` etc.
  Changing option lists clears every item's value (snapshot first).
- Windows console is cp1252: decode CLI output with `errors="replace"`. Strip `\r` from scripts sent to Linux.
- The CLI's `catalog` endpoint needs a bearer token (Studio login), not the CLI key.
- The auto-mode safety check blocks loosening protections unless the owner states the exact change.
- Local `C:\Work\automaton-apps` is the owner's working copy: never publish from it.
