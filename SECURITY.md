# Security Policy

This policy applies to every repository in the [PondProtocol](https://github.com/PondProtocol) organization.

## Scope and current status

Pond Protocol is pre-mainnet. The code here builds and submits XRP Ledger transactions and is exercised against Devnet; it has **not** been independently audited, and there is no bug bounty program. Please treat it accordingly and do not run it against accounts holding real value without reviewing it yourself.

Only the current `main` branch of each repository is supported. There are no maintained release branches, and no backports to older tags.

## Reporting a vulnerability

**Please do not open a public issue, pull request, or discussion for a security problem.**

Report it privately instead:

1. **Preferred — GitHub private vulnerability reporting.** Open the affected repository, go to the **Security** tab, and choose **Report a vulnerability**. This creates a private advisory visible only to maintainers.
   > **TODO (org owner): enable Private Vulnerability Reporting for the organization (Settings → Code security) so this path actually works. Until then, only the email route below is available.**
2. **Email.**
   > **TODO (org owner): add a monitored security address here, for example `security@<your-domain>`, and publish the corresponding `[[PRINCIPALS]]` email in `xrp-ledger.toml`.**

If neither route is available to you yet, contact an organization owner directly through GitHub and ask for a private channel before sending any detail.

### What to include

A report is much easier to act on when it has:

- The repository, branch, and commit you tested.
- What an attacker can achieve, concretely — funds at risk, keys exposed, wrong transaction submitted, metadata forged.
- Reproduction steps, ideally a `dry-run` output or a failing test.
- The network involved (Devnet, Testnet, Mainnet) and your Node version.
- Any transaction hashes, with addresses redacted if you prefer.

**Never include a seed, secret key, or `.env` contents in a report.** If a key is already exposed, tell us that it is exposed and where, not the key itself.

## What we will do

We will acknowledge your report and keep you updated while we investigate. If the issue is valid we will work on a fix, credit you in the advisory unless you would rather stay anonymous, and coordinate disclosure timing with you. If we decide it is not a vulnerability, we will explain why.

> **TODO (org owner): commit to a concrete acknowledgement and fix-timeline target here once someone is actually on call for this inbox.**

Please give us a reasonable opportunity to fix the issue before disclosing it publicly.

## In scope

- Key handling: anything that logs, persists, transmits, or otherwise leaks `ISSUER_SEED`, `OPERATIONAL_SEED`, or `HOLDER_SEED`.
- Transaction construction: a builder that produces a transaction materially different from what the command claims, wrong amounts or destinations, missing or incorrect flags.
- Token parameter handling: anything that could cause an issuance to be created with wrong immutable values — `assetScale`, `maximumAmount`, or the flags that permanently disable clawback.
- Metadata: XLS-89 or XLS-26 encoding flaws that let metadata be forged, truncated, or mis-attributed.
- Supply chain: a malicious or compromised dependency reachable from our lockfile.

## Out of scope

- Vulnerabilities in the XRP Ledger protocol itself or in `rippled`. Report those through [Ripple's bug bounty](https://ripple.com/bug-bounty/), not here.
- Issues in third-party libraries with no exploitable path through this code. Report them upstream.
- Devnet or Testnet instability, faucet failures, or network resets.
- Loss of funds caused by using faucet or example keys on mainnet, which the documentation explicitly warns against.
- Missing security headers or configuration on any website not yet operated by this organization.
- Reports generated wholesale by automated scanners with no demonstrated impact.

## Impersonation and scams

We will never DM you asking for a seed, a private key, or a wallet connection, and we will never ask you to send funds to claim anything. Verify anything claiming to be from Pond Protocol against the repositories in this organization.

The issuing account is published on the [organization profile](https://github.com/PondProtocol), and it has issued nothing on any network. Check it directly with `account_lines` and `account_objects` rather than trusting a screenshot. Because no token exists yet, **anything currently presented as \$PND or \$rPND is fraudulent**, whatever ticker or address it carries — and note that the currency code `PND` is not exclusive, so unrelated tokens already use it and near-identical variants of it.
