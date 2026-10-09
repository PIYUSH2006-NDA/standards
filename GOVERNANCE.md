# Governance

> **Status:** interim governance for the draft phase (before v1.0).

## Maintainers

| Maintainer | GitHub | Role |
|---|---|---|
| roenu | [@roenudev](https://github.com/roenudev) | Founding editor and maintainer |

Maintainers review and merge changes, triage issues, and keep the drafts consistent. New maintainers are added by agreement of the existing maintainers, based on sustained, constructive contribution.

## Decisions

- **Small changes** (typos, links, clarifications that do not change meaning) are merged by a maintainer after review.
- **Substantial changes** (new or changed normative rules, profile requirements, new documents) go through a proposal issue and, where requested, an RFC using [rfcs/0000-template.md](rfcs/0000-template.md).
- RFCs are open for comment for at least 14 days. Maintainers aim for consensus. If consensus is not reached, the maintainers decide and record the reasoning in the RFC.
- Accepted RFCs are numbered and merged into `rfcs/`, and the drafts and [CHANGELOG.md](CHANGELOG.md) are updated.

## Document Status Labels

| Label | Meaning |
|---|---|
| **Draft** | Under active development. Not for implementation. May change completely |
| **Candidate** | Feature-complete for its version; open for implementation feedback and test suites |
| **Stable** | Released version; changes only through a new version |
| **Superseded** | Replaced by a newer document; kept for history |

All current documents are **Draft**. Superseded drafts live in `drafts/history/`.

## Versioning

- Each document has its own version (for example runtime v0.3, communication v0.1).
- Before v1.0, any version may introduce breaking changes.
- Conformance profiles become claimable beyond self-assessment only after a v1.0 release and a separately agreed conformance process.

## Path to Neutral Maintainership

Before any document reaches v1.0, the project intends to:

1. add maintainers from more than one organisation and jurisdiction;
2. decide on a long-term home (an open working group, a foundation, or an existing standards organisation);
3. adopt a written intellectual-property and patent policy suitable for an open standard;
4. clarify the trademark status of the project names (see [TRADEMARKS.md](TRADEMARKS.md)).

## Code of Conduct

All participation is governed by the [Code of Conduct](CODE_OF_CONDUCT.md).
