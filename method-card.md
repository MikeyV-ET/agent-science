# Method Cards

## The Problem

Experiment cards pin a **recipe for one trial**: this item set, this
question, this output. Data cards pin **bytes**. Code in a `methods/`
folder is only identity if you hash the file. None of those is the
**transformation**: tapes + h5 → grade rows, with a contract a skeptic
can execute on a fixture.

## Why This Is a Separate Primitive

Core test: *If a skeptical reader sees this card in the graph, does it
give them enough information to verify the claim or reproduce the
relevant result?*

| Card | The check |
|------|-----------|
| Data | Hash the file |
| Method | Run this map on a fixture; match the contract |
| Experiment | Run this recipe on this item set; match the finding |
| Hypothesis | See whether cited experiments discriminate the claim |
| Study | See which cards and edges constitute the record |

A data card of a `.py` file verifies **identity of the file**. A method
card verifies **behavior of the map**. An experiment card **uses** a
method card: run method M on set S.

## Structure

```yaml
card: M-h5-grade-from-tapes
type: method
maps:
  from: [uuid updates.jsonl tapes, official tests, h5]
  to: [jsonl of {session_id, passed, sec, rc, err}]
code:
  path: methods/h5-grade-from-tapes.py
  hash: sha256:...
  data_card: D-h5-grade-from-tapes   # the bytes
  invoke: python3 methods/h5-grade-from-tapes.py --sit sv46_34_1 --step 34.1
check: |
  Hash the .py. Re-run. Compare session_id, passed, rc to the output
  data card (sec may differ). One-tape smoke: uuid … must pass.
does_not:
  - choose which tapes are legitimate
  - claim a sample size
```

`invoke` may show a default item set; the method card still does not
own the scientific question. That stays on the experiment card.

## Sets

When many artifacts share a role (80 uuid tapes), describe a **set**:
directory, glob, `n`. Edges point at the set. Placeholders (`D-<uuid>`)
and N copy-pasted edges both fail a skeptic. An agent expands the glob
and checks `n`.

## Relation to experiment cards

Experiment cards should **reference** method cards rather than restating
the four numbered steps. The experiment names the item set and the
question. The method names the map.
