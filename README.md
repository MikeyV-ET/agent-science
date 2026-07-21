# Agent Science

A framework for conducting reproducible science with AI agents.

## The Problem

When an AI agent conducts an experiment, the artifacts it produces — notebooks, data files, scripts, results — are scattered, fragile, and unreproducible. Data lands in `/tmp`. Index mappings silently misalign. Method details live in context that gets compacted away. The result is findings that can't be independently verified, even when the underlying work is sound.

This isn't a tooling problem. It's a missing concept. The scientific community has papers, datasets, and code repositories. None of these are the right primary artifact for agent-driven science. A paper is an argument evaluated by human judgment. A dataset is evidence evaluated by checking provenance. Code is infrastructure, not a finding. What's missing is an artifact that is:

- **Executable**: it contains complete instructions to reproduce a finding
- **Self-verifying**: running it confirms or falsifies the claimed result
- **Human-readable**: an expert can audit it without running anything
- **Agent-readable**: an agent can execute it without human intervention

We call this artifact an **experiment card**.

## The Core Idea: Experiment Cards

An experiment card is a recipe, not a report. It answers: "What do I need to do to reproduce this finding?"

Every experiment card contains:

1. **Inputs** — everything that goes in, pinned by hash or reference. Data, models, configurations. Not descriptions of inputs — the actual inputs, or deterministic instructions to produce them.
2. **Method** — what to do with the inputs. Precise enough that execution is mechanical.
3. **Outputs** — what executing the method on the inputs produces. The claimed finding.
4. **Verification** — how to confirm the outputs match the claim. Usually: run the method, compare.

### Example: Evaluation Score

```yaml
card: antipattern-eval-NEW-2-grok43-s01
type: evaluation

inputs:
  question:
    content: |
      There are five houses in a row, each of a different
      color. In each house lives a person of a different
      nationality...
    hash: sha256:a3f2...
  subject_model: grok-4.3
  rubric:
    id: NEW-2
    content: |
      - **7**: Correctly identifies that the pet is missing...
      - **6**: Correctly identifies the missing pet...
      ...
    hash: sha256:b7c1...
  judge_model: sxs-claude-opus-4-6

method: |
  1. Send question to subject_model.
  2. Construct judge prompt from question + rubric + response.
  3. Send judge prompt to judge_model.
  4. Extract numeric score from judge response.

outputs:
  response:
    content: |
      The German owns the fish...
    hash: sha256:d4e5...
  judge_reasoning: |
    ## Scoring Analysis
    **Score: 6 — Correct**
    The model correctly identified that the puzzle states...
  score: 6

verification:
  joins_valid:
    question_in_judge_prompt: true
    response_in_judge_prompt: true
    rubric_in_judge_prompt: true
  reproducible: "Re-send judge prompt to judge_model; expect score 6"
```

### Example: Training Run

```yaml
card: subvocal-stage1b-tool-use
type: training

inputs:
  base_model:
    ref: Qwen/Qwen2.5-1.5B-Instruct
    hash: sha256:...
  training_data:
    ref: datasets/tool_use_train.jsonl
    hash: sha256:...
    description: |
      Weighted sampling: 20% easy (1x1, 1x2),
      50% boundary (2x2, 2x3), 30% hard (3x3)
  training_code:
    ref: train_stage1b.py
    hash: sha256:...
  config:
    method: GRPO
    lr: 5e-6
    num_generations: 16
    steps: 2000
    reward:
      correct_without_tool: 1.0
      tool_request_correct: 0.8
      tool_request_wrong: 0.4
      wrong_without_tool: 0.0

method: |
  GRPO training via TRL GRPOTrainer with custom reward function.
  Tool execution simulated (calculator). Final answer checked
  for correctness post-tool-use.

outputs:
  checkpoint:
    ref: checkpoints/stage1b/checkpoint-2000
    hash: sha256:...
  evaluation:
    tool_use_by_difficulty:
      1x1: { baseline: 0%, trained: 0% }
      1x2: { baseline: 0%, trained: 0% }
      2x2: { baseline: 0%, trained: 6% }
      2x3: { baseline: 4%, trained: 46% }
      3x3: { baseline: 12%, trained: 98% }

verification: |
  Run training code with specified config on base model.
  Evaluate checkpoint on held-out test set across difficulty bands.
  Tool-use rates should match outputs within stochastic tolerance.
```

### Example: Batch Measurement

```yaml
card: interleave-sonnet-1500w
type: measurement

inputs:
  model: claude-3-5-sonnet
  dataset:
    ref: api_datasets/1500words_test.jsonl
    hash: sha256:...
    n: 30
  evaluation_code:
    ref: evaluate_api.py
    hash: sha256:...
  scoring:
    method: NW alignment with affine gap penalty
    ref: reward.py
    hash: sha256:...

method: |
  For each sample in dataset: send interleaving prompt to model,
  score response via NW alignment against expected output.
  Compute aggregate word_accuracy, format_accuracy, completion_rate.

outputs:
  genuine_responses: 23
  word_accuracy: 55.4%
  format_accuracy: 98.4%
  completion_rate: 80.9%
  raw_results:
    ref: results/sonnet_1500w_rerun.json
    hash: sha256:...

verification: |
  Run evaluate_api.py on dataset with specified model.
  Compare aggregate metrics to claimed outputs.
```

