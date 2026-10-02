# Data Cards

## The Problem

Experiment cards pin **inputs** and record **outputs**, but the same
blob of bytes often plays both roles across a DAG of work:

- `users_only.txt` is **data in** to a memory probe.
- `REPORT.md` is **data out** (the measurable) of that probe.
- The same `REPORT.md` is **data in** to a comparison or scoring step.

If provenance lives only inside each experiment card's `inputs:` block,
descriptions drift, hashes get copy-pasted wrong, and "which corpus?"
becomes a folder archaeology problem. Industry practice already treats
datasets as first-class documented artifacts (**dataset cards** /
**datasheets for datasets**). Agent science needs the same primitive
for *any* pinned artifact that flows through recipes — not only
training sets.

## Why This Is a Separate Primitive

Core test (all cards): *If a skeptical reader sees this card in the graph, does it give them enough information to verify the claim or reproduce the relevant result?* A data card fails that test if provenance or hash is missing.


| Primitive | Kind of thing | Lifecycle |
|-----------|---------------|-----------|
| **Experiment card** | Recipe (in → method → out → verify) | Written before clean run; frozen when claimed |
| **Hypothesis card** | Bet about the world | Updated as many experiments accumulate |
| **Data card** | Identity + provenance of an artifact | Created when the blob is minted or frozen; referenced, not rewritten, by recipes |

A data card answers: **What is this object, where did it come from, how
may it be used, and how must it not?**

An experiment card answers: **What do I do with these objects to produce
a finding?**

## Industry Parallel

- **Datasheets for Datasets** (Gebru et al.) — motivation, composition,
  collection, preprocessing, uses, distribution, maintenance.
- **Dataset cards** (Hugging Face, Google, etc.) — practical schema,
  splits, license, limitations.
- **Model cards** — same job for models.

We adopt the spirit: durable documentation of an artifact. We generalize
beyond "dataset" to any hashed blob (prompts, reports, indices, model
checkpoints referenced as data).

## Structure

```yaml
card: <project>-D<data-id>
type: data
version: <int or semver>          # immutable once referenced by a run
date: <when this version was minted>
revised: <optional; prefer new version over in-place edit>
investigators:
  - <who>

# Human label for this freeze
brief_summary: |
  One paragraph: what this is and why it exists.

# Content identity
artifact:
  path: <repo-relative or absolute path at mint time>
  hash: sha256:...
  media_type: text/plain | application/json | ...
  size_bytes: <n>
  # Optional token estimate for tooling (state the rule)
  token_estimate:
    rule: utf8_bytes_div_4    # grok binary heuristic, or other
    value: <n>

# How it was produced
provenance:
  produced_by: experiment | generator | human | import
  # If produced_by experiment:
  experiment_card: <card-id@version>
  # If produced_by generator:
  generator: |
    command or deterministic procedure
  source_artifacts:
    - ref: <other data card or path>
      hash: sha256:...

# Shape of the content
schema:
  description: |
    Human-readable format contract.
  # Optional machine hints
  format: users_only_v1 | report_phase1 | jsonl | ...
  fields: []                  # optional structured field list

# Roles this artifact is allowed to play
roles:
  - input                     # may appear in experiment inputs
  - output                    # may appear in experiment outputs
  # both is fine — same card, different edges

intended_use: |
  What experiments or analyses this is for.

not_for: |
  Misuses (e.g. "not ground-truth product ontology",
  "not a substitute for conversation.jsonl SoR").

limitations: |
  Known gaps (truncation history, missing agents, privacy, bias).

license_or_sensitivity: |
  e.g. contains human speech; internal only.

# Optional links
related_cards:
  experiments_using: []
  derived_from: []
  derives: []
```

## Data In vs Data Out

There is **one** data-card type. **In** and **out** are edges on the
experiment DAG, not separate species of card.

```text
[Data card: users_only@C3]  --input-->  [Exp card: v6-effort]
                                              |
                                              v
                                    [Data card: REPORT_v6]
                                              |
                                              +--input--> [Exp: score-negspace]
```

- **Data in** for a run: referenced under experiment `inputs:` by
  `card: ...@version` + hash check.
- **Data out** for a run: minted as a **new** data card (or new
  version) under experiment `outputs:`, with `produced_by: experiment`
  and hash of the written bytes.

Never "edit the REPORT data card" after a claimed run. Mint
`REPORT@2` or a new id.

## Relationship to Experiment Card `inputs:` / `outputs:`

Prefer **references** over inlining large blobs:

```yaml
# experiment card (excerpt)
inputs:
  corpus:
    data_card: memory-D-users-only-c3@1
    hash: sha256:ff9b...   # must match data card

outputs:
  report:
    data_card: memory-D-report-v6@1
    hash: sha256:...
    path: outputs/v6/REPORT.md
```

Small inline content (a short question string) may still live on the
experiment card with its own hash, as today. Large or reused artifacts
get data cards.

## When to Create a Data Card

- When you **freeze** a corpus for multiple trials (e.g. users_only C3).
- When an experiment **emits** a durable measurable (REPORT, metrics table).
- When an external dataset or model weights are **imported**.
- When two experiments would otherwise **duplicate** a long `inputs:` blurb.

## Anti-patterns

**"Mutable path is the card."** A folder named `v4` that keeps changing
is not a data card. Hash the bytes.

**"Edit after publish."** Once an experiment claims an output hash,
that data-card version is frozen. Next change = new version.

**"One card for the whole study."** The study/DAG is a separate
versioned bundle (see README: Versioning). Data cards are nodes; the
study is the graph.

**"Schema optional forever."** If agents parse REPORT sections, the
schema field should say what "valid REPORT" means (e.g. Section A + B,
no Section C).
