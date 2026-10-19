# Finding Cards

## The Problem

Hypothesis cards capture a bet that is still open: a claim, a ledger,
and what would kill it. Experiment cards capture a recipe that was
run. Neither is the locked scientific result.

Agents (and humans) often treat a notebook recap as the finding, or
leave a hypothesis “open” after the decision is already made. A
skeptic then cannot tell the assertion from the trail that led there.

## Why This Is a Separate Primitive

Core test: *If a skeptical reader sees this card in the graph, does it
give them enough information to verify the claim or reproduce the
relevant result?*

| Card | The check |
|------|-----------|
| Data | Hash the file |
| Method | Run this map on a fixture; match the contract |
| Experiment | Run this recipe on this item set; match the outputs |
| Hypothesis | See whether cited experiments discriminate the claim |
| **Finding** | See the assertion and the graph required to support it |
| Study | See which cards and edges constitute the record |

A hypothesis stays revisable as evidence accumulates. A finding is
**locked**: further disagreement mints a new finding or a revision,
it does not silently rewrite the old assertion.

## Structure

```yaml
card: F-<short-name>
type: finding
version: 1
status: record
date: <locked>
investigators:
  - <names>
assertion: |
  One paragraph. What is true, in the object language of the work.
support: |
  Which data, method, and experiment cards a skeptic opens next.
  Join recipe in two sentences if the graph is a join (e.g. tape
  session_id to a grade row).
excluded: |
  Nearby claims this finding does not make.
not_to_be_confused_with:
  F-other: one line on the distinction
bears_on: |
  What membership, KEEP, pack, or downstream recipe this locks.
related_cards:
  data: [D-...]
  methods: [M-...]
  experiments: [E-...]
  pack: [S-...]
```

`related_cards` uses **full references**: card id (and `@version` once
claimed), or a named set with glob and `n`. Not placeholders.

## Lifecycle

1. Work produces data cards, methods, experiments, maybe a hypothesis.
2. When the assertion is ready to lock, mint the finding. Point it at
   the support graph. Write `excluded` and `not_to_be_confused_with`
   while the near-misses are still audible.
3. Freeze the finding revision. Study snapshot includes it.
4. A later, different assertion is a new finding (or `@n+1` with a
   changelog), not an edit in place.

## What a Finding Is Not

- A lab-notebook recap of how the week felt.
- A hypothesis that still lists discriminators as open work.
- A data card of a tape. The tape is evidence; the finding is the
  assertion the tape supports.
- Project-specific KEEP / mill policy. Those live on the study or
  pack that uses the method.

## Authors

Framework: Eric Terry, MikeyV-Squiggy.