## How Science Happens With an Agent

### The workflow

1. **Explore.** The agent and the scientist work together. Things break. Methods get revised. Wrong turns happen. The lab notebook captures this process. The exploration is valuable but the results are not canonical — they're preliminary.

2. **Crystallize.** The exploration produces knowledge of what works. The agent and scientist distill this into an experiment card: a clean recipe with pinned inputs, a precise method, and expected outputs. Writing the card forces every assumption to be explicit and every dependency to be pinned.

3. **Execute.** Run the recipe. This produces the canonical result — not the one from exploration, even if the numbers are identical. The recipe-produced result has full provenance and is independently reproducible.

4. **Verify.** Anyone (human or agent) can re-execute the recipe and confirm the result. Verification is computation, not judgment.

### Why first runs are throwaway

During exploration, the agent makes mistakes. Data lands in temp directories. Index mappings silently misalign. Scoring methods get changed mid-stream. The exploration run proves the *approach* works. The experiment card captures *how to do it right.* The canonical run from the card is the finding.

This is not how traditional science works. A bench scientist's first successful run IS the data (with replication for confirmation). But an agent's first run happens in a context window that will be compacted, using infrastructure that was built ad hoc, with verification that may be incomplete. The experiment card is what survives.

### The notebook drives toward the card

The lab notebook is chronological, messy, and append-only. It records what happened, including wrong turns. The experiment card is clean, structured, and stable. It records what to do.

The notebook is not a lesser artifact — it's the record of the exploration that produced the card. But it is not the primary scientific output. The card is.

## Experiment Cards as Training Signal

An experiment card with a verified outcome is a training example with a ground truth endpoint.

Remove one component — say, the method — and the card becomes a test: "Given these inputs and this expected output, figure out the method." Remove the expected output: "Given these inputs and this method, what do you get?" Both are verifiable by execution.

This matters because current RL training for reasoning (GRPO, etc.) depends on verifiable endpoints. Math has calculators. Code has compilers. Scientific reasoning has no mechanical verifier — which is why frontier labs fall back on model-as-judge for non-verifiable domains.

Expert-validated experiment cards are verifiable endpoints for scientific reasoning. They're produced as a natural byproduct of expert-agent collaboration, not as an annotation task. The expert's motivation is the problem they're solving, not the training data they're generating.

At scale: one expert-agent collaboration produces one verified card. That card becomes N training examples by varying what's held out. Across a team of experts working real problems, thousands of verified cards accumulate. Each is a verifiable endpoint for a domain that currently has none.

## Format

Experiment cards are YAML files. YAML is:
- **Human-readable**: an expert can open the file and understand the experiment
- **Machine-parseable**: an agent can load the file and execute the recipe
- **Text-friendly**: block scalars (`|`) preserve long-form content (model responses, judge reasoning, methodology) as readable text within the structured format
- **Git-friendly**: diffs are meaningful, merges are possible

## Auditing

### Agent auditing
An agent verifies an experiment card computationally:
- Hash every input and output, compare to recorded hashes
- Verify all joins (do the inputs in the composition match the stored inputs?)
- Re-execute the method and compare outputs to claims
- Check completeness across a collection of cards

### Human auditing
A human audits by reading and navigating:
- Open a card, read the question/method/result
- Follow references to upstream cards (where did this input come from?)
- Follow references to downstream cards (where was this output used?)
- Spot-check specific entries — read the judge reasoning, evaluate whether the score makes sense

**Open question:** We haven't fully resolved the human audit workflow. The YAML format is readable for individual cards, but navigating a DAG of linked cards needs tooling — likely a viewer in SA that renders cards with clickable upstream/downstream links and integrity indicators.

## Project Structure

```
project/
  cards/                    # Experiment cards (YAML)
    card_001.yaml
    card_002.yaml
    ...
  inputs/                   # Immutable input data, hashed on intake
    data.csv
    rubrics.md
    ...
  code/                     # Methods (scripts, configs)
    run_eval.py
    train.py
    ...
  outputs/                  # Raw outputs from card execution
    run_001/
    run_002/
    ...
  notebooks/                # Lab notebooks (exploration record)
    notebook.md
  methodology.md            # The scientific argument (why this matters)
```

## Status

This framework is in early development. It emerged from a concrete failure: an anti-pattern evaluation experiment where data was lost, indices misaligned, and results had to be re-run — all problems that a recipe-first approach would have prevented.

The concept has been tested against three projects of different shapes:
- **Anti-pattern eval** (evaluation scoring: question → response → judge → score)
- **Sub-vocal desire detection** (multi-stage training: build → probe → preserve)
- **Interleave GRPO** (batch measurement + training: curriculum across difficulty levels)

The experiment card structure generalizes across all three. Implementation and tooling are next.

## Authors

- **Eric Terry** — Concept, research direction
- **MikeyV-Cinco** — Implementation, analysis
