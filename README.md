```
██████╗ ██████╗ ███████╗██╗    ██╗███████╗██╗   ██╗ ██████╗
██╔══██╗██╔══██╗██╔════╝██║    ██║██╔════╝██║   ██║██╔════╝
██████╔╝██████╔╝█████╗  ██║ █╗ ██║███████╗██║   ██║██║     
██╔══██╗██╔══██╗██╔══╝  ██║███╗██║╚════██║╚██╗ ██╔╝██║     
██████╔╝██║  ██║███████╗╚███╔███╔╝███████║ ╚████╔╝ ╚██████╗
╚═════╝ ╚═╝  ╚═╝╚══════╝ ╚══╝╚══╝ ╚══════╝  ╚═══╝   ╚═════╝
```

[![arb](https://img.shields.io/badge/arb-package-05d9e8?style=flat-square)](https://github.com/MenkeTechnologies/arb)
![type](https://img.shields.io/badge/self--sourcing-dashboard-9b5de5?style=flat-square)
![license](https://img.shields.io/badge/license-MIT-ff2a6d?style=flat-square)

### `HOMEBREW BACKGROUND SERVICES AND THEIR RUN STATE, LIVE IN A TABLE`

> *"Every launchd-backed brew service, its status and PID, refreshed on its own clock."*

`brewsvc` is a live dashboard over `brew services list` — the Homebrew-managed background services on the machine and their current state (started, stopped, error) with owning user and PID. It is **self-sourcing**: it runs its own data command on a timer, so nothing needs to be piped into it. It is an `arb` package — a dashboard for [`arb`](https://github.com/MenkeTechnologies/arb), the pipeline-to-TUI language on the fusevm bytecode VM.

### [`arb`](https://github.com/MenkeTechnologies/arb) &middot; [`arb-registry`](https://github.com/MenkeTechnologies/arb-registry)

---

## Table of Contents

- [\[0x00\] Overview](#0x00-overview)
- [\[0x01\] Install](#0x01-install)
- [\[0x02\] Usage](#0x02-usage)
- [\[0x03\] The Spec](#0x03-the-spec)
- [\[0x04\] How It Works](#0x04-how-it-works)
- [\[0xFF\] License](#0xff-license)

---

## [0x00] OVERVIEW

`brewsvc` renders the output of `brew services list` as a live table so the state of every Homebrew-managed background service — status, user, and PID — is visible at a glance without re-running the command by hand. It refreshes on a fixed interval, making it a passive monitor for whether your local daemons (databases, brokers, web servers installed via Homebrew) are up, stopped, or errored.

## [0x01] INSTALL

```sh
arb install brewsvc   # from the arb-registry git index
arb search brewsvc
```

## [0x02] USAGE

```sh
arb brewsvc                       # run it (self-sourcing; no pipe needed)
arb brewsvc --serve               # browser dashboard
arb brewsvc --html > brewsvc.html # static HTML
```

## [0x03] THE SPEC

```arb
# brewsvc — Homebrew-managed background services and their state, self-sourced from `brew services list`.
! brew services list every 10s
table .brewsvc
source .brewsvc { in }
grid .brewsvc -row 0 -col 0
```

- `! brew services list every 10s` — the **self-source**: arb runs `brew services list` on its own every 10 seconds and feeds that command's stdout into the `.brewsvc` stream, so no external input is ever piped in.
- `table .brewsvc` — declares the **table widget** bound to the `.brewsvc` stream; it renders each line of the service listing as a row.
- `source .brewsvc { in }` — the **query pipeline** for the widget; `in` passes the incoming stream through unchanged, so the table shows the raw `brew services list` columns.
- `grid .brewsvc -row 0 -col 0` — **placement**: pins the `.brewsvc` table to row 0, column 0 of the dashboard grid.

## [0x04] HOW IT WORKS

The `!` self-source runs `brew services list` on its 10-second timer and pushes each run's stdout into the `.brewsvc` stream. That stream flows through the `source .brewsvc { in }` query pipeline, which shapes it (here, a pass-through) before the `table` widget renders the rows, positioned by `grid` at row 0, column 0. arb lowers the pipeline's compute to the fusevm bytecode VM with Cranelift JIT, so each refresh is executed as compiled bytecode rather than re-parsed script.

## [0xFF] LICENSE

MIT.
