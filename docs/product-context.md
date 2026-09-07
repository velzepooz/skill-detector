# What this tool is for

`skill-detector` is the detection engine behind **SkillTrust**, a product that
scores the trust and security of AI-agent skills.

This repository is the engine and nothing else: the file walk, the rules, the
grading, the CLI. Everything a scan concludes is decided here.

## The problem

Installing a skill from a third-party source means running someone else's
instructions inside an agent that already has your filesystem, your shell and
your credentials. The dangerous part of a skill is rarely a compiled artifact —
it is text, sitting in a manifest or an instruction file, that an agent will
read and act on. Reading every line of every skill by hand does not scale, and
skimming it does not work: the interesting content is written to look ordinary.

The same problem shows up one level out. A repository accumulates agent
configuration — instruction files at several levels, settings, hooks, MCP
servers — and no one has a picture of what all of it permits taken together.

## The two objects

The product is built around two questions, and the engine serves both.

**Check a skill** — *"Can I install this thing?"* The subject is one skill: a
manifest, or an archive containing exactly one skill root, with everything
under it. The reader is a developer about to install someone else's skill and
deciding whether to.

**Check a repository** — *"What can my agents do in this project?"* The subject
is a whole tree: every skill in it, plus the instruction files, the settings,
the hooks and the MCP configuration. The reader is whoever is responsible for
the project.

The difference between the two is what the user is pointing at, not how much of
the tool they get. The engine handles both through the same pipeline — pointing
it at a skill root and pointing it at a repository differ in what discovery
finds, not in which rules run.

## Who uses it

- **Developers installing third-party agent skills**, who want a verdict before
  the skill reaches their machine.
- **Maintainers gating their own repositories**, who want the agent surface
  checked on every change rather than audited once and forgotten.

Both care about the same thing: catching a problem before an agent acts on it.

## The three surfaces

The engine reaches users three ways.

- **This CLI.** Run it locally against a directory or a file. Offline,
  deterministic, no account, no network dependency. It is also the Go library
  the other two surfaces embed.
- **The hosted scanner at [skilltrust.app](https://skilltrust.app).** Point it
  at a skill or a repository in the browser and read the report there.
- **The GitHub Action.** Runs a scan in CI and fails the build on a threshold,
  so a repository's agent surface is checked on every pull request.

All three run the same rules and produce the same grades. See
[`cross-repo.md`](cross-repo.md) for how they are wired together.

## What a result means

A scan grades four axes on an A–F scale, and reports the findings that drove
each letter. The grades are a statement about the files the rules actually
read: when a scan finds no agent surface to inspect, it says so and issues no
grades rather than reporting a clean result about a tree nothing looked at.

An `A` on an axis means nothing was counted against it. It is the absence of a
detection, not a certificate — a scanner reports what it can recognise, and a
clean result is evidence, not a guarantee.
