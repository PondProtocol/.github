<h1 align="center">Pond Protocol</h1>

<p align="center">
  <em>A paired two-token design for the XRP Ledger &mdash; $PND as an issued currency, $rPND as a Multi-Purpose Token.</em>
</p>

<p align="center">
  <img alt="Status: pre-issuance" src="https://img.shields.io/badge/status-pre--issuance-C2410C?style=flat-square">
  <img alt="XRP Ledger" src="https://img.shields.io/badge/XRP_Ledger-IOU_%2B_MPT-23292F?style=flat-square&logo=xrp&logoColor=white">
  <img alt="Standards" src="https://img.shields.io/badge/standards-XLS--26_%7C_XLS--89-1F6FEB?style=flat-square">
  <img alt="Node" src="https://img.shields.io/badge/Node-22%2B-5FA04E?style=flat-square&logo=nodedotjs&logoColor=white">
  <img alt="TypeScript" src="https://img.shields.io/badge/TypeScript-3178C6?style=flat-square&logo=typescript&logoColor=white">
  <img alt="License" src="https://img.shields.io/badge/license-Apache--2.0-556?style=flat-square">
</p>

---

## What this is

Pond Protocol is designing two related assets for the XRP Ledger. They are deliberately different ledger primitives rather than two copies of the same thing: **\$PND** is a classic issued currency (an IOU held over trust lines), and **\$rPND** is a Multi-Purpose Token (MPT), the newer XRPL object type with on-ledger metadata and explicit holder authorization. The organization holds three repositories: the protocol specification, a token-facing reference for each asset, and the issuance tooling that would configure an issuer, issue \$PND, and create \$rPND.

