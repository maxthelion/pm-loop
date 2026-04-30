# pm-loop

A reusable Project Management workflow for codebases that use Claude Code or
Codex sub-agents. Originally extracted from the `in-sequence` music sequencer's
roadmap loop.

## What it gives you

Drop this bundle into any repo and you get:

- A **feature-directory schema** under `docs/roadmap/<feature-slug>/`
  (notes, user stories, existing-state, prototypes, ux-review, architecture,
  architecture-review, spec, plan, implementation-handoff, plus a `feedback/`
  inbox).
- A **deterministic selector** (`next-roadmap-actions.sh`) that reads each
  feature directory and emits the likely next planning action by checking the
  first missing artifact, with deferred status / unresolved feedback /
  blocked-metadata winning first.
- A **PM heartbeat skill** (`pm-next-action`) that reads the selector output,
  dispatches the `pm-assistant` sub-agent against the next agent item, commits
  via `commit-roadmap-action.sh`, and surfaces user-attention items.
- Capture helpers (`capture-clarification.sh`, `capture-feedback.sh`) that turn
  raw user input into routable artifacts under the feature directory.
- Process documentation (`working-through-a-roadmap.md`) and prototype style
  rules (`html-prototype-guidelines.md`) that ground the loop.
- A promotion helper (`promote-ready-item-to-worktree.sh`) that bridges a
  ready feature into a separate implementation loop / build worktree.

The loop is deliberately PM-only. It does not edit production code, run
implementation work, or write to `docs/specs/` / `docs/plans/`. A separate
build loop owns that.

## Goals of the bundle

- **Project-management as a deterministic file walk.** The selector is bash;
  it never asks an LLM what should happen next. Only the dispatcher uses an
  LLM, and only for one action per heartbeat.
- **The feature directory is the inventory.** All artefacts and conversations
  about a feature stay together in one place: notes, prototypes, reviews,
  feedback queue, spec, plan. Nothing is centralised.
- **User input is captured as structured files, not chat history.**
  `capture-clarification.sh` and `capture-feedback.sh` are the only on-ramps;
  every PM follow-up starts from a file on disk.
- **One repo as canonical source.** This repo is the source of truth.
  Consuming projects install a copy and record the source SHA so drift is
  visible.

## Layout

```
pm-loop/
├── manifest.yaml                     # what installs where
├── VERSION
├── scripts/
│   ├── next-roadmap-actions.sh       # selector
│   ├── capture-clarification.sh      # raw clarification → notes.md + intent.md
│   ├── capture-feedback.sh           # raw feedback → feedback/<ts>-<scope>.md
│   ├── attention-summary.sh          # blocked items + open questions
│   ├── commit-roadmap-action.sh      # commits docs/roadmap/** only
│   └── promote-ready-item-to-worktree.sh
├── claude/
│   ├── agents/pm-assistant.md        # sub-agent role + per-action contract
│   └── skills/pm-next-action/SKILL.md
└── docs/
    ├── working-through-a-roadmap.md  # process reference
    └── html-prototype-guidelines.md
```

When installed in a target project, the layout becomes:

```
<project>/
├── scripts/roadmap/*.sh
├── .claude/agents/pm-assistant.md
├── .claude/skills/pm-next-action/SKILL.md
├── docs/working-through-a-roadmap.md
├── docs/html-prototype-guidelines.md
├── docs/roadmap/                       # owned by the project
└── .pm-loop.lock                        # records source SHA + file hashes
```

## Install

The recommended installer is the `meta` hub:

```sh
meta install-pm-loop <project-path>
```

Or copy the manifest manually — `manifest.yaml` lists every source/target
pair. See [maxthelion/meta](https://github.com/maxthelion/meta) for the
installer + dashboard.

## Use

After install, in the consuming project:

```sh
mkdir -p docs/roadmap
scripts/roadmap/next-roadmap-actions.sh   # generate next-actions.md
/loop 10m /pm-next-action                 # autonomous heartbeat (Claude Code)
```

The first feature requires a `docs/roadmap/<slug>/README.md` with frontmatter
(`id`, `title`, `status: inventory`, `priority`, `blocked_by`, `stage`,
`owner`, `updated`). Add a clarification with `capture-clarification.sh`; the
loop takes it from there.

## Status

v0.1 — extracted from `in-sequence` as a faithful copy. The selector and
agent contract are well-exercised but the bundle is new; expect the install
contract and any consuming-project assumptions (e.g. `Sources/`, `Tests/`,
`wiki/pages/` references in `pm-assistant.md`) to be parameterised in
follow-up versions.

## Repos

- This bundle: pm-loop (https://github.com/maxthelion/pm-loop after publish)
- Hub + installer: https://github.com/maxthelion/meta
- Source project: in-sequence (private)
