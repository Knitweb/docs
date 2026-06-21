# 5mart.ml — deployment & operations runbook

**What it is.** `5mart.ml` is the owner's live **TransIP shared hosting** (PHP 8.1 + MySQL, mod_rewrite on; **no long-lived processes**, only `crontab`). It hosts the production site at `~/www` and the **MOLGANG dapp** at `~/www/molgang` (live: https://5mart.ml/molgang/).

> ⚠️ It is a **live production host** with real PHP, payment code, a `.env`, and backups in `~/www`. Touch **only `~/www/molgang/`**; back up before changing; never read/modify the other apps.

## Access (passwords intentionally NOT written here)
| | Value |
|---|---|
| **SSH host** | `5martm.ssh.transip.me` (port 22) |
| **SSH user** | `5martml` |
| **SSH password** | `‹from your password manager — omitted for safety›` |
| **Home** | `/data/sites/web/5martml/` · web root `~/www` |
| **MySQL host** | `127.0.0.1` (⚠️ **not** `localhost` — the socket path differs; `localhost` fails) |
| **MySQL db** | `5martm_ED` |
| **MySQL user** | `5martm_develuse` |
| **MySQL password** | `‹from your password manager — omitted for safety›` |

> The SSH and DB passwords were provided to the operator out-of-band. Do **not** commit them or store them in any synced/inspectable file. The dapp reads the DB password only from `~/www/molgang/config.php` (chmod 600, `.htaccess`-blocked, gitignored).

```bash
sshpass -e ssh -o PreferredAuthentications=password 5martml@5martm.ssh.transip.me   # SSHPASS set in your shell, not on disk
```

## The MOLGANG PHP dapp layout (`~/www/molgang/`)
```
index.php · .htaccess (rewrites /api → index.php; blocks src/, schema.sql, config.php)
src/ (Bar.php, Db.php, Parse.php, Chemistry.php, Progression.php)
config.php  (DB creds — chmod 600, NEVER in git)
index.html · app.js · style.css · avatars/ · schema.sql
explorer.html · graph.json  (the static knowledge-graph explorer)
```

## (Re)deploy
```bash
cd ~/www && cp -r molgang molgang.bak.$(date +%s)            # always back up
git clone --depth 1 https://github.com/knitweb/molgang /tmp/mg
cp -rf /tmp/mg/php/public/. molgang/ ; rm -rf molgang/src ; cp -rf /tmp/mg/php/src molgang/src
sed -i 's#/\.\./src/#/src/#g' molgang/index.php             # flat layout: index.php → ./src
# config.php: host=127.0.0.1, name=5martm_ED, user=5martm_develuse, pass=‹secret›
# schema (web context only — CLI can't reach the DB): hit a one-time token-guarded setup.php, then delete it
```

## Gotchas learned in production
- **DB host = `127.0.0.1`** (the web SAPI; `localhost`/`/tmp/mysql.sock` is unreachable, and the CLI can't reach MySQL at all → apply schema via a web-context PHP script).
- **mod_rewrite is on** (the main site uses it) → `/molgang/api/*` routes to `index.php`.
- The DB `5martm_ED` is **shared with the main site** (e.g. a `tickermappings` table) — keep MOLGANG table names namespaced; never drop unknown tables.
- Errors are masked to `{"error":"server error"}`; real cause goes to the PHP error log.

## Keep-alive / scheduling
No daemons — use **`crontab`** for the daily digest (#76) and any maintenance. p2p/relay presence is HTTP-only (#61/#62).