> [!IMPORTANT]
> **Neither token has been issued, on any network.** Everything in this organization is documentation and tooling. There is nothing to buy, hold, trade, or integrate, and any \$PND- or \$rPND-branded asset you encounter today did not come from us. See [What is on ledger today](#what-is-on-ledger-today).

## The two-token model

The table below describes how the two assets are **defined in [`rpnd/config/tokens.json`](https://github.com/PondProtocol/rPND/blob/main/config/tokens.json)** — the parameters the issuance transactions would apply if they were submitted. It is a statement of intent, not a description of the ledger — except the final row, which is a fact about mainnet as it stands.

| | **\$PND** | **\$rPND** |
| --- | --- | --- |
| Ledger type | Issued currency (IOU) | Multi-Purpose Token (MPT) |
| Identifier | Currency code `PND` **plus** the [issuer address](#what-is-on-ledger-today) — the pair is the identity; the code alone is not | Ticker `RPND`, plus the `MPTokenIssuanceID` that creation would assign |
| Holder opt-in | `TrustSet` to the issuer | `MPTokenAuthorize` |
| Amount shape | `{ currency, issuer, value }` | `{ mpt_issuance_id, value }` |
| Decimals / scale | 6 display decimals, tick size 5 | `AssetScale` 6 |
| Metadata | XLS-26 `xrp-ledger.toml`, to be linked from the issuer `Domain` | XLS-89 JSON, hex-encoded into `MPTokenMetadata` at creation (1024-byte cap) |
| Flags the config would set | Default Ripple, Disallow XRP, transfer rate 0 | Transferable, lockable, trading not enabled, and **clawback permanently disabled** via `ImmutableFlags` |
| How supply works | No ledger cap; the outstanding amount is whatever the issuer owes across its trust lines | `MaximumAmount`, fixed at creation and enforced by the ledger |
| Tradable once issued, on mainnet as it stands | Yes — DEX order books and AMM pools | **No** — mainnet rejects MPT trading outright ([why](#what-mainnet-supports-today)) |

Supply works in opposite directions for the two, which is worth understanding before either exists. The XRP Ledger stores no supply figure for an issued currency, so \$PND's **target supply of 100,000,000,000** is an issuer policy target rather than anything the ledger would enforce; the outstanding amount at any moment is simply the sum of what the issuer owes. \$rPND is the reverse: an MPT's `MaximumAmount` is fixed at creation and enforced by the protocol, but the figure itself is still an open decision. The value in `config/tokens.json` is an unfrozen working default and should not be quoted as \$rPND's supply.

Both assets are intended to be issued from one cold account. The \$rPND metadata would record `paired_iou_currency = "PND"` so indexers and operators can see the relationship — but that is documentation, not a ledger guarantee. The XRPL does not atomically bind an IOU to an MPT, and the two would not be interchangeable on ledger.

\$rPND also requires a network with MPT support. The config marks Devnet and Mainnet as MPT-capable and Testnet as not — see [What mainnet supports today](#what-mainnet-supports-today) for what that does and does not mean in practice.

## What is on ledger today

Nothing.

The issuing account is **`rPNDRmfNNrUZstkA23haCUkCp7qLEPnaYc`** on mainnet. It is funded, and that is all. **It has not been configured and has issued nothing.** No account flags are set, so there is no `Domain`, no `TransferRate`, and no `TickSize`. It owns no ledger objects, which means no trust lines, no \$PND obligations outstanding, and no \$rPND issuance. On Devnet and Testnet the account does not exist at all. Funding an account is not a launch; it is the prerequisite to one.

So every parameter in the table above is a configured intention that takes effect only when the issuance transactions are actually submitted, and none of them have been.

### Account topology

That one account issues **both** tokens — \$PND and \$rPND share a single issuer. Two further accounts complete the design, and all three addresses are published here, each with what it is on the ledger right now:

| Role | Address | On ledger now |
| --- | --- | --- |
| **Issuer** — signs for both \$PND and \$rPND | `rPNDRmfNNrUZstkA23haCUkCp7qLEPnaYc` | Funded, unconfigured, has issued nothing |
| **Treasury** — will hold 90,000,000,000 \$PND in escrow | `rPNDcL2UrGtSoGwruWx6ocMQ6ey8uPZm2b` | **Does not exist yet** — `actNotFound` |
| **Operations** — will receive 10,000,000,000 \$PND for liquidity, and runs operations | `rPNDAwFzgXzsjvUbVWz1ErB28v9SkcR2in` | **Does not exist yet** — `actNotFound` |

Treasury and operations together account for the full 100,000,000,000 target. Neither has been funded, so neither appears on the ledger at all — an unfunded XRPL address is just a string until someone sends it XRP. These are the addresses that *will* hold the escrow and the liquidity once that happens, published now rather than later, for the reason given [below](#why-the-address-matters).

### Check it yourself

All three accounts are public ledger data, so none of the above needs to be taken on trust:

```bash
# the issuer
account_info     rPNDRmfNNrUZstkA23haCUkCp7qLEPnaYc   # account flags, Domain, TransferRate, TickSize
account_lines    rPNDRmfNNrUZstkA23haCUkCp7qLEPnaYc   # trust lines — who holds $PND
account_objects  rPNDRmfNNrUZstkA23haCUkCp7qLEPnaYc   # MPT issuance objects — whether $rPND exists
gateway_balances rPNDRmfNNrUZstkA23haCUkCp7qLEPnaYc   # outstanding obligations — issued $PND supply

# treasury and operations — both should come back actNotFound
account_info     rPNDcL2UrGtSoGwruWx6ocMQ6ey8uPZm2b
account_info     rPNDAwFzgXzsjvUbVWz1ErB28v9SkcR2in
```

As things stand, the issuer returns an unconfigured account — no flags set, no trust lines, no objects, no obligations — and the other two return `actNotFound`. If any of that stops being true, this page is out of date: believe the ledger, not the page. Deliberately quoted nowhere here are balances, ledger indexes, and sequence numbers, because they go stale; run the queries against a current validated ledger instead.

### Why the address matters

The currency code `PND` is not exclusive to us, and that is the practical reason the address is published here rather than held back. Any XRPL account can issue a token under any code, and several already do: other mainnet accounts issue tokens under the exact code `PND`, and others under near-identical variants such as `Pnd`, `PNDN`, and `PNDC`. An IOU's identity is the pair *(currency code, issuer address)* — the code on its own tells you nothing, and an interface that keys off the ticker alone will conflate all of them.

The same logic is why treasury and operations are listed above while they are still empty. A record published in advance is harder to argue with than one produced after the fact: if anyone later presents a different treasury or operations address as ours, it contradicts what was committed here at a point when those accounts provably held nothing.

> [!CAUTION]
> **Nothing trading as `PND` today is \$PND.** The issuing account above has issued nothing, on any network. Any `PND` token you can currently find on an exchange or in a DEX order book came from a different account and has no connection to this project. When \$PND is issued it will come from `rPNDRmfNNrUZstkA23haCUkCp7qLEPnaYc` and this page will say so. Until then, treat the address as a tool for refusing to buy the wrong token.

## What mainnet supports today

Checked against the live amendment set rather than assumed from config. A flat "mainnet supports MPTs" would be misleading in both directions, so the detail matters:

| Amendment | Mainnet | What it means for Pond Protocol |
| --- | --- | --- |
| `MPTokensV1` | Enabled | MPTs exist on mainnet, so an \$rPND issuance is possible in principle |
| `DynamicMPT` | **Not enabled** | The \$rPND config sets `ImmutableFlags`, which depends on this — see below |
| `Clawback` | Enabled | Available on the IOU side; whether \$PND would use it is undecided |
| `AMM`, `AMMClawback` | Enabled | AMM pools are available to IOUs, but not to MPTs — see below |
| `Escrow`, `TokenEscrow`, `fixTokenEscrowV1` | Enabled | Escrow of issued tokens is supported on the IOU side |

Three consequences follow, and they all matter for anyone waiting on these tokens.

**The \$rPND config as written would be rejected on mainnet today.** It sets `ImmutableFlags` to disable clawback permanently, and that field arrives with `DynamicMPT`, which is not enabled. Submitting the create transaction as currently configured would return `temDISABLED`. Either the config or the amendment set has to change before \$rPND can exist on mainnet.

**One irreversible step has a fixed order.** Because a single account issues both tokens, \$rPND's `MPTokenIssuanceCreate` has to be submitted *before* that issuer is ever blackholed: a blackholed account can never sign again, so blackholing first would permanently make \$rPND impossible at that address. Since the create would fail today regardless, blackholing is blocked until the config drops `ImmutableFlags` or `DynamicMPT` activates.

> [!WARNING]
> **MPTs cannot trade on mainnet at all.** `OfferCreate` and `AMMCreate` both return `temDISABLED` for an MPT, even when the issuance sets `CanTrade` — and the \$rPND config does not set it in any case. There is no order book and no AMM pool for an MPT today. If both tokens were issued right now, **\$PND, the IOU, would be the only tradable one of the two.** \$rPND could be held and transferred between holders, but not traded on ledger.

## Repositories

| Repo | What it is | Status |
| --- | --- | --- |
| [**protocol**](https://github.com/PondProtocol/Protocol) | The protocol itself — specification, architecture, and design decisions. Holds no keys and submits no transactions. | Draft PR open with architecture notes, a glossary, and an eight-file spec skeleton. Self-described pre-specification: sections record what the configuration already fixes and mark everything else as a numbered open question. |
| [**pnd**](https://github.com/PondProtocol/PND) | \$PND, the IOU — the token-facing reference for holders, wallets, exchanges, and indexers. | Draft PR open with the token spec, trust-line mechanics, integration notes, and a \$PND-versus-\$rPND comparison. Documentation only; the transactions that would issue \$PND live in `rpnd`. |
| [**rpnd**](https://github.com/PondProtocol/rPND) | \$rPND, the MPT — and the operator source of truth for on-ledger config: `config/tokens.json`, XLS-26/XLS-89 metadata, transaction builders, and an operator CLI. Apache-2.0. | Furthest along. The toolkit runs, builds its transactions offline, and is covered by a test suite and CI, with an opt-in live Devnet suite; a token-facing README, the \$rPND spec, and the rationale for choosing an MPT over an IOU are merged to `main`. |

All three are documentation and tooling. Nothing has been issued on any network, and none of these repositories has been audited.

## Where to start

**Reading about the tokens** — [`docs/tokens.md`](https://github.com/PondProtocol/rPND/blob/main/docs/tokens.md) defines the identity of each asset and [`docs/issuance.md`](https://github.com/PondProtocol/rPND/blob/main/docs/issuance.md) covers accounts, the issuance sequence, and metadata publication. For \$rPND specifically, [`docs/rpnd-spec.md`](https://github.com/PondProtocol/rPND/blob/main/docs/rpnd-spec.md) is the token spec and [`docs/mpt-vs-iou.md`](https://github.com/PondProtocol/rPND/blob/main/docs/mpt-vs-iou.md) explains why it is an MPT rather than a second IOU. The equivalent \$PND and protocol documents are still in the open pull requests linked above.

**Running it** — the toolkit needs Node 22+ and nothing else. Clone [`rpnd`](https://github.com/PondProtocol/rPND) and:

```bash
npm install
cp .env.example .env
npm test

# inspect the transactions without touching a network
npx tsx src/cli.ts dry-run
npx tsx src/cli.ts encode-metadata
```

`dry-run` and `encode-metadata` are offline, so they are the safest first look: you get the exact unsigned transactions and the XLS-89 blob before anything is submitted. From there, `npx tsx src/cli.ts fund` creates faucet-funded Devnet wallets and `configure-issuer` → `issue-pnd` → `issue-rpnd` → `status` walks the full sequence. `npx tsx src/cli.ts help` lists every command.

Devnet and Testnet are reset periodically by Ripple. Treat every seed those commands produce as disposable, and never reuse one on mainnet.

## Contributing

Issues and pull requests are welcome across all repositories. [`CONTRIBUTING.md`](https://github.com/PondProtocol/.github/blob/main/CONTRIBUTING.md) covers how to set up, what a reviewable change looks like, and the rules that apply specifically to token parameters. Participation is governed by our [Code of Conduct](https://github.com/PondProtocol/.github/blob/main/CODE_OF_CONDUCT.md).

Found a security problem? Please do not open a public issue — see the [Security Policy](https://github.com/PondProtocol/.github/blob/main/SECURITY.md).

<!--
  TODO (org owner) — nothing below exists yet, so no links are published.
  Fill these in and add them back to this file when they are real:
    - TODO: website / domain (also becomes ISSUER_DOMAIN and the host for /.well-known/xrp-ledger.toml)
    - TODO: X / Twitter handle
    - TODO: Discord or other community channel
    - TODO: contact email for general enquiries
    - TODO: $rPND MPTokenIssuanceID, once MPTokenIssuanceCreate actually succeeds.
           No addresses remain outstanding: issuer, treasury and operations are all published
           in "What is on ledger today" — see the MAINTENANCE note below before touching them.
    - TODO: final $rPND MaximumAmount. The config value is an unfrozen working default and is
           deliberately not quoted on this page; publish it only once the figure is settled.
    - TODO: how the $PND 100B target supply is enforced, if at all (issuer key policy,
           blackholing, or nothing). The target is stated above as policy, not as a ledger rule.
           ORDERING CONSTRAINT, and it is irreversible: the issuer signs for BOTH tokens, so
           $rPND's MPTokenIssuanceCreate must be submitted BEFORE any blackholing. A blackholed
           account can never sign again, so blackholing first makes $rPND impossible at that
           address, permanently and with no recovery. Today it is doubly blocked: the create
           would fail with temDISABLED anyway, because the config sets ImmutableFlags and
           DynamicMPT is not enabled on mainnet. So blackholing cannot be considered until
           either the config drops that field or the amendment activates — and even then, only
           after the create has succeeded.
  Do not add placeholder or "coming soon" links to the rendered page.

  MAINTENANCE: the account topology table pairs every address with its current on-ledger state,
  and that pairing is the whole point — an address published without its state implies
  infrastructure that does not exist. All three are published: issuer (funded, unconfigured,
  issued nothing), treasury and operations (both actNotFound). If any of them is funded,
  configured, or issues anything, update its row, the "As things stand" sentence under the
  query block, and the CAUTION callout in the SAME commit. An address and its stated state
  must never drift apart.

  MAINTENANCE: "What mainnet supports today" states live amendment status and will go stale.
  Re-check MPTokensV1 and DynamicMPT before any launch announcement. If DynamicMPT is enabled,
  the note that the $rPND create would be rejected stops being true; if MPT trading becomes
  possible on mainnet, the tradability warning must be revised or removed.
-->
