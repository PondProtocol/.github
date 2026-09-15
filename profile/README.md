<h1 align="center">Pond Protocol</h1>

<p align="center">
  <em>A paired two-token system on the XRP Ledger &mdash; $PND as an issued currency, $rPND as a Multi-Purpose Token.</em>
</p>

<p align="center">
  <img alt="XRP Ledger" src="https://img.shields.io/badge/XRP_Ledger-IOU_%2B_MPT-23292F?style=flat-square&logo=xrp&logoColor=white">
  <img alt="Standards" src="https://img.shields.io/badge/standards-XLS--26_%7C_XLS--89-1F6FEB?style=flat-square">
  <img alt="Node" src="https://img.shields.io/badge/Node-22%2B-5FA04E?style=flat-square&logo=nodedotjs&logoColor=white">
  <img alt="TypeScript" src="https://img.shields.io/badge/TypeScript-3178C6?style=flat-square&logo=typescript&logoColor=white">
  <img alt="License" src="https://img.shields.io/badge/license-Apache--2.0-556?style=flat-square">
</p>

---

## What this is

Pond Protocol issues and operates two related assets on the XRP Ledger. They are deliberately different ledger primitives rather than two copies of the same thing: **\$PND** is a classic issued currency (an IOU held over trust lines), and **\$rPND** is a Multi-Purpose Token (MPT), the newer XRPL object type with on-ledger metadata and explicit holder authorization. The organization holds three repositories: the protocol specification, a token-facing reference for each asset, and the issuance tooling that configures an issuer, issues \$PND, and creates \$rPND. Everything here is documentation and tooling. Issuance is exercised against Devnet by default; there is no mainnet issuance yet.

## The two-token model

| | **\$PND** | **\$rPND** |
| --- | --- | --- |
| Ledger type | Issued currency (IOU) | Multi-Purpose Token (MPT) |
| Identifier | Currency code `PND` + issuer address | Ticker `RPND`, plus the `MPTokenIssuanceID` returned at creation |
| Holder opt-in | `TrustSet` to the issuer | `MPTokenAuthorize` |
| Amount shape | `{ currency, issuer, value }` | `{ mpt_issuance_id, value }` |
| Decimals / scale | 6 display decimals, tick size 5 | `AssetScale` 6 |
| Metadata | XLS-26 `xrp-ledger.toml`, linked from the issuer `Domain` | XLS-89 JSON, hex-encoded into `MPTokenMetadata` (1024-byte cap) |
| Notable flags | Default Ripple on, Disallow XRP on, transfer rate 0 | Transferable, lockable, **clawback permanently disabled** at creation |
| Supply | No ledger cap; outstanding amount is the issuer's obligations | `MaximumAmount`, fixed at creation and enforced by the ledger |

Supply works in opposite directions for the two. The XRP Ledger stores no supply figure for an issued currency, so \$PND's **target supply of 100,000,000,000** is an issuer policy target rather than anything the ledger enforces — the outstanding amount at any moment is simply the sum of what the issuer owes across its trust lines. \$rPND is the reverse: an MPT's `MaximumAmount` is fixed at creation and enforced by the protocol, but the figure itself is still an open decision. The value currently in `config/tokens.json` is an unfrozen working default used for Devnet rehearsal and should not be quoted as \$rPND's supply.

Both assets share one cold issuing account. The \$rPND metadata records `paired_iou_currency = "PND"` so indexers and operators can see the relationship — but that is documentation, not a ledger guarantee. The XRPL does not atomically bind an IOU to an MPT, and the two are not interchangeable on ledger.

\$rPND also requires a network with the MPTokens amendment enabled. Devnet and Mainnet qualify; Testnet does not, and is flagged accordingly in config.

## Repositories

| Repo | What it is | Status |
| --- | --- | --- |
| [**protocol**](https://github.com/PondProtocol/Protocol) | The protocol itself — specification, architecture, and design decisions. Holds no keys and submits no transactions. | Draft PR open with architecture notes, a glossary, and an eight-file spec skeleton. Self-described pre-specification: sections record what is already true on ledger and mark everything else as a numbered open question. |
| [**pnd**](https://github.com/PondProtocol/PND) | \$PND, the IOU — the token-facing reference for holders, wallets, exchanges, and indexers. | Draft PR open with the token spec, trust-line mechanics, integration notes, and a \$PND-versus-\$rPND comparison. Documentation only; the transactions that issue \$PND run from `rpnd`. |
| [**rpnd**](https://github.com/PondProtocol/rPND) | \$rPND, the MPT — and the operator source of truth for on-ledger config: `config/tokens.json`, XLS-26/XLS-89 metadata, transaction builders, and an operator CLI. Apache-2.0. | Furthest along. The toolkit is working and Devnet-exercised, with tests and CI; a token-facing README, the \$rPND spec, and the rationale for choosing an MPT over an IOU are merged to `main`. |

All three are documentation and tooling. Nothing has been issued on mainnet, no issuer address is published yet, and none of these repositories has been audited.

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
    - TODO: mainnet issuer address + $rPND MPTokenIssuanceID, once issuance actually happens
           (issuer address deliberately withheld until the account's funding status is confirmed)
    - TODO: final $rPND MaximumAmount. The config value is an unfrozen working default and is
           deliberately not quoted on this page; publish it only once the figure is settled.
    - TODO: how the $PND 100B target supply is enforced, if at all (issuer key policy,
           blackholing, or nothing). The target is stated above as policy, not as a ledger rule.
  Do not add placeholder or "coming soon" links to the rendered page.
-->
