# mytune — a real-time MySQL monitor (free)

![.NET 11](https://img.shields.io/badge/.NET-11.0-512BD4)
![C# 15](https://img.shields.io/badge/C%23-15-239120)
![Blazor Hybrid](https://img.shields.io/badge/Blazor-Hybrid-5C2D91)
![MySQL 8.0+](https://img.shields.io/badge/MySQL-8.0%2B-4479A1)
![Windows x64](https://img.shields.io/badge/Windows-x64-0078D6)

**[Download the latest release](https://github.com/Doni-Kim/mytune-release/releases/latest)** ·
[한국어 설명](README.ko.md) · [Manual (Korean, HTML)](mytune.html)

mytune is a desktop monitor for MySQL. One window, one executable, nothing to install on the server —
everything is read from `performance_schema`, `information_schema` and `sys`.
Free to use, no strings attached.

## Screenshots

| Live dashboard | Top SQL |
|---|---|
| ![Dashboard](screenshots/dashboard.png) | ![Top SQL](screenshots/top-sql.png) |

| Lock chains | Session detail |
|---|---|
| ![Locks](screenshots/locks.png) | ![Session detail](screenshots/session-detail.png) |

| History | Alerts |
|---|---|
| ![History](screenshots/history.png) | ![Alerts](screenshots/alerts.png) |

![Dark theme](screenshots/dashboard-dark.png)

The screenshots show a throwaway demo server with a made-up `shop` schema.

## Install

- Unzip, keep the folder together, and run `mytune.exe` (single-file publish).
- No .NET install needed — the runtime is inside the executable.
- The only thing to set up is a connection file next to the executable. The zip ships two, `mytuneNode1.json` and
  `mytuneNode2.json` — one file per server. Watching a single server? Keep one and delete the other.
  Either edit it (see below), or just run `mytune.exe` — when the values do not connect, a connection dialog opens with
  them filled in. Once the connection has succeeded, what you typed is saved back (the password is stored encrypted).

## What it does

- **Live dashboard** — buffer pool hit ratio and connections (gauges), QPS / TPS / running / waiting (trend graphs),
  row lock waits, deadlocks, rollbacks, slow queries, rows read and modified, disk read / write, network in / out,
  redo and history list length, top wait events, and the session list. Each metric appears once.
  Cumulative server counters are always shown as the change since the previous sample.
- **Lock chains that name the real blocker** — row locks come from `data_lock_waits`; metadata locks are
  resolved with the server's own lock-compatibility rules, so a session is listed as a blocker only when
  its lock really conflicts. `F5` shows blockers together with the sessions they block.
- **Session detail** (`Enter`) — full statement, execution plan, locks held, connection attributes.
  `Ctrl+K` kills the query or the whole connection; `Ctrl+X` exports the session to Excel.
- **Panels** — Server (`I`), Connections (`C`), Locks (`A`), InnoDB (`V`), Replication (`W`),
  Top SQL (`T`, by digest, with a delta mode and text search), Index diagnostics (`X`), Disk (`D`).
- **Alerts** — 12 rules: connection saturation, waiting sessions, idle in transaction, long statements,
  buffer pool hit, rollback ratio, temp tables on disk, lock chains, deadlocks, history list,
  replication lag and stopped replication threads — with your own thresholds.
- **History** — press `L` to log every sample into a local SQLite file, then `H` to look back.
  - Ranges: 1 hour / 6 hours / 24 hours / 1 week / 1 month / all. 17 metrics.
  - Old rows are trimmed automatically (30 days of metrics, 7 days of sessions by default; change it with `O`).
- **Excel export** — built on ClosedXML, so the `.xlsx` is written even without Excel installed.
- Reconnects by itself when the connection drops.
- **New in 2.2** — Error log popup (`E`, from `performance_schema.error_log`, MySQL 8.0.22+); Changed settings and
  Memory in use in Server; Load by user in Connections; Hot tables in Index; P95 / P99 in Top SQL.
- **Blocking tree** — Locks (`A`) draws who blocks whom as a tree, and the Excel export carries the whole chain.
- **Find** — `/` filters the session list by text.
- **When an alert fires** — the blocking tree, statements or the deadlock report are saved to a file under `captures\`,
  and a Critical alert flashes the taskbar and shows a Windows notification when the window is not in front.
- In History, click a point in time to see the sessions that were logged at that moment.
- `sslMode` gains `verify-ca` / `verify-full`, with `sslCa` for the CA file; a rejected certificate is explained in the
  connection dialog. Hot tables now shows 50 rows with its share of all table I/O time and a `Δ delta` mode.
- **Admin reference** — the second `F1` tab looks up, as you type, the documentation the server itself carries: the
  `sys` schema routines (description, parameters, example) and the server help tables that `HELP` reads (700 statements
  and functions, with a documentation link) — so it always matches the server's version, with no internet needed.
  54 common admin tasks come with a sample to copy; mytune never runs admin statements.
- **One settings file per server** — with two or more next to the executable, mytune asks which one to use at startup.
  A file broken by a hand edit is listed in red with the line and position of the error.
- **Settings screen** — `O` changes the collection interval (3–60 s, 5 by default), log retention, Top SQL, Excel, alert
  thresholds and which sessions `L` logs. Values are checked, saved to the settings file in use and applied at once;
  the connection itself is changed only in the startup window.
- **12 themes** — six light, six dark, GitHub Light by default; pick one from the top bar.
- **Server not answering at startup** — a small window shows whom mytune is connecting to and for how long, with
  Cancel to fix the connection, instead of an empty screen for 20-odd seconds.
- Press `F1` for the keyboard shortcuts, and again for the Admin Reference tab.

The bundled `mytune.html` is the full manual with screenshots (in Korean).

## Requirements and limits

- **Windows only.** The UI is web-based (Blazor Hybrid), so it does not run standalone on Linux or macOS.
- **MySQL 8.0 or later** with `performance_schema=ON` — checked on 8.0.46 and the current release.
  MySQL-compatible distributions with the same `performance_schema` layout (such as Percona Server 8.0+) work the same way.
- **MariaDB is not supported** — its `performance_schema` has no `data_locks` / `data_lock_waits`.
  mytune tells you so in the connection dialog.
- The code is obfuscated with ConfuserEx — a free tool, so do not expect strong protection.

## A dedicated monitoring account

```sql
CREATE USER 'mytune'@'%' IDENTIFIED BY '...';
GRANT PROCESS ON *.* TO 'mytune'@'%';                  -- other users' sessions, innodb_metrics
GRANT SELECT ON performance_schema.* TO 'mytune'@'%';
GRANT SELECT ON sys.* TO 'mytune'@'%';                 -- index diagnostics
GRANT REPLICATION CLIENT ON *.* TO 'mytune'@'%';       -- optional: Replication panel and alerts
GRANT CONNECTION_ADMIN ON *.* TO 'mytune'@'%';         -- optional: Ctrl+K on other users' sessions
```

With such an account the plan in Session detail is an *estimated* plan (and says so); the live plan of
another user's running statement is only available to an administrative account such as root.

Disk (`D`) and index diagnostics (`X`) see only schemas the account has a privilege on — `information_schema`
hides the rest without an error. mytune says how many schemas are hidden; grant `SELECT` on them to see them.

## Blank window? (WebView2)

If the window opens but stays blank, the WebView2 runtime is missing.

- **Windows 11** — built into the OS, always present.
- **Windows 10** — shipped through Windows Update since 2021, so it is there on most machines.
  A PC that has not been updated in a long time, or a special edition such as LTSC, may not have it.
- **Windows Server (2016/2019/2022)** — often not included; install it separately.

Install Microsoft's "Evergreen Standalone Installer"
(`MicrosoftEdgeWebView2RuntimeInstallerX64.exe`) from
https://developer.microsoft.com/microsoft-edge/webview2/

## If something breaks

Errors are written to `mytune.log` next to the executable (the file only appears when something went wrong).

- **Bugs and questions** — open an [issue](https://github.com/Doni-Kim/mytune-release/issues).
  Please do not attach the log there: it holds no passwords, but it can contain server addresses and SQL text.
- **The log file**, or anything you would rather not post in public — mail it to **doniikim@gmail.com**.

## Built with

- .NET 11.0 (x64), C# 15, Blazor Hybrid
- MySqlConnector · Microsoft.Data.Sqlite · ClosedXML · Microsoft.Web.WebView2 · Microsoft.AspNetCore.Components.WebView.WindowsForms
- Copyright notices and license texts of these bundled components: [THIRD-PARTY-NOTICES.txt](THIRD-PARTY-NOTICES.txt) (also inside the zip)

## Connection files (`mytuneNode1.json` …)

The zip ships two small templates, `mytuneNode1.json` (this PC — `localhost:3306`, `root`) and `mytuneNode2.json`
(another server, monitoring login `mytune`) — one file per server, and any name works (`prod.json`, `dev.json` …). With two or more next to `mytune.exe`, mytune asks
which one to use at startup — the list shows `user@server:port/schema`, never the password. With just one, it connects
straight away. The chosen file is that run's settings: the encrypted password, window position and theme are saved to it.
Only one mytune runs per folder; to watch several servers at the same time, use one folder per server.
Edit a file, or let the connection dialog fill it in:

```json
{
  "databases": [
    {
      "userId": "root",
      "password": "change-me",
      "server": "127.0.0.1",
      "port": "3306",
      "database": "",
      "sslMode": "preferred"
    }
  ],
  "interval": 5
}
```

- Write `password` in plain text — it is encrypted on the first run and stored back.
- `database` may stay empty — mytune watches the whole server.
- `sslMode`: `none` / `preferred` / `required` / `verify-ca` / `verify-full`. `sslCa`: the CA file (.pem) the verify modes trust — empty means the Windows certificate store. An unknown `sslMode` is reported instead of connecting.
- `interval` is the collection interval in seconds, 3–60 (5 when left out).
- Sections such as `alerts`, `topSql`, `logRetention` and `logFilter` (which sessions `L` logs) are optional —
  mytune fills them in with the defaults when it closes, and `O` edits them on screen. `mytune_sample_en.json` in the zip
  documents every setting.

## Terms

Free to use, at work or at home. Please do not redistribute the binary or reverse-engineer it.
The source is not published.

## Contact

DBMS Works — **doniikim@gmail.com**

Also available for Oracle → PostgreSQL / MySQL migration and database performance tuning work.
