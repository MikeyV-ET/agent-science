# Hypothesis Cards

## The Problem

Experiment cards capture what happened. But agents (and humans) don't
just collect data — they build models. The model-building process is
where most errors occur: premature commitment to a narrative,
confirmation bias, failure to consider alternatives, confusing
"consistent with" for "proven by."

A hypothesis card is a structured artifact that makes the
model-building process auditable. It names a claim, lists what
supports it, lists what would kill it, and tracks its status as
evidence accumulates.

## Why This Is a Separate Primitive

Core test (all cards): *If a skeptical reader sees this card in the graph, does it give them enough information to verify the claim or reproduce the relevant result?* For hypotheses, that means claim + ledger + discriminators clear enough to audit without trusting the narrative.


An experiment card is a recipe. A hypothesis card is a bet.

They have different lifecycles:
- An experiment card is written before execution and updated with
  results. It is tied to a specific method and dataset.
- A hypothesis card is written when a pattern is noticed and updated
  as multiple experiments produce evidence. It is tied to a claim
  about the world.

Multiple experiment cards may reference the same hypothesis card.
A single experiment may update multiple hypothesis cards. The
relationship is many-to-many.

## Structure

```yaml
card: <project>-H<number>
type: hypothesis
date: <when first proposed>
revised: <when last updated>
investigators:
  - <who proposed it>

claim: |
  A plain-language statement of what you think is true.
  Specific enough to be wrong.

consistent_with:
  - "observation or experiment card ref that supports the claim"
  - "another supporting observation"

inconsistent_with:
  - "observation or experiment card ref that weakens the claim"
  - "leave empty list [] if nothing contradicts yet"

discriminators:
  - observation: |
      A specific, concrete thing you could observe.
    eliminates: <hypothesis_id or this_hypothesis>
    supports: <hypothesis_id or list of ids>
    how_to_test: |
      Optional. What experiment would produce this observation.
  - observation: |
      Another discriminating observation.
    eliminates: <hypothesis_id>
    supports: <hypothesis_id>

status: open | supported | weakened | eliminated | superseded
strength: strong | moderate | weak | speculative
superseded_by: <hypothesis_id, if status is superseded>
```

## Key Fields

### claim
The thing you think is true. Must be specific enough that you can
describe what would prove it wrong. "Something is going on with
the proxy" is not a hypothesis. "The proxy routes requests to
different pools based on request shape" is.

### consistent_with / inconsistent_with
Evidence ledger. Every observation that bears on the claim goes
in one list or the other. This is the audit trail — it makes
visible how much support the claim actually has vs how much you
feel like it has.

### discriminators
The most important field. Each discriminator names:
1. A specific observation (not vague — "binary gets an OK while
   curl gets PRESENT in the same second" not "they disagree")
2. Which hypothesis that observation eliminates
3. Which hypothesis it supports

This forces you to think about what would change your mind.
An agent that can list discriminators but not design experiments
to produce them is doing half the work. An agent that can design
experiments but not list discriminators is doing the other half.
Both halves are necessary.

### status
- **open**: not enough evidence to call it
- **supported**: preponderance of evidence favors it, no
  contradictions, but not conclusively proven
- **weakened**: some evidence against, or a competing hypothesis
  explains the data better
- **eliminated**: a discriminating observation has falsified it
- **superseded**: a more specific or more general hypothesis
  has replaced it

### strength
How much you'd bet on it. Subjective but honest. Update it
as evidence accumulates.

## Usage

### When to create a hypothesis card
When you notice yourself explaining data with a story. The story
is the hypothesis. Write it down before you start believing it.

### When to update
After every experiment that bears on the claim. Add to
consistent_with or inconsistent_with. Check discriminators
against new data. Update status and strength.

### When to create a new one
When a discriminating observation eliminates one hypothesis and
supports a new one that you hadn't previously articulated.

### Relationship to experiment cards
- Experiment card references hypothesis card: "this experiment
  was designed to test discriminator D on hypothesis H."
- Hypothesis card references experiment card: "experiment E
  produced observation O, which is consistent/inconsistent
  with this claim."

## Anti-patterns

**"Consistent with everything, eliminates nothing."** If your
hypothesis can't be wrong, it's not a hypothesis. Add
discriminators or make the claim more specific.

**"One observation, strong confidence."** Strength should reflect
evidence quantity and quality, not narrative plausibility.

**"Updating consistent_with but never inconsistent_with."**
Confirmation bias. Every observation should be scored against
all open hypotheses, not just the favored one.

**"Discriminator requires an impossible observation."** "If we
had the proxy source code" is not actionable. Discriminators
should describe observations you could actually produce with
available tools.
