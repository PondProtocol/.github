# .github

Organization-level defaults for [PondProtocol](https://github.com/PondProtocol).

Nothing here is a product. These files configure GitHub itself: the profile shown on the organization page, and the community health files that every other repository in the organization inherits unless it ships its own copy.

| Path | What it does |
| --- | --- |
| [`profile/README.md`](profile/README.md) | Rendered at [github.com/PondProtocol](https://github.com/PondProtocol) as the organization profile |
| [`CONTRIBUTING.md`](CONTRIBUTING.md) | Default contributing guide |
| [`CODE_OF_CONDUCT.md`](CODE_OF_CONDUCT.md) | Default code of conduct (Contributor Covenant 2.1) |
| [`SECURITY.md`](SECURITY.md) | Default security policy and private reporting instructions |
| [`.github/ISSUE_TEMPLATE/`](.github/ISSUE_TEMPLATE) | Default issue forms, including one for token parameter changes |
| [`.github/PULL_REQUEST_TEMPLATE.md`](.github/PULL_REQUEST_TEMPLATE.md) | Default pull request template |

A repository that needs different rules can override any of these by adding its own file at the same path; the org-level version applies only where the repository is silent.

Several of these files contain explicit `TODO (org owner)` markers for details that do not exist yet — a domain, contact addresses, social links. Search for `TODO` before treating them as finished.
