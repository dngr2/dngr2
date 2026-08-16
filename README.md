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

Pull requests into projects I don't maintain. Same rule in each: a test that
reproduces the defect **before** the fix, so the suite proves the fix does something.
Every one below fails against the unpatched code.

### Merged

[**cachix/secretspec #358**](https://github.com/cachix/secretspec/pull/358) — a JSON `null` in a
secret reference became the four-character password `null`, satisfying a required secret. The
maintainer asked for the three copies of that rendering to be shared; doing so surfaced the same
defect in a third call site, and the existing suite then caught me flattening a difference between
them that was deliberate and tested. Merged 119 minutes after opening.

[**cachix/secretspec #352**](https://github.com/cachix/secretspec/pull/352) — `close()` deletes the
temp files holding `as_path` secrets. Python and Ruby stopped at the first file the OS refused,
stranding every later secret on disk. Go and .NET already recorded the error and cleaned up the
rest — the project's own contract, unimplemented in two of six SDKs.

[**bsorescu/herdr-mobile #1**](https://github.com/bsorescu/herdr-mobile/pull/1) and
[**#2**](https://github.com/bsorescu/herdr-mobile/pull/2) — a redraw skipped when it would be
identical, and the polling SSH calls moved off the event loop. Both came back with changes
requested, and the review was the useful part: the maintainer showed that two of my tests passed
with *and* without their fix, which is the one thing a regression test must never do. Fixed, plus
a third bug the re-check turned up — the skip path never re-pinned the log, so the last rows sat
under the remote bar indefinitely.

| Language | Where |
|---|---|
| Java | [json-schema-validator #1273](https://github.com/networknt/json-schema-validator/pull/1273) · [webauthn4j #1495](https://github.com/webauthn4j/webauthn4j/pull/1495) · [cbor-java #265](https://github.com/c-rack/cbor-java/pull/265) |
| C++ | [tt-npe #131](https://github.com/tenstorrent/tt-npe/pull/131) |
| Rust | [secretspec #358](https://github.com/cachix/secretspec/pull/358) **(merged)** |
| Go | [go-retryablehttp #297](https://github.com/hashicorp/go-retryablehttp/pull/297) · [nanorix-verify #1](https://github.com/nanorix-io/nanorix-verify/pull/1) |
| TypeScript | [keep #6698](https://github.com/keephq/keep/pull/6698) |
| Python | [sqlglot #8192](https://github.com/tobymao/sqlglot/pull/8192) · [sqlparse #876](https://github.com/andialbrecht/sqlparse/pull/876) · [keep #6687](https://github.com/keephq/keep/pull/6687) · [#6688](https://github.com/keephq/keep/pull/6688) · [#6689](https://github.com/keephq/keep/pull/6689) · [secretspec #352](https://github.com/cachix/secretspec/pull/352) **(merged)** · [herdr-mobile #1](https://github.com/bsorescu/herdr-mobile/pull/1) **(merged)** · [#2](https://github.com/bsorescu/herdr-mobile/pull/2) **(merged)** · [didwebvh-py #41](https://github.com/decentralized-identity/didwebvh-py/pull/41) · [jsoncanon #1](https://github.com/sveinugu/jsoncanon/pull/1) |
| C# | [CsvHelper #2387](https://github.com/JoshClose/CsvHelper/pull/2387) |
| Ruby | [secretspec #352](https://github.com/cachix/secretspec/pull/352) **(merged)** · [json-canonicalization #7](https://github.com/dryruby/json-canonicalization/pull/7) |
| PHP | [phpseclib #2165](https://github.com/phpseclib/phpseclib/pull/2165) |
| JavaScript | [PapaParse #1142](https://github.com/mholt/PapaParse/pull/1142) |
| SQL | [sqlglot #8192](https://github.com/tobymao/sqlglot/pull/8192) · [sqlparse #876](https://github.com/andialbrecht/sqlparse/pull/876) |
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

[**hashicorp/go-retryablehttp #297**](https://github.com/hashicorp/go-retryablehttp/pull/297) —
`Retry-After` seconds are converted with `time.Second * time.Duration(sleep)`. A `Duration` counts
nanoseconds, so any value past ~292 years wraps negative, and `time.NewTimer` fires immediately on a
negative duration. A server asking to be left alone was answered with a burst of retries. The guard
for a negative value in the header already existed; a positive one that *becomes* negative did not.

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

[**tobymao/sqlglot #8192**](https://github.com/tobymao/sqlglot/pull/8192) — the tokenizer
decoded only fixed two-character escapes, so Postgres' `e'\x41'` was carried as the four-character
text `\x41` instead of `A`. Three consequences from one cause: the value changed silently on the
way to eight dialects, the literal **vanished** for seventeen others (`SELECT E'hello'` generated
`SELECT `), and a backslash was lost round-tripping Postgres to itself — `e'C:\\tmp\\new'` came
back as a path containing a tab and a newline. A maintainer had closed the previous attempt at
this as intractable; the escape set is finite and documented, so decoding it removes all three.
Values verified against DuckDB over 175 escape sequences ([writeup](https://github.com/tobymao/sqlglot/issues/8191)).

[**phpseclib/phpseclib #2165**](https://github.com/phpseclib/phpseclib/pull/2165) — `divide()`
returns the "common residue", the first positive modulo, so only a *negative* remainder has the
divisor added. When the division is exact the remainder is already zero, and the pure-PHP engines
added the divisor anyway: `-256 / 256` came back with a remainder of **256**, a residue equal to
its own modulus. GMP and BCMath return 0, so the answer depended on which extension happened to be
installed — and the PHP engines are the fallback when neither is, in a cryptography library. Found
by running 1278 operations through all three engines and diffing; 32 disagreed, and the documented
rule sided with BCMath in all 32. Their `testDivide` only ever divides a *positive* number exactly.

[**mholt/PapaParse #1142**](https://github.com/mholt/PapaParse/pull/1142) — the UTF-8 BOM was
stripped for string input only. A `File`, a download or a Node stream went straight to the chunk
parser, so the mark stayed inside the first field of the first row, where nothing renders it and
`row[0] === 'name'` is simply false. `header: true` hid it, because the header is stripped
separately. Both existing BOM tests pass a string, and the repo's own `utf-8-bom-sample.csv` is
only ever read *into* a string — the streaming path had no BOM coverage at all.

[**webauthn4j/webauthn4j #1495**](https://github.com/webauthn4j/webauthn4j/pull/1495) — the
CTAP2 canonical CBOR serializer listed an RSA COSE key's fields negatives-first and descending, so
a public key `{1:kty, 3:alg, -1:n, -2:e}` serialized with its keys ordered `-2, -1, 1, 3` instead of
the canonical `1, 3, -1, -2`. Canonical CBOR orders map keys by their encoded bytes — positive labels
before negative — and the sibling EC2 and EdDSA serializers already do, which is what proves the RSA
list is a mistake and not a convention. The output isn't canonical, so anything that re-encodes or
thumbprints the key diverges — in a WebAuthn/FIDO library. Built the 585★ project and asserted the
serialized map starts with `kty`; it started with a negative label.

[**dryruby/json-canonicalization #7**](https://github.com/dryruby/json-canonicalization/pull/7) —
numbers were formatted with `"%.15E"`, which yields 16 significant digits, but an IEEE-754 double
needs 17 to round-trip. So `0.1 + 0.2` canonicalized to `"0.3"` — a string that parses back to a
*different* double, in a scheme whose entire purpose is that two parties hash identical bytes. The
maintainer had commented the failing cases out as "Outside Ruby Range"; they weren't — `5e-324` and
`Float::MAX` are perfectly representable, they just needed the 17th digit. 9,152 of 20,000 random
doubles came out wrong; the fix takes it to zero.

Also: [cbor-java #265](https://github.com/c-rack/cbor-java/pull/265) (canonical map keys sorted with
Java's *signed* byte, so a key byte `0xff` sorted before `0x01` — the reverse of the byte order its
own Javadoc specifies; 326 tests all used ASCII keys under `0x80` and never hit the sign boundary),
[didwebvh-py #41](https://github.com/decentralized-identity/didwebvh-py/pull/41) (a DIF `did:webvh`
implementation hashed DID-log entries through a non-RFC-8785 canonicalizer that turned integers
`≥ 2^63` into JSON *strings*, so its SCIDs disagreed with any conformant verifier),
[nanorix-verify #1](https://github.com/nanorix-io/nanorix-verify/pull/1) (a Go JCS encoder stripped
the `+` from positive exponents and kept the padded zero in negatives — `1e21`→`"1e21"`,
`1e-7`→`"1e-07"` — breaking its own documented byte-equivalence to Rust `serde_jcs`),
[jsoncanon #1](https://github.com/sveinugu/jsoncanon/pull/1) (same 16-digit float bug, in Python;
their own test already expected the correct output, so the suite shipped red),
[keep #6687](https://github.com/keephq/keep/pull/6687) (results returned twice),
[#6688](https://github.com/keephq/keep/pull/6688) (`json.loads("123")` returns an int
without raising, so only dict/list parses are accepted),
[tt-system-tools #28](https://github.com/tenstorrent/tt-system-tools/pull/28) (hugepage
setup replaced the allocation instead of extending it),
[pulp-ethernet #6](https://github.com/pulp-platform/pulp-ethernet/pull/6) (receive path
never drove `TKEEP`, so a consumer read zero valid bytes in every frame),
[secretspec #358](https://github.com/cachix/secretspec/pull/358) (a JSON `null` became
the four-character password `null`),
[sqlparse #876](https://github.com/andialbrecht/sqlparse/pull/876) (`keyword_case` recased the
contents of a time zone literal, because the `AT TIME ZONE 'Asia/Tokyo'` rule matched the literal
as part of the keyword; the existing test used `'UTC'`, which reads the same either way),
[CsvHelper #2387](https://github.com/JoshClose/CsvHelper/pull/2387) (a UTF-8 BOM was
stripped only on the first buffer fill, so a file whose BOM straddled the boundary kept
it inside the first field — the tests for it passed with **and** without the fix, because
xUnit's collection comparer does not surface a leading U+FEFF).

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

**Python · Rust · Go · TypeScript · JavaScript · Java · C++ · C# / .NET · PHP · Ruby · SQL · Bash · SystemVerilog**

**FastAPI · SQLAlchemy · Selenium · Linux · PostgreSQL · Docker · Maven · Cargo · CMake**

Available for backend and systems work — automation, services, data plumbing,
scraping and monitoring. Most at home in Python, but the table above is the
honest answer to "can you work in X": each row is a defect found and fixed in
someone else's codebase, not a line on a skills list.
Send me a URL and I'll tell you whether it's extractable before you commit to anything.
