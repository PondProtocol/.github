<!--
  Thanks for contributing. Delete any section that does not apply.
  Security fix? Do not describe the vulnerability here — see SECURITY.md first.
-->

## What this changes

<!-- A short description, and the issue it closes (e.g. "Closes #12"). -->

## Why

<!-- The reason, if it is not obvious from the description. -->

## How it was verified

<!--
  For changes to transaction building or metadata, paste the relevant output:
    npx tsx src/cli.ts dry-run
    npx tsx src/cli.ts encode-metadata
-->

- [ ] `npm test` passes
- [ ] `npm run typecheck` passes
- [ ] Tests added or updated for behaviour changes
- [ ] Docs in `docs/` updated if this makes them wrong

## Risk

- [ ] This changes on-ledger behaviour (transactions built or submitted)
- [ ] This changes token metadata (XLS-26 or XLS-89)
- [ ] This touches key or secret handling
- [ ] This adds or upgrades a dependency
- [ ] This changes `config/tokens.json`

<!--
  If you ticked config/tokens.json, state for each field:
    1. which field, from what to what
    2. whether it is fixed at creation time or updatable later
    3. what it means for issuances that already exist
-->

- [ ] No seeds, keys, or `.env` contents are included in this PR or its output
