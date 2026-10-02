# Versioning: Cards and Studies

Versioning exists so a skeptical reader can still verify or reproduce after the lab notebook has moved on. See README: **The Core Test**.

## The Shift

Without explicit versioning, people **edit cards in place**. That
destroys the recipe that produced a past result.

With versioning, **editing becomes minting a new revision**. Past runs
stay answerable.

| Habit | Meaning |
|-------|---------|
| Edit the card, save | History-destroying unless the card was never used |
| Version the card | `card@n` stays runnable; changes are `card@n+1` |
| Version the study | The **set** of cards + DAG topology is what you publish |

## Three Layers

### 1. Artifact bytes (content hash)

Every data blob has `sha256` (or equivalent). Content-addressed truth.
If the hash changes, it is a **different artifact**, regardless of path.

### 2. Card revision (recipe / claim / data identity record)

Experiment, hypothesis, and data cards each carry a `version` (integer
or semver). 

**Rule:** Once a clean run **claims** outputs against a card revision,
that revision is **immutable**. Further changes require a new version
(or a new card id).

Drafts may live as `wip/` or `status: draft` and remain mutable until
first claimed execution.

### 3. Study / experiment DAG (bundle)

A **study** is the publishable unit: a named, versioned set of card
revisions and the edges between them.

```yaml
study: memory-thiasai-negspace
version: 0.3
date: 2026-09-12
summary: |
  Phase I memory probes: Eric-only corpus → REPORT measurable.

cards:
  - ref: memory-D-users-only-c3@1
  - ref: memory-E-v6-effort@1
  - ref: memory-D-report-v6@1
  - ref: memory-H-effort-vs-prompt@1

edges:
  - from: memory-D-users-only-c3@1
    to: memory-E-v6-effort@1
    role: input
  - from: memory-E-v6-effort@1
    to: memory-D-report-v6@1
    role: output
  - from: memory-E-v6-effort@1
    to: memory-H-effort-vs-prompt@1
    role: evidence
```

- **Card version** = this recipe or claim text  
- **Study version** = which pinned cards + topology support a finding  
- **Publishable evidence** = study snapshot (git tag / release), not
  "whatever is in the directory today"

## Why Study Versioning Matters

Hypothesis and experiment cards have many-to-many links. Without a
study bundle, you cannot say which **set** of card versions constituted
"the memory neg-space work" when a conclusion was drawn. Folder renames
(`v4`, `luna.low`, …) are not versions.

## Git as the Store

Practical default:

- Cards and study YAML live in git.
- Tag or release = study version.
- Large blobs in `inputs/` / `outputs/` with hashes on the data card;
  git-lfs optional.

## Relationship to Exploration

Lab notebooks and fuck-around packs (e.g. `memory_agent_expts/`) are
**not** versioned studies. They drive toward cards. Crystallization
minting H/E/D cards **and** a study version is the step up in rigor.
