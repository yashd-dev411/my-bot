<div align="center">

# ⬡ my-bot

**Strategies, market data and a one-command Docker stack for [Algorithex](https://github.com/yashd-dev411/algorithex).**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Python](https://img.shields.io/badge/python-3.11%20%7C%203.12-3776AB)](https://www.python.org/)
[![Docker](https://img.shields.io/badge/docker-compose-2496ED?logo=docker&logoColor=white)](docker/docker-compose.yml)

</div>

---

## What this repo is

This is an **Algorithex project workspace**, not the trading framework itself.

It holds the parts that belong to one specific bot — your strategies, your
stored market data, and the compose stack that runs them — so they stay
version-controlled separately from the engine that executes them.

| | |
| --- | --- |
| **This repo** — `my-bot` | Your strategies, your data, your settings |
| **The framework** — [`yashd-dev411/algorithex`](https://github.com/yashd-dev411/algorithex) | The engine, dashboard and research modules |

Clone this repo, point it at the published framework image, and you have a
working trading stack. Nothing here needs to be edited to get started.

---

## What's inside

| Path | What it holds |
| --- | --- |
| `strategies/` | One directory per strategy, each with an `__init__.py`. The dashboard's **Strategies** tab discovers them from here |
| `storage/` | Generated candles, logs, charts and backtest results |
| `docker/` | The compose stack — app + PostgreSQL + Redis |
| `.env.example` | Every setting the stack reads, with the safe defaults |
| `AGENTS.md` | Rules for MCP-compatible AI assistants working on this project's strategies |

---

## Quick start

```bash
git clone https://github.com/yashd-dev411/my-bot.git my-bot
cd my-bot
cp .env.example .env
```

### 1. Point it at the published image

This step is easy to skip and hard to debug, so do it first.

The compose file declares a build context of `../../algorithex`, which only
resolves on a machine that already has the framework checked out **beside this
repo**. A fresh clone has no such directory, so compose tries to build and fails
with a missing-context error.

In `.env`, set the published image instead:

```bash
ALGORITHEX_IMAGE=ghcr.io/yashd-dev411/algorithex:latest
```

That image is public — no GitHub login is needed to pull it.

### 2. Start the stack

```bash
cd docker
docker compose up -d
docker compose ps
```

The dashboard is at **<http://localhost:9000>**. Sign in with the `PASSWORD`
value from `.env`.

`docker compose ps` is worth watching on the first run. The app waits for
PostgreSQL and Redis to report *healthy* before it starts, so you should see
them pass and only then the app come up. The first boot is slow because it
installs the workspace — the healthcheck allows 120 seconds for that before it
starts counting failures.

---

## The example strategy

`strategies/ExampleStrategy/` is a working EMA trend-following starter. It goes
long whenever the fast EMA is above the slow one, and sizes the position from a
risk percentage of the account rather than a fixed quantity.

```python
@property
@cached
def fast_ema(self):
    return ta.ema(self.candles, self.hp['fast_period'])

def should_long(self) -> bool:
    return self.fast_ema > self.slow_ema
```

It ships with five hyperparameters — `fast_period`, `slow_period`, `risk_pct`,
`take_profit_pct` and `stop_loss_pct` — so you have something real to optimize
in the dashboard rather than an empty framework to start from.

---

## Configuration

Everything is set in `.env` at the repository root. Two values matter before
anything can reach the dashboard:

| Variable | Why |
| --- | --- |
| `PASSWORD` | The dashboard login. Its sha256 is what the API compares, so changing it invalidates existing sessions |
| `POSTGRES_PASSWORD` | Set this if the database is reachable from anywhere but this machine |

The rest are local defaults, and the compose file falls back to the same
defaults if a variable is unset.

---

## Ports

| Port | Service |
| --- | --- |
| `9000` | Dashboard |
| `9001` | Language server (editor intelligence) |
| `9002` | MCP server — lets an AI assistant drive this project |
| `8888` | Jupyter |

---

## What is not committed

`.gitignore` keeps runtime artifacts out of history: `.env`, generated candles,
logs, charts, backtest results, and `docker/postgres-data/`. That last one is a
live PostgreSQL data directory holding files over 100 MB, which GitHub rejects
outright — it is recreated by `docker compose up`.

Your strategies and your configuration stay yours; the noise does not.

---

## Credits and License

Algorithex is a fork of [Jesse](https://github.com/jesse-ai/jesse) by **Jesse
Mir** and contributors, released under the MIT License. Upstream copyright is
retained in [`LICENSE`](LICENSE).

This workspace is distributed under the same terms. MIT permits use, modification
and redistribution provided the original copyright and permission notices are
kept intact — which is why they are still here.