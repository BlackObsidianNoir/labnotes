# BlackMountain: Hosting and Migration Summary v1.0

Prepared for Edward Nathan Page, October 31, 2026.

## What it is

BlackMountain (formerly WeatherEdge) is Ed's Python trading system for Kalshi event contracts. It began with daily high-temperature contracts and no longer trades weather. It runs live with capital caps of about $1,500. The identifier everywhere is `blackmountain` (lowercase, no hyphen or underscore).

## Where it runs today

- An M1 Mac mini at home (`~/weather_edge`), with jobs scheduled by launchd.
- The M1 hosts three systems: BlackMountain (nine jobs), novig_edge (five jobs) and baccarat (one job). It also runs Roon Server.
- Remote access through Tailscale, tmux and Claude Code Remote Control.
- Databases: `edge.db` (about 4.1 GB) and `trades.db` (about 0.4 GB), both SQLite.

## Done so far

- Private GitHub repo `BlackObsidianNoir/blackmountain` created; 622 commits and 195 files pushed after a clean secret scan.
- Nightly push from the M1 to GitHub at 05:30, using a deploy key limited to that repo.
- Nightly database backup at 01:15 with an integrity check and a weekly test restore. It writes to `~/BlackMountainBackups` on the M1; Time Machine carries it to a Synology NAS, which backs up to a RAID drive, which Backblaze backs up off-site. (macOS blocks background jobs from writing to network drives, so the backup writes locally first.)
- Healthchecks.io (dead-man switch) and Pushover (phone alerts) set up for the backup and the GitHub push.
- Render account created, two-factor on. Nothing deployed yet.
- `MIGRATION.md` (dependency inventory) committed in the repo.

## Target architecture (decided)

- Production trading moves to Render: a background worker plus Render Postgres.
- Postgres holds live-money records: orders, fills, positions, risk state, reconciliation. Model artifacts can stay as files.
- Arm and halt switches become database flags, off by default.
- Reconcile against Kalshi on every restart before any order. Kalshi is the source of truth for positions.
- Timestamps in UTC; each contract's settlement date stored separately with its named time zone. A test suite covers daylight-saving changes and midnight rollovers, which also fixes a known date bug.
- One GitHub repo per system; BlackMountain is the template for future systems.
- Analysis projects get read-only access to trading data and can never place orders.
- AI agents propose changes on branches; only an approved merge to `main` deploys. GitHub Pro and branch protection on `main` are required before the first production deploy.
- A separate Schwab experiment (Claude vs. ChatGPT trading two small Schwab accounts) will also run on Render with its own repo, database and secrets. Nothing is shared with BlackMountain.

## Migration sequence

- **A1. Protect what exists:** done, except excluding the live databases from Time Machine.
- **A2. Stabilize on the M1:** Python 3.9 to 3.12, fix SQLite lock contention, separate the nightly research job's permissions from live order placement, health checks on all jobs.
- **A3. Build the cloud execution core on Render,** with the M1 still the only live order writer.
- **A4. Shadow test:** cloud copy runs on live data without placing orders; compare decisions with the M1.
- **A5. Cut over** one probe at a time at natural experiment boundaries. Halt the M1, reconcile, arm the cloud copy. Keep the M1 as rollback for 7 to 10 days while Ed is home.

## Latest decision

Given how much the M1 actually runs, BlackMountain and novig_edge stay on the M1 for now rather than rushing the cloud move. A2 and A3 have not started.

## Open questions

- Does novig_edge move to the cloud or retire?
- Does Kalshi support expiration on resting orders?
- Render cost: estimated $40 to $80 a month for both systems, to verify.

Use this as context. Ask before assuming anything not stated here.
