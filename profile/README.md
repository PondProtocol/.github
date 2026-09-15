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

The table below describes how the two assets are **defined in [`rpnd/config/tokens.json`](https://github.com/PondProtocol/rPND/blob/main/config/tokens.json)** — the parameters the issuance transactions would apply if they were submitted. It is a statement of intent, not a description of the ledger.

| | **\$PND** | **\$rPND** |
| --- | --- | --- |
| Ledger type | Issued currency (IOU) | Multi-Purpose Token (MPT) |
| Identifier | Currency code `PND` + issuer address (address not published) | Ticker `RPND`, plus the `MPTokenIssuanceID` that creation would assign |
| Holder opt-in | `TrustSet` to the issuer | `MPTokenAuthorize` |
| Amount shape | `{ currency, issuer, value }` | `{ mpt_issuance_id, value }` |
| Decimals / scale | 6 display decimals, tick size 5 | `AssetScale` 6 |
| Metadata | XLS-26 `xrp-ledger.toml`, to be linked from the issuer `Domain` | XLS-89 JSON, hex-encoded into `MPTokenMetadata` at creation (1024-byte cap) |
| Flags the config would set | Default Ripple, Disallow XRP, transfer rate 0 | Transferable, lockable, and **clawback permanently disabled** via `ImmutableFlags` |
| How supply works | No ledger cap; the outstanding amount is whatever the issuer owes across its trust lines | `MaximumAmount`, fixed at creation and enforced by the ledger |

Supply works in opposite directions for the two, which is worth understanding before either exists. The XRP Ledger stores no supply figure for an issued currency, so \$PND's **target supply of 100,000,000,000** is an issuer policy target rather than anything the ledger would enforce; the outstanding amount at any moment is simply the sum of what the issuer owes. \$rPND is the reverse: an MPT's `MaximumAmount` is fixed at creation and enforced by the protocol, but the figure itself is still an open decision. The value in `config/tokens.json` is an unfrozen working default and should not be quoted as \$rPND's supply.

Both assets are intended to be issued from one cold account. The \$rPND metadata would record `paired_iou_currency = "PND"` so indexers and operators can see the relationship — but that is documentation, not a ledger guarantee. The XRPL does not atomically bind an IOU to an MPT, and the two would not be interchangeable on ledger.

\$rPND also requires a network with the MPTokens amendment enabled. The config marks Devnet and Mainnet as MPT-capable and Testnet as not; the mainnet flag has not been checked against live amendment status.

## What is on ledger today

Nothing.

The issuer account exists and is funded on mainnet, but **it has not been configured and has issued nothing**. No account flags are set, so there is no `Domain`, no `TransferRate`, and no `TickSize`. It owns no ledger objects, which means no trust lines, no \$PND obligations outstanding, and no \$rPND issuance. On Devnet and Testnet the account does not exist at all. Funding an account is not a launch; it is the prerequisite to one.

So every parameter in the table above is a configured intention that takes effect only when the issuance transactions are actually submitted, and none of them have been.

The issuer address is deliberately not published here yet, precisely because publishing it now would imply a launch that has not happened. When it is published, you will be able to confirm all of the above yourself instead of taking this page's word for it:

```bash
account_info     <issuer>   # account flags, Domain, TransferRate, TickSize
account_lines    <issuer>   # trust lines — who holds $PND
account_objects  <issuer>   # MPT issuance objects — whether $rPND exists
gateway_balances <issuer>   # outstanding obligations — issued $PND supply
```

This page quotes no balances, ledger indexes, or sequence numbers, because they go stale. Run the queries against a current validated ledger and trust that instead.

## Repositories

| Repo | What it is | Status |
| --- | --- | --- |
| [**protocol**](https://github.com/PondProtocol/Protocol) | The protocol itself — specification, architecture, and design decisions. Holds no keys and submits no transactions. | Draft PR open with architecture notes, a glossary, and an eight-file spec skeleton. Self-described pre-specification: sections record what the configuration already fixes and mark everything else as a numbered open question. |
| [**pnd**](https://github.com/PondProtocol/PND) | \$PND, the IOU — the token-facing reference for holders, wallets, exchanges, and indexers. | Draft PR open with the token spec, trust-line mechanics, integration notes, and a \$PND-versus-\$rPND comparison. Documentation only; the transactions that would issue \$PND live in `rpnd`. |
| [**rpnd**](https://github.com/PondProtocol/rPND) | \$rPND, the MPT — and the operator source of truth for on-ledger config: `config/tokens.json`, XLS-26/XLS-89 metadata, transaction builders, and an operator CLI. Apache-2.0. | Furthest along. The toolkit runs, builds its transactions offline, and is covered by a test suite and CI, with an opt-in live Devnet suite; a token-facing README, the \$rPND spec, and the rationale for choosing an MPT over an IOU are merged to `main`. |

All three are documentation and tooling. Nothing has been issued on any network, no issuer address is published yet, and none of these repositories has been audited.

> If a link above 404s for you, that repository is still private.

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
    - TODO: mainnet issuer address + $rPND MPTokenIssuanceID, once issuance actually happens.
           The issuer account is funded on mainnet but entirely unconfigured, so the address is
           withheld on purpose: publishing it now would read as a launch announcement.
           When it does go in, update "What is on ledger today" in the same commit — that
           section and the address must never disagree.
    - TODO: final $rPND MaximumAmount. The config value is an unfrozen working default and is
           deliberately not quoted on this page; publish it only once the figure is settled.
    - TODO: how the $PND 100B target supply is enforced, if at all (issuer key policy,
           blackholing, or nothing). The target is stated above as policy, not as a ledger rule.
  Do not add placeholder or "coming soon" links to the rendered page.
-->
