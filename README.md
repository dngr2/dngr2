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
reproduces the defect **before** the fix, so the suite proves the fix does something.
Every one below fails against the unpatched code.

| Language | Where |
|---|---|
| Java | [json-schema-validator #1273](https://github.com/networknt/json-schema-validator/pull/1273) |
| C++ | [tt-npe #131](https://github.com/tenstorrent/tt-npe/pull/131) |
| Rust | [secretspec #358](https://github.com/cachix/secretspec/pull/358) |
| TypeScript | [keep #6698](https://github.com/keephq/keep/pull/6698) |
| Python | [keep #6687](https://github.com/keephq/keep/pull/6687) · [#6688](https://github.com/keephq/keep/pull/6688) · [#6689](https://github.com/keephq/keep/pull/6689) · [secretspec #352](https://github.com/cachix/secretspec/pull/352) · [herdr-mobile #1](https://github.com/bsorescu/herdr-mobile/pull/1) · [#2](https://github.com/bsorescu/herdr-mobile/pull/2) |
| Ruby | [secretspec #352](https://github.com/cachix/secretspec/pull/352) |
| SystemVerilog | [axi_stream #8](https://github.com/pulp-platform/axi_stream/pull/8) · [#9](https://github.com/pulp-platform/axi_stream/pull/9) · [pulp-ethernet #6](https://github.com/pulp-platform/pulp-ethernet/pull/6) |
| Bash | [tt-installer #143](https://github.com/tenstorrent/tt-installer/pull/143) · [tt-system-tools #28](https://github.com/tenstorrent/tt-system-tools/pull/28) · [tt-flash #108](https://github.com/tenstorrent/tt-flash/pull/108) |

**The ones worth reading:**

[**json-schema-validator #1273**](https://github.com/networknt/json-schema-validator/pull/1273) — `uniqueItems`
compared items through Jackson node equality, which is type-sensitive, so `[1, 1.0]`
validated. The spec compares numbers mathematically. The official conformance suite
covers this rule with `[1.0, 1.0, 1]` — which **passes either way**, because the two
identical decimals are caught before an integer is ever compared against a decimal.
8,479 green tests, and the rule was still broken.

[**pulp-platform/axi_stream #9**](https://github.com/pulp-platform/axi_stream/pull/9) — a
64→8 AXI-Stream downsizer's fast path consumed a beat carrying `TLAST` but never
padded, so a frame one beat past a word boundary was silently truncated and its
remainder left merged into the next frame. Only lengths ≡1 (mod 8) reach it, and only
with `TVALID` held high — a sweep of every length found it, a single test case would not.

[**cachix/secretspec #352**](https://github.com/cachix/secretspec/pull/352) — `close()`
deletes the temp files holding `as_path` secrets. Python and Ruby stopped at the first
file the OS refused, stranding every later secret on disk. Go and .NET already recorded
the error and cleaned up the rest — the project's own contract, unimplemented in two of
six SDKs.

[**tenstorrent/tt-npe #131**](https://github.com/tenstorrent/tt-npe/pull/131) — an empty
golden-cycle map left a `{Cycle::max(), 0}` sentinel that underflowed to `1` on
subtraction. Worse than a zero: `1` passes the `> 0` guard written to suppress exactly
that case, so a 100-cycle estimate was reported as 9900% error.

[**keephq/keep #6698**](https://github.com/keephq/keep/pull/6698) — a CEL filter was
translated to JavaScript with `replace(/contains/g, "includes")`, rewriting the inside
of quoted search strings. `description.contains("contains")` searched for *"includes"*,
and a field named `contains_pii` became one that doesn't exist.

Also: [keep #6687](https://github.com/keephq/keep/pull/6687) (results returned twice),
[#6688](https://github.com/keephq/keep/pull/6688) (`json.loads("123")` returns an int
without raising, so only dict/list parses are accepted),
[tt-system-tools #28](https://github.com/tenstorrent/tt-system-tools/pull/28) (hugepage
setup replaced the allocation instead of extending it),
[pulp-ethernet #6](https://github.com/pulp-platform/pulp-ethernet/pull/6) (receive path
never drove `TKEEP`, so a consumer read zero valid bytes in every frame),
[secretspec #358](https://github.com/cachix/secretspec/pull/358) (a JSON `null` became
the four-character password `null`).

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

**Python · Rust · TypeScript · Java · C++ · Ruby · C# / .NET · Bash · SystemVerilog**

**FastAPI · SQLAlchemy · Selenium · Linux · PostgreSQL · Docker · Maven · Cargo · CMake**

Available for Python and C# work — automation, backend services, dashboards,
scraping and monitoring.
Send me a URL and I'll tell you whether it's extractable before you commit to anything.
