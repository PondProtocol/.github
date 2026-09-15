# Contributing to Pond Protocol

Thanks for taking the time. This file applies to every repository in the [PondProtocol](https://github.com/PondProtocol) organization unless a repo ships its own `CONTRIBUTING.md`.

The project is early. Interfaces, token parameters, and repository boundaries are all still moving, so the most useful thing you can do before writing code is open an issue describing what you intend to change.

## Before you start

- **Search existing issues first.** If something is already tracked, comment there rather than opening a duplicate.
- **Open an issue for anything non-trivial.** Bug fixes and typo corrections can go straight to a pull request. Anything that changes behaviour, dependencies, or on-ledger parameters should be discussed first — it is much cheaper to disagree in an issue than in a review.
- **Security issues do not belong in the tracker.** Follow [SECURITY.md](SECURITY.md) instead.

## Development setup

The `rpnd` repository is the one with code today. It needs **Node 22 or newer** and has no other system dependencies.

```bash
git clone https://github.com/PondProtocol/rPND.git
cd rPND
npm install
cp .env.example .env
```

Before you push, both of these must pass:

```bash
npm test        # node:test suite, offline
npm run typecheck
```

`npm test` does not touch the network. The live Devnet suite is opt-in and separate:

```bash
npm run test:live   # sets RUN_XRPL_LIVE=1, requires a funded Devnet wallet
```

If you are changing transaction building or metadata encoding, please also paste the output of `npx tsx src/cli.ts dry-run` (and `encode-metadata` where relevant) into the pull request. Seeing the actual unsigned transaction is the fastest way for a reviewer to confirm a change is correct.

## Working with keys and networks

- **Never commit a seed, secret, or `.env` file.** `var/` is gitignored because faucet output lands there; keep it that way.
- Use **Devnet** for anything involving \$rPND. Testnet does not have the MPTokens amendment and is flagged `supportsMpt: false` in config.
- Devnet and Testnet are reset periodically. Assume every key from `npx tsx src/cli.ts fund` is disposable, and never reuse one on mainnet.
- If you believe you have exposed a key that holds anything, say so immediately — see [SECURITY.md](SECURITY.md).

## Changes to token parameters

`config/tokens.json` is the canonical definition of \$PND and \$rPND. Some of those values are **permanent once a token is created on a network**, in particular the MPT's `assetScale`, `maximumAmount`, and the immutable flag set that permanently disables clawback.

Pull requests touching that file need to spell out, in the description:

1. Which field changes, and from what to what.
2. Whether the field is fixed at creation time or can be updated later.
3. What it means for any issuance that already exists.

Reviewers will be deliberately slow here. That is intentional.

## Pull requests

- Branch from `main` and keep the change focused on one thing.
- Write commit messages in the imperative mood, with a short subject line and a body explaining *why* when the reason is not obvious.
- Match the surrounding code style. TypeScript, ES modules, explicit types on exported functions. There is no formatter or linter configured yet, so mirror the file you are editing.
- Add or update tests for behaviour changes. The existing suites in `test/` are a good template.
- Update the docs in `docs/` when your change makes them wrong. Stale documentation about issuance procedure is worse than none.
- Open the PR as a draft while it is still in progress, and mark it ready when CI is green.

CI runs the test suite and typecheck on every pull request. A red build will not be merged.

## Reporting bugs

Use the issue templates. The most useful bug reports include the command you ran, the network (`XRPL_NETWORK`), your Node version, what you expected, and the actual output or transaction result code. Redact addresses and seeds you do not want public — and never paste a seed at all.

## Code of Conduct

Participation in this project is governed by the [Code of Conduct](CODE_OF_CONDUCT.md). Please read it.

## Licensing

`rpnd` is Apache-2.0. Contributions are accepted under the license of the repository you are contributing to. There is no CLA.
