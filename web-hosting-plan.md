# Web App Hosting Plan v1.0

Prepared for Edward Nathan Page, October 31, 2026. Claude's response to ChatGPT's proposal (Render plus Cloudflare Access). Nothing has been deployed.

## Recommendation

Host small personal and family apps on **Cloudflare Workers with D1 databases**, protected by **Cloudflare Access**. Use Render only for apps that need Python, long-running jobs or heavy compute, and for trading systems. Ed already runs this pattern: the Melissa app is on Cloudflare behind Zero Trust Access.

ChatGPT's security principles stay: app-level authorization on every page and API, no trust in the launcher or hidden links, a separate database per sensitive app, sample data first, and tested restores. The disagreement is about the platform.

## Why Workers instead of Render plus Access

1. **Origin bypass is closed by the platform.** On Render, Access only protects traffic routed through Ed's Cloudflare zone. The onrender.com address and PR preview URLs are a second door. Render can disable the onrender.com subdomain once a custom domain exists, but it must be done per service, and the app's token check carries the whole load. On Workers, Access applies however the request arrives: custom domain, route, workers.dev subdomain or preview URL. An account-level policy can make every Worker private by default, with public sites as explicit exceptions.
2. **Private static sites stop being a special case.** A Render static site has no server, so it cannot validate Access tokens. On Workers, a static site is a Worker with assets and gets the same protection.
3. **Isolation is free.** Each app gets its own Worker and its own D1 database. A Worker can reach only the databases bound to it. D1 is priced on queries and storage, not per database, so there is no cost reason to group apps.

## Costs

| Apps | Render ($7 web + $6 Postgres each) | Workers + D1 |
|---|---|---|
| 5 | 5 x $13 = $65/mo | about $5/mo |
| 10 | 10 x $13 = $130/mo | about $5/mo |
| 20 | 20 x $13 = $260/mo | about $5/mo |

Render storage is extra. Workers Paid is $5/month and includes 10 million requests and 30 million CPU milliseconds; family apps will not approach that. Domain about $10 to $15 a year. Access is free up to 50 users.

## Permissions

- **Cloudflare Access decides who gets in.** One Access policy per app, built from reusable groups ("Ed only", "Family"). Family uses one-time email codes; Ed uses Google or GitHub with a passkey.
- **The app decides view or edit.** A members table maps email to viewer or editor, checked server-side on every write.
- **Revoking:** remove the person from the Access policy and revoke their sessions.
- **Known gap:** access lives in two places (Access and the members table). Default unknown users to viewer and review both quarterly.

## Medication dashboard and HIPAA

HIPAA covers healthcare providers, health plans, clearinghouses and their business associates. A family member running a personal dashboard is none of these, so a provider's BAA terms do not apply. The privacy bar stays high: Access on every URL, no health data in logs or repos, sample data until the template passes its tests, and Mom knowing what is stored.

## Isolation from trading

Web apps live on Cloudflare; trading lives on the M1 and, later, Render. Different platforms, accounts and credentials, so no web app has a path to a trading key. The shared risk is the Cloudflare login itself: protect it with a passkey or hardware key and use deploy tokens scoped to Workers only.

## Project template (one private repo per app)

- Worker with static assets and an API, one D1 database, members table, Access on all hostnames.
- Pull request with tests; merge to main deploys; preview versions are Access-protected.
- Code rollback: Workers keeps prior versions; revert in one click.
- Data rollback: D1 Time Travel restores to any minute in the last 30 days (paid plan), plus a nightly export to the Synology.

## Trade-offs

- Workers code is JavaScript or TypeScript, not Python. Claude writes it either way.
- D1 is SQLite, not Postgres. Fine at this size, and it exports to plain SQL.
- Apps needing Python, background jobs or heavy compute go to Render, using ChatGPT's design: disable onrender.com, validate the Access token's audience and expiry, and protect or disable preview environments.

## Setup sequence

1. Choose and register the domain at Cloudflare.
2. Secure the Cloudflare account with a passkey; turn on Workers Paid ($5/month).
3. Zero Trust: set login methods, create "Ed only" and "Family" groups, turn on "protect all Workers" for production and previews.
4. Build the template with a throwaway test app and prove it:
   - An unauthenticated request to every URL, including workers.dev and a preview, gets the login page.
   - A family viewer can open the app but cannot write.
   - Removing a user locks them out.
   - A bad deploy rolls back.
   - The database restores from Time Travel and from the Synology export.
5. Build Mom's dashboard on the template with sample data.
6. Switch to real data only after those tests pass.

The App Launcher is optional once there are three or more apps.

## Confidence

High on the cost math, the HIPAA answer and Workers' Access coverage, checked against current Render and Cloudflare documentation on October 31, 2026. Moderate that Workers fits every future app; some will likely need Render. The recommendation flips to Render if most apps turn out to need Python or background jobs.

## Questions for ChatGPT

1. Where is this plan wrong, risky or overcomplicated?
2. Do you agree Workers plus D1 beats Render for small family apps? If not, why?
3. Is there a gap in the two-layer permission model?
4. Anything missing from the proof tests in step 4?
