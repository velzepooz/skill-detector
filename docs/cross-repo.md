# Downstream consumers

`skill-detector` is one of three repositories. It is the only one that decides
what a scan concludes; the other two present its results to users. Anyone
changing an exported symbol here is changing two other codebases.

## The three repositories

| Repository | Role | Visibility |
|---|---|---|
| `skill-detector` | The detection engine. The file walk, the rules, the grading, and the CLI. | Public. |
| The hosted scanner, `skilltrust` | The web application at [skilltrust.app](https://skilltrust.app): the scan pages, the reports, the badge service and the GitHub App backend. Embeds this engine as a Go library. | Private. |
| `scan-action`, the GitHub Action | Wraps a scan so a repository can gate its pull requests on the result. | Public. |

## `pkg/` is a published API

Everything under `pkg/` is imported as a library by both of the other two.
`cmd/` is the CLI entry point and is not imported by anything.

The consequence is direct: **every exported change under `pkg/` is
downstream-visible.** A renamed field on `model.Finding`, a changed signature
on `scanner.Options`, a new required argument — each is a compile break in two
repositories that this repository's own test suite will not notice. The types
that travel furthest are `model.ScanResult` and everything nested inside it,
`axes.Axis` and `axes.Grade`, the `rules.Rule` interface and
`rules.DefaultRegistry()`, and `delta.Compute`.

The JSON wire format travels further still, because consumers read it out of
process. Its version lives in `pkg/model.SchemaVersion` and is bumped in the
same commit as any change to the shape of `ScanResult` — see
[`architecture.md`](architecture.md).

Two properties of `pkg/` exist because it is embedded elsewhere, and are not
incidental: the packages that do the reasoning are pure — no IO, no global
state — and `pkg/rules` never touches the filesystem, reading only the
`FileContext` it is handed. Introducing a global, or a disk read inside a rule,
breaks the embedding rather than just this repository.

## A release is not done when the tag is cut

Three places downstream pin this engine's version, and **none of them notices a
new tag**:

1. **`action.yml` in the GitHub Action** — the `detector-version` input. Moving
   it is necessary but not sufficient: the Action needs its own release before
   the change reaches anyone using it by tag.
2. **`go.mod` in the hosted scanner** — the version of this module it compiles
   against.
3. **The hosted scanner's CI fixture clone** — a second, separate pin, checking
   out this repository at a fixed version to generate test fixtures. It is easy
   to move the first two and forget this one; the symptom is a green build
   testing an engine nobody is running.

Moving all three is part of the release, not follow-up work. Until they move,
the tag exists and no user is running it.

An agent working only inside this repository cannot do any of that — two of the
three pins are in a repository it cannot see. The correct ending for a release
done from here is a handoff that names the three pins, not a claim that the
work shipped.

## Merging to the hosted scanner's `main` deploys production

The hosted scanner deploys from its default branch. A merge to `main` there is
a production deployment, including a documentation-only merge. This matters
from here because a change in this repository frequently lands as a version
bump there, and that bump is a deploy.

## Coordination records

Records of how these repositories are coordinated — release history, downstream
state, and the reasoning behind the arrangement above — are kept privately by
the maintainer and are not part of this repository. If you need to know
something about a downstream consumer that this page does not answer, open an
issue.
