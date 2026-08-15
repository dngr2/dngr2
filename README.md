## Python automation, scraping, and things that keep running

I build tools for the boring failure modes — the ones where nothing crashes,
nothing alerts, and the data has been quietly wrong for three weeks.

A scraper that returns `0 results` because it parsed a block page as data.
A bot that died on one API timeout at 3am. A monitor that alerts on the
timestamp instead of the price. Every repo below exists because of one of those.

**Everything here runs.** Most have a one-command demo that needs no key,
no signup, and no network.

---

### Contributing upstream

Open pull requests into projects I don't maintain. Same rule in each: a test that
reproduces the bug **before** the fix, so the suite proves the fix does something.

**[keephq/keep](https://github.com/keephq/keep)** — open-source AIOps and alert management (12.2k ★)

- [#6687](https://github.com/keephq/keep/pull/6687) — the HTTP provider returned every result twice.
  `BaseProvider.notify()` already appends the result, and calling `query()` appended it again.
- [#6688](https://github.com/keephq/keep/pull/6688) — a templated JSON body arrived as a string and got
  form-encoded. Parsing it is the fix, but only a `dict`/`list` parse is accepted:
  `json.loads("123")` returns an int without raising, which would turn a body into a number.
- [#6689](https://github.com/keephq/keep/pull/6689) — expose an instant PromQL query as a provider method.

**[cachix/secretspec](https://github.com/cachix/secretspec)** — declarative secrets manager, Rust (1.3k ★)

- [#352](https://github.com/cachix/secretspec/pull/352) — `Resolved.close()` deletes the temp files
  holding `as_path` secrets. In the Python and Ruby SDKs the first file the OS refused to remove
  aborted the loop, so **every later secret stayed on disk** — the one outcome the method exists to
  prevent — and the caller couldn't tell which. The Go and .NET SDKs already recorded the first error
  and cleaned up the rest, so this was the project's own contract, unimplemented in two of six SDKs.
  Ruby also silently skipped dangling symlinks: `File.exist?` follows links, so a broken one read as
  already-gone. Verified against both real compiled extensions — pyo3 via maturin, and the Ruby
  native extension.

**[tenstorrent](https://github.com/tenstorrent)** — AI accelerator hardware stack

- [tt-installer #143](https://github.com/tenstorrent/tt-installer/pull/143) — install `rustup` from the
  distro repos where it's packaged rather than curl-piping the upstream script. Version comparison via
  `sort -V`, with the tests written to fail if the comparison is inverted.
- [tt-flash #108](https://github.com/tenstorrent/tt-flash/pull/108) — implement `tt-flash verify`.
- [tt-system-tools #28](https://github.com/tenstorrent/tt-system-tools/pull/28) — hugepage setup
  *replaced* the existing allocation instead of adding to it, silently shrinking memory on a
  second run. Fixed with a marker file so the baseline survives a re-run.

**[bsorescu/herdr-mobile](https://github.com/bsorescu/herdr-mobile)** — phone-friendly Textual TUI for
driving coding agents over SSH

Two performance PRs, both led by measurement rather than by reading:

- [#1](https://github.com/bsorescu/herdr-mobile/pull/1) — the poll cleared and rewrote all 200 rows
  every 2s even when the agent's output was byte-identical: **20.35ms → 0.24ms**. The cache key has to
  be `(content, width)`, not content alone — there is no `on_resize` handler, so a rotate is only
  picked up *because* the next poll re-renders, and keying on content would have broken it silently.
- [#2](https://github.com/bsorescu/herdr-mobile/pull/2) — every CLI call was a
  `subprocess.run(timeout=10)` on Textual's event loop. Measured with a 10ms heartbeat, a 300ms call
  stalled the UI for **310ms**; a hung CLI would freeze it for the full 10s. Split each poll into a
  pure renderer and a `@work(thread=True, exclusive=True)` fetcher — **12ms worst case, and not one of
  the project's 174 existing tests had to change.**

I benchmarked the text pipeline first and left it alone: 430 `strip_ansi` calls looked like the
bottleneck and cost 1.98ms. The rewrite was ~90% of the time. Optimising the obvious thing would have
been effort spent on a non-problem.

---

### Repos

### 🕸 [stealth-scrape](https://github.com/dngr2/stealth-scrape)
Config-driven scraping for sites that don't want to be scraped. Adding a site is
a YAML file, not code. The core idea: a block page returns `200 OK`, so every
response is **classified before extraction** — WAF, captcha, rate-limit, empty
shell — and a layout change fails loudly instead of silently writing blanks.
Falls back to search-engine discovery when a site's own search is closed off.

```bash
python -m stealth_scrape --site olx --query "rtx 3090" --out results.csv
```

### 📦 [amazon-product-extraction](https://github.com/dngr2/amazon-product-extraction)
Real extraction output from live product pages, plus a write-up of four layout
traps that produce a scraper which looks correct on your test page and returns
nulls across a catalogue — including the price living in a visually-hidden node
that Selenium's `.text` returns empty for.

### 📈 [fastapi-trading-dashboard](https://github.com/dngr2/fastapi-trading-dashboard)
Real-time dashboard: FastAPI, WebSocket streaming, live candlestick charts,
dark/light themes. Instrument switching without reconnecting. Tabular figures so
columns don't jitter twice a second — the detail you only notice after sitting
in front of one for an hour.

```bash
pip install -r requirements.txt && uvicorn app.main:app
```

### 🤖 [resilient-telegram-bot](https://github.com/dngr2/resilient-telegram-bot)
A bot built around what happens when the API *doesn't* answer. Transient and
permanent failures are separate types — a 502 retries, a 401 never does. Backoff
uses full jitter so retries don't land as one synchronised stampede. Exhausting
the retry budget logs and continues; only a permanent error exits.

```bash
python -m bot.main --demo    # no token — watch it recover from a flaky API
```

### 👁 [pagewatch](https://github.com/dngr2/pagewatch)
Alerts you won't mute. Every page differs on every fetch — timestamps, view
counters, session tokens — so it watches a **CSS-targeted value**, normalises
known noise, and never mistakes a WAF challenge for a change.

```bash
python -m pagewatch --demo   # baseline → noise (silent) → real change → blocked
```


### 🏢 [fastapi-multitenant](https://github.com/dngr2/fastapi-multitenant)
Multi-tenant FastAPI + SQLAlchemy 2.0. Tenant isolation is enforced by a session-level
ORM event rather than per query — an event can't be forgotten by whoever adds a query
next year. The suite includes a mutation check: one test queries through an *unscoped*
session and asserts both tenants' rows **are** visible, proving the isolation tests can
actually fail. Migrations tested in both directions against a seeded database.

### 🗡 [mu-server-architecture](https://github.com/dngr2/mu-server-architecture)
MMO server architecture in C# / .NET 8. Login, Game World and Chat are separate
assemblies — delete the World project and Chat still compiles. The demo shows the part
that matters: the client asks to teleport to 9999,9999 and the server refuses and sends
back the real position. Client-authoritative movement is the exploit most ready-made
server packs ship with.

### 📊 [spreadsheet-consolidator](https://github.com/dngr2/spreadsheet-consolidator)
Merges spreadsheets that disagree, and **reports** the disagreements instead of silently
picking a winner. Handles the quiet data-loss cases: `"1,250.00"` parsed as a number
rather than becoming `NaN`, and whitespace stripped from text columns that a
`dtype == object` check would skip under pandas 2.x.

### ✉️ [org-email-resolver](https://github.com/dngr2/org-email-resolver)
Derives corporate email addresses from an organisation's naming convention, and is
explicit about what it cannot know — a derived address is a candidate, not a verified
mailbox.

---

**Python · C# / .NET · Bash · Selenium · FastAPI · SQLAlchemy · Linux · PostgreSQL · Docker**

Available for Python and C# work — automation, backend services, dashboards,
scraping and monitoring.
Send me a URL and I'll tell you whether it's extractable before you commit to anything.
