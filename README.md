## The failures that don't crash — across ten languages, now smart contracts

I find the bugs where nothing crashes and nothing alerts: a value that satisfies
every check while being meaningless, code that passes its own test suite for the
wrong reason, data that has been quietly wrong for three weeks.

Twenty-plus of these are merged into projects I don't maintain — ten languages,
each fix carrying a test that reproduces the defect first. The same method now
applied to Solidity, Move, and Solana: an installable invariant/exploit-check library
plus security-PoC repos below — vulnerability classes, a written audit, invariant
fuzzing, proxy/upgrade takeovers, and Move/Solana authorization bugs — and a
competitive-audit researcher profile on
[Sherlock](https://audits.sherlock.xyz/watson/dngr2) and
[Cantina](https://cantina.xyz/u/dngr2).

The rest is backend and scraping: a block page returns `200 OK`, so every response
is classified before extraction and never stored as data; bots built around the
3am timeout; monitors that watch the value, not the timestamp. **Everything runs**
— most with a one-command demo, no key or signup.

---

## Deployable on-chain products

A line of production-grade Solidity infrastructure — the contracts most projects
have to deploy anyway, each built the way I audit: fee-on-transfer-safe, reentrancy-safe,
no admin path to user funds, and a **funds-conservation invariant** fuzzed over tens of
thousands of calls. Every repo is MIT, Foundry-tested, CI-green, and earns a **bounded,
capped** protocol fee — verifiable on-chain, not a rug. Deploy your own instance; the fee
is the only thing I take, and it's in the source.

| Product | What it is | Earns |
|---|---|---|
| [**v4-fee-hooks**](https://github.com/dngr2/v4-fee-hooks) | Uniswap v4 hook suite + factory/launchpad flywheel — dynamic/LVR fees, staking, referral | protocol cut per swap + launch fee |
| [**onchain-escrow**](https://github.com/dngr2/onchain-escrow) | Milestone escrow with arbiter dispute resolution — marketplace/freelance settlement | fee per release |
| [**yield-vault**](https://github.com/dngr2/yield-vault) | ERC-4626 vault, inflation-attack-safe, mgmt + high-water performance fees | mgmt + perf fee |
| [**token-tools**](https://github.com/dngr2/token-tools) | The launch kit: merkle airdrop, vesting/locker, multisend, payment splitter | fee per use |
| [**token-factory**](https://github.com/dngr2/token-factory) | ERC-20 launcher: trustworthy capped/mintable/permit token + a bounded-fee token | fee per launch |
| [**token-sale**](https://github.com/dngr2/token-sale) | Fixed-price presale/IDO: soft/hard caps, refund-on-fail, purchase vesting, allowlist | fee on raise |
| [**nft-launchpad**](https://github.com/dngr2/nft-launchpad) | NFT collection minter: phased allowlist/public mint, ERC-2981 royalties, reveal lock | fee per mint |
| [**staking-rewards**](https://github.com/dngr2/staking-rewards) | Synthetix-style liquidity mining, checkpoint-before-balance, solvency-checked notify | fee per pool |
| [**paymaster**](https://github.com/dngr2/paymaster) | ERC-4337 gas sponsorship (v0.7): verifying (sponsored) + token (pay gas in ERC-20) | deposit + markup |
| [**payment-streaming**](https://github.com/dngr2/payment-streaming) | Linear+cliff token streams (payroll/grants), cancelable | fee per stream |
| [**subscriptions**](https://github.com/dngr2/subscriptions) | Recurring pull-payments, permissionless keeper charge, over-charge-proof | fee per charge |
| [**dao-factory**](https://github.com/dngr2/dao-factory) | One call → ERC20Votes token + Governor + Timelock, full lifecycle tested | fee per DAO |
| [**multisig-wallet**](https://github.com/dngr2/multisig-wallet) | m-of-n multisig, self-governed owner set, reentrancy-safe execution | fee per deploy |
| [**auction-house**](https://github.com/dngr2/auction-house) | English + Dutch NFT auctions, anti-snipe, pull-refunds | fee per sale |
| [**onchain-game-economy**](https://github.com/dngr2/onchain-game-economy) | EIP-712 capped reward vouchers, ERC-1155 items, marketplace, rate-limited treasury | fee per trade |

Safety isn't a claim — it's a suite: [**v4-hook-invariants**](https://github.com/dngr2/v4-hook-invariants)
and [**invariant-kit**](https://github.com/dngr2/invariant-kit) are the property tests these
products are checked against. You don't trust the author; you run the proof.

### More on-chain primitives

Non-custodial building blocks, same discipline: a **stateful solvency/conservation invariant**
fuzzed over 12,800 calls (proven non-hollow), a **mutation check** on the core math, CI-green, MIT.
Most are fee-less infrastructure — where a bounded fee exists it's named; a `—` means no protocol fee,
no admin path to funds.

| Primitive | What it is | Fee |
|---|---|---|
| [**stablecoin-cdp**](https://github.com/dngr2/stablecoin-cdp) | Single-collateral CDP: mint a stablecoin against collateral, RAY stability-fee index, penalty liquidation, debt ceiling | stability fee |
| [**savings-rate-vault**](https://github.com/dngr2/savings-rate-vault) | DSR-style ERC-4626: chi-index per-second accrual, reserve-backed (solvent by construction) | — |
| [**minimal-dex**](https://github.com/dngr2/minimal-dex) | Constant-product AMM (Pair/Factory/Router), k-invariant enforced, fee-on-transfer-safe | LP fee |
| [**dutch-auction**](https://github.com/dngr2/dutch-auction) | Descending-price sale (linear + convex), seller-favorable rounding, conservation invariants | — |
| [**bonding-curve-token**](https://github.com/dngr2/bonding-curve-token) | Linear bonding curve, exact-integral buy/sell, reserve-solvent, no risk-free round-trip | optional bounded |
| [**otc-order-desk**](https://github.com/dngr2/otc-order-desk) | EIP-712 signed OTC orders (EOA + EIP-1271), partial fills, non-custodial zero-escrow settlement | — |
| [**merkle-distributor**](https://github.com/dngr2/merkle-distributor) | Gas-efficient Merkle airdrop, bitmap-tracked claims, expiry sweep of unclaimed | — |
| [**merkle-vesting-airdrop**](https://github.com/dngr2/merkle-vesting-airdrop) | Proof-gated claim into a cliff+linear vesting grant, deadline sweep with backed-amount guard | — |
| [**vesting-wallet**](https://github.com/dngr2/vesting-wallet) | Multi-grant cliff+linear vesting, revocable grants with conservation, FoT-safe funding | — |
| [**payment-splitter**](https://github.com/dngr2/payment-splitter) | Pull-based split of ETH + ERC-20 among fixed immutable shares, reentrancy-safe | — |
| [**crowdfund-escrow**](https://github.com/dngr2/crowdfund-escrow) | All-or-nothing crowdfunding: creator claims only on goal-by-deadline, else backers refund | — |
| [**liquidity-locker**](https://github.com/dngr2/liquidity-locker) | Non-custodial LP/token locker, only-lengthen extension, no admin unlock | — |
| [**nft-staking**](https://github.com/dngr2/nft-staking) | Synthetix-style NFT staking: stake ERC-721 for per-second ERC-20 rewards, solvency-checked | — |
| [**timelock-controller**](https://github.com/dngr2/timelock-controller) | Governance delayed role-gated execution, batch ops, self-governed min-delay | — |
| [**soulbound-credentials**](https://github.com/dngr2/soulbound-credentials) | ERC-5192 non-transferable credential registry: issuer-attested, expiry + revocation + burn | — |
| [**oracle-aggregator**](https://github.com/dngr2/oracle-aggregator) | Defensive price oracle: Chainlink staleness + L2 sequencer-uptime + TWAP-deviation guards | — |
| [**deploy-tools**](https://github.com/dngr2/deploy-tools) | Front-run-safe CREATE2 deterministic factory + Multicall3-compatible aggregator | — |
| [**prediction-market**](https://github.com/dngr2/prediction-market) | Binary YES/NO market: complete-set full-collateral (solvent by construction, not an AMM), oracle-resolved | — |
| [**rentable-nft**](https://github.com/dngr2/rentable-nft) | ERC-4907 rentable NFT + escrow rental market: time-limited user role, renter can never own or transfer | rental fee |
| [**meta-tx-forwarder**](https://github.com/dngr2/meta-tx-forwarder) | ERC-2771 gasless meta-transactions: EIP-712 relayed requests, nonce/replay-safe, 63/64 gas guard | — |
| [**fixed-term-bond**](https://github.com/dngr2/fixed-term-bond) | Zero-coupon bond: buy at a discount, redeem par at maturity, fully reserve-backed so redemption is always honored | — |
| [**stream-nft**](https://github.com/dngr2/stream-nft) | Tokenized vesting stream: the stream is a tradeable ERC-721, per-stream accounting makes a mid-stream sale safe, cancelable | — |
| [**tranched-vault**](https://github.com/dngr2/tranched-vault) | Tranched structured product: senior (fixed return, protected) / junior (first-loss, keeps upside) waterfall over a yield source | — |
| [**multitoken-6909**](https://github.com/dngr2/multitoken-6909) | ERC-6909 minimal multi-token: per-id balances in one contract, operator + per-id allowance auth, supply-conservation invariant | — |
| [**flash-mint-token**](https://github.com/dngr2/flash-mint-token) | ERC-20 with ERC-3156 flash minting: mint/burn liquidity in-tx (no pool), bounded fee to treasury, supply-conservation invariant | flash fee |
| [**wrapped-native**](https://github.com/dngr2/wrapped-native) | Modern WETH: wrap native coin 1:1 into an ERC-20 with EIP-2612 permit, fully-backed invariant, no owner/fee | — |
| [**inheritance-vault**](https://github.com/dngr2/inheritance-vault) | On-chain dead-man's switch: owner checks in to keep control, beneficiary claims after the owner goes silent; non-custodial | — |
| [**allowance-module**](https://github.com/dngr2/allowance-module) | Treasury with per-spender rolling spending limits (refill each period), owner-revocable delegate caps | — |
| [**nft-fractionalize**](https://github.com/dngr2/nft-fractionalize) | Lock an ERC-721 into ERC-20 fractions; reclaim the NFT only by holding the entire supply (all-or-nothing redeem) | — |
| [**disperse**](https://github.com/dngr2/disperse) | Batch-send native coin + ERC-20 to many recipients in one tx (payroll/airdrop), exact-sum + refund, holds-no-value | — |

And a reference, **play-money** [**provably-fair-casino**](https://github.com/dngr2/provably-fair-casino) —
dice / coinflip / roulette / slots on a commit-reveal RNG, with bankroll-solvency invariants and
verified house edges. Published as an open-source *reference primitive*, not an operated service:
running it for real-money wagers is a licensing/KYC question for the deployer, stated plainly in the repo.

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

[**c-rack/cbor-java #265**](https://github.com/c-rack/cbor-java/pull/265) — the canonical map-key
comparator compared key bytes with Java's *signed* byte, so a byte `0xff` sorted before `0x01` —
the reverse of the byte order its own Javadoc specifies. Every canonical map with a high byte in a
key came out non-canonical. Merged same day.

[**c-rack/cbor-java #266**](https://github.com/c-rack/cbor-java/pull/266) — the decoder read a
declared array/map/string length and preallocated a collection of that size *before* reading a
single byte of payload. Five bytes — `9A 7F FF FF FF`, an array claiming 2,147,483,647 elements —
were enough to `OutOfMemoryError` the process. A parser is exactly where untrusted input arrives,
so the length has to be treated as a claim, not a promise; preallocation is now bounded by what the
stream can actually supply. Merged.

[**networknt/json-schema-validator #1273**](https://github.com/networknt/json-schema-validator/pull/1273)
— `uniqueItems` compared items through Jackson node equality, which is type-sensitive, so `[1, 1.0]`
validated. The official conformance suite covers the rule with `[1.0, 1.0, 1]`, which passes either
way, so 8,479 green tests and the rule still broken. Merged.

[**json-c/json-c #957**](https://github.com/json-c/json-c/pull/957) — strict mode rejected a
leading zero but checked nothing else about a number's shape, so `2.e3` and `1.` parsed clean. And
because json-c writes the token back out verbatim, it then *emitted* JSON that stricter parsers
reject — a library quietly manufacturing invalid documents. The fix enforces the whole RFC 8259
number grammar. Merged.

| Language | Where |
|---|---|
| Python | [sqlparse #876](https://github.com/andialbrecht/sqlparse/pull/876) · [keep #6687](https://github.com/keephq/keep/pull/6687) · [#6688](https://github.com/keephq/keep/pull/6688) · [#6689](https://github.com/keephq/keep/pull/6689) · [secretspec #352](https://github.com/cachix/secretspec/pull/352) **(merged)** · [herdr-mobile #1](https://github.com/bsorescu/herdr-mobile/pull/1) **(merged)** · [#2](https://github.com/bsorescu/herdr-mobile/pull/2) **(merged)** · [didwebvh-py #41](https://github.com/decentralized-identity/didwebvh-py/pull/41) · [jsoncanon #1](https://github.com/sveinugu/jsoncanon/pull/1) · [pathspec #130](https://github.com/cpburnz/python-pathspec/pull/130) · [areos-open #4](https://github.com/bsorescu/areos-open/pull/4) **(merged)** · [PyMySQL #1262](https://github.com/PyMySQL/PyMySQL/pull/1262) **(merged)** |
| Java | [json-schema-validator #1273](https://github.com/networknt/json-schema-validator/pull/1273) **(merged)** · [webauthn4j #1495](https://github.com/webauthn4j/webauthn4j/pull/1495) **(merged)** · [cbor-java #265](https://github.com/c-rack/cbor-java/pull/265) **(merged)** · [#266](https://github.com/c-rack/cbor-java/pull/266) **(merged)** · [semver4j #467](https://github.com/semver4j/semver4j/pull/467) · [json-schema-validator #1274](https://github.com/networknt/json-schema-validator/pull/1274) · [#1275](https://github.com/networknt/json-schema-validator/pull/1275) · [JSqlParser #2487](https://github.com/JSQLParser/JSqlParser/pull/2487) **(merged)** |
| Go | [go-retryablehttp #297](https://github.com/hashicorp/go-retryablehttp/pull/297) · [nanorix-verify #1](https://github.com/nanorix-io/nanorix-verify/pull/1) · [whatwg-url #46](https://github.com/nlnwa/whatwg-url/pull/46) · [gronx #71](https://github.com/adhocore/gronx/pull/71) **(merged)** · [#72](https://github.com/adhocore/gronx/pull/72) **(merged)** · [go-version #208](https://github.com/hashicorp/go-version/pull/208) · [pion/stun #286](https://github.com/pion/stun/pull/286) **(merged)** · [go-mysql #1184](https://github.com/go-mysql-org/go-mysql/pull/1184) |
| Rust | [secretspec #358](https://github.com/cachix/secretspec/pull/358) **(merged)** · [#353](https://github.com/cachix/secretspec/pull/353) **(merged)** |
| C++ | [tt-npe #131](https://github.com/tenstorrent/tt-npe/pull/131) · [utfcpp #144](https://github.com/nemtrif/utfcpp/issues/144) **(reported)** · [cpp-httplib #2540](https://github.com/yhirose/cpp-httplib/pull/2540) **(merged)** · [toml11 #317](https://github.com/ToruNiina/toml11/pull/317) |
| JavaScript | [PapaParse #1142](https://github.com/mholt/PapaParse/pull/1142) · [node-ignore #163](https://github.com/kaelzhang/node-ignore/pull/163) · [luxon #1799](https://github.com/moment/luxon/pull/1799) · [linebreak #53](https://github.com/foliojs/linebreak/pull/53) · [awesome-dsh #1201](https://github.com/awesome-dsh-plugin/awesome-dsh-plugin/pull/1201) **(merged)** · [#1277](https://github.com/awesome-dsh-plugin/awesome-dsh-plugin/pull/1277) **(merged)** · [fast-uri #212](https://github.com/fastify/fast-uri/pull/212) |
| C | [json-c #957](https://github.com/json-c/json-c/pull/957) **(merged)** · [tomlc17 #47](https://github.com/cktan/tomlc17/pull/47) **(merged)** |
| TypeScript | [keep #6698](https://github.com/keephq/keep/pull/6698) **(merged)** · [sql-formatter #964](https://github.com/sql-formatter-org/sql-formatter/pull/964) |
| C# | [CsvHelper #2387](https://github.com/JoshClose/CsvHelper/pull/2387) · [Cronos #95](https://github.com/HangfireIO/Cronos/pull/95) · [Tomlyn #135](https://github.com/xoofx/Tomlyn/pull/135) **(merged)** |
| Ruby | [secretspec #352](https://github.com/cachix/secretspec/pull/352) **(merged)** · [json-canonicalization #7](https://github.com/dryruby/json-canonicalization/pull/7) |
| PHP | [phpseclib #2165](https://github.com/phpseclib/phpseclib/pull/2165) **(merged via #2167/#2168)** · [#2167](https://github.com/phpseclib/phpseclib/pull/2167) **(merged)** · [#2168](https://github.com/phpseclib/phpseclib/pull/2168) **(merged)** · [composer/semver #184](https://github.com/composer/semver/pull/184) · [league/csv #586](https://github.com/thephpleague/csv/issues/586) **(reported)** |
| SystemVerilog | [axi_stream #8](https://github.com/pulp-platform/axi_stream/pull/8) · [#9](https://github.com/pulp-platform/axi_stream/pull/9) · [pulp-ethernet #6](https://github.com/pulp-platform/pulp-ethernet/pull/6) |
| SQL | [sqlparse #876](https://github.com/andialbrecht/sqlparse/pull/876) |
| Bash | [tt-installer #143](https://github.com/tenstorrent/tt-installer/pull/143) · [tt-system-tools #28](https://github.com/tenstorrent/tt-system-tools/pull/28) · [tt-flash #108](https://github.com/tenstorrent/tt-flash/pull/108) |

**The ones worth reading:**

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
and a field named `contains_pii` became one that doesn't exist. Merged.

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
serialized map starts with `kty`; it started with a negative label. Merged.

[**dryruby/json-canonicalization #7**](https://github.com/dryruby/json-canonicalization/pull/7) —
numbers were formatted with `"%.15E"`, which yields 16 significant digits, but an IEEE-754 double
needs 17 to round-trip. So `0.1 + 0.2` canonicalized to `"0.3"` — a string that parses back to a
*different* double, in a scheme whose entire purpose is that two parties hash identical bytes. The
maintainer had commented the failing cases out as "Outside Ruby Range"; they weren't — `5e-324` and
`Float::MAX` are perfectly representable, they just needed the 17th digit. 9,152 of 20,000 random
doubles came out wrong; the fix takes it to zero.

Also: [semver4j #467](https://github.com/semver4j/semver4j/pull/467) (`compareTo` threw
`NumberFormatException` on a spec-valid numeric prerelease identifier past `Long.MAX_VALUE`),
[gronx #71](https://github.com/adhocore/gronx/pull/71) (`0 0 31 * *` returned overflow dates — a
February 31st as March 4th — that its own matcher then rejects),
[luxon #1799](https://github.com/moment/luxon/pull/1799) (`fromISO` accepted out-of-range UTC
offsets like `+00:60` and `+24:00` and silently normalised them, while rejecting the same
out-of-range clock fields),
[pathspec #130](https://github.com/cpburnz/python-pathspec/pull/130) and
[node-ignore #163](https://github.com/kaelzhang/node-ignore/pull/163) (both matched POSIX classes
like `[[:digit:]]` against nothing, so Black / pre-commit / ESLint / Prettier disagreed with git on
which files are ignored),
[linebreak #53](https://github.com/foliojs/linebreak/pull/53) (missing Rule LB30's East Asian
exception, so a line break before a wide bracket like `〈` or `（` was wrongly forbidden),
[whatwg-url #46](https://github.com/nlnwa/whatwg-url/pull/46) (an opaque-path space before `?` or
`#` was not percent-encoded as the current URL Standard requires),
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

### 🧰 [invariant-kit](https://github.com/dngr2/invariant-kit)
Drop-in Foundry invariants and known-exploit checks for DeFi primitives — point a
module at your own contract and let Foundry fuzz the properties that matter, or run
a coded exploit check and get a failing transaction if the bug is present, not a
vague warning. `forge install dngr2/invariant-kit`. Four modules: ERC-4626 (solvency +
first-depositor inflation), staking (`notifyRewardAmount` restriction), constant-product
AMM (k-never-decreases + round-trip drain), and ERC-1967 proxy (unprotected-init +
storage-collision). Productizes the demo repos below into something a team actually installs.

### 🪝 [v4-hook-invariants](https://github.com/dngr2/v4-hook-invariants)
Property-based invariant testing for Uniswap v4 hooks — the new attack surface v4
core assumes a hook preserves but doesn't enforce. Drops a real `PoolManager` plus the
hook under test into a Foundry invariant harness and fuzzes swaps against reusable
predicates: a fee bound (a hook can't skim more than a fair share of swap output) and
claim/reserve solvency (LP-withdrawable reserves stay backed). Ships a fair reference
hook that passes and a 50% fee-skimmer the suite **catches** — a hook that plain v4
executes without complaint.

### ⚙️ [v4-fee-hooks](https://github.com/dngr2/v4-fee-hooks)
Three Uniswap v4 hook products that give pool users a service and earn a
**provably-bounded** on-chain fee — each verified against the fee bound from
[v4-hook-invariants](https://github.com/dngr2/v4-hook-invariants), so the cut it can
take is capped and checkable, not a rug. A **dynamic-fee** hook (raises the LP fee on
volatility, eases it when calm), an **auto-compounding vault** (reinvests LP fees for a
bounded cut of the *yield*), and an on-chain **limit-order** hook (single-sided
liquidity filled as the price crosses a tick, fee per fill). Foundry, six tests green.

### 🦀 [move-security-notes](https://github.com/dngr2/move-security-notes)
The same method carried to Move (Aptos): access control (bare `address` vs `&signer`),
precision (a dust deposit rounding to zero shares), resource-not-found DoS, and fee-bypass
rounding. Vulnerable/fixed pairs, each with an `aptos move test`.

### ◎ [solana-security-notes](https://github.com/dngr2/solana-security-notes)
And to native Solana: missing signer check and missing owner check (account
substitution) — the authorization footguns Solana leaves entirely to the program.
Vulnerable/fixed pairs with unit tests.

### 🛡 [contract-security-notes](https://github.com/dngr2/contract-security-notes)
Six ways a Solidity contract loses money while every line looks correct — vault
inflation, reentrancy, missing access control, spot-price oracle manipulation,
unchecked ERC20 returns, signature replay. Each is a vulnerable/fixed pair with a
Foundry test that **carries out the exploit on-chain** and shows the fix stops it.
The same idea as the rest of this work — the failure that doesn't revert — in Solidity.

```bash
forge test   # 12 tests: exploit + fix for each of the six findings
```

### 🔬 [staking-audit](https://github.com/dngr2/staking-audit)
A worked security review of a Synthetix-model `StakingRewards` protocol — the
deliverable a competitive audit actually asks for. Three planted bugs, each proven
by a Foundry PoC and written up in a severity-rated findings report: `stake()`
skips the reward checkpoint so a one-day staker walks off with a seven-day
staker's rewards (High), `notifyRewardAmount` is unauthenticated (Medium), reward
rounding strands dust (Low). See [`AUDIT.md`](https://github.com/dngr2/staking-audit/blob/main/AUDIT.md).

### 🎲 [invariant-fuzzing](https://github.com/dngr2/invariant-fuzzing)
Finding a bug the way a real audit does — state the property the system must
always hold and let Foundry hunt for a sequence that breaks it. A constant-product
AMM whose invariant (`reserve0 * reserve1` never decreases) holds over 4,096
fuzzed swaps on the fixed pool and fails on a buggy one that lets a trader
round-trip 100,000 into 122,000, draining the LPs.

### 🧬 [proxy-security](https://github.com/dngr2/proxy-security)
The two ways upgradeable contracts get taken over — where much of the real
high-severity money is, since almost every protocol runs behind a proxy. Against a
real ERC-1967 delegatecall proxy: an **unprotected initializer** lets an attacker
re-initialize and drain the vault, and a **storage collision** on upgrade (a V2
that prepends a field) makes `owner` read the old deposit total. Each with a
Foundry PoC and a fixed, append-only counterpart.

### 🕸 [stealth-scrape](https://github.com/dngr2/stealth-scrape)
Config-driven scraping for sites that don't want to be scraped. Adding a site is
a YAML file, not code. The core idea: a block page returns `200 OK`, so every
response is **classified before extraction** — WAF, captcha, rate-limit, empty
shell — and a layout change fails loudly instead of silently writing blanks.
Falls back to search-engine discovery when a site's own search is closed off.

```bash
python -m stealth_scrape --site olx --query "rtx 3090" --out results.csv
```

### 🏢 [fastapi-multitenant](https://github.com/dngr2/fastapi-multitenant)
Multi-tenant FastAPI + SQLAlchemy 2.0. Tenant isolation is enforced by a session-level
ORM event rather than per query — an event can't be forgotten by whoever adds a query
next year. The suite includes a mutation check: one test queries through an *unscoped*
session and asserts both tenants' rows **are** visible, proving the isolation tests can
actually fail. Migrations tested in both directions against a seeded database.

### 👁 [pagewatch](https://github.com/dngr2/pagewatch)
Alerts you won't mute. Every page differs on every fetch — timestamps, view
counters, session tokens — so it watches a **CSS-targeted value**, normalises
known noise, and never mistakes a WAF challenge for a change.

```bash
python -m pagewatch --demo   # baseline → noise (silent) → real change → blocked
```

### 🗡 [mu-server-architecture](https://github.com/dngr2/mu-server-architecture)
MMO server architecture in C# / .NET 8. Login, Game World and Chat are separate
assemblies — delete the World project and Chat still compiles. The demo shows the part
that matters: the client asks to teleport to 9999,9999 and the server refuses and sends
back the real position. Client-authoritative movement is the exploit most ready-made
server packs ship with.

### 🤖 [resilient-telegram-bot](https://github.com/dngr2/resilient-telegram-bot)
A bot built around what happens when the API *doesn't* answer. Transient and
permanent failures are separate types — a 502 retries, a 401 never does. Backoff
uses full jitter so retries don't land as one synchronised stampede. Exhausting
the retry budget logs and continues; only a permanent error exits.

```bash
python -m bot.main --demo    # no token — watch it recover from a flaky API
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

**Python · Rust · Go · TypeScript · JavaScript · Java · C++ · C# / .NET · PHP · Ruby · Solidity · SQL · Bash · SystemVerilog**

**FastAPI · SQLAlchemy · Selenium · Linux · PostgreSQL · Docker · Maven · Cargo · CMake**

Available for backend and systems work — automation, services, data plumbing,
scraping and monitoring. Most at home in Python, but the table above is the
honest answer to "can you work in X": each row is a defect found and fixed in
someone else's codebase, not a line on a skills list.
Send me a URL and I'll tell you whether it's extractable before you commit to anything.
