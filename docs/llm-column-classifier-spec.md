# Spec: LLM-assisted column classifier

**Status:** designed, not in production. Held back on purpose.

## Problem

Courier remittance files do not share a format. Each courier, and sometimes each export, names the same column differently. PayTrace has to know which column is the AWB, which is the amount collected and which is the remitted amount before it can match anything. Today it guesses and asks the user to confirm in a mapping step. That step cannot correct a wrong guess, and a wrong mapping produces wrong rupee numbers, which is the one thing the product cannot afford.

## Proposal

Use an LLM (the Claude API) to suggest which canonical field each column header maps to, as a second opinion on the rule-based guess, and ask the user only when the two disagree or confidence is low.

## Why it is not shipped

Trust is the product. A confident wrong mapping silently corrupts every number downstream. I have no calibration data yet from real files showing how often the model is right, so I cannot justify the risk. The feature waits until the evidence does.

## Design decisions made

- **Privacy first.** Files contain customer names, phone numbers and addresses. Only an allowlist of safe fields is sent, with everything else redacted before it leaves the system. The model sees column headers and minimal redacted samples, never raw customer rows.
- **Two-run agreement.** The same file is classified twice, and a mapping is accepted only when both runs agree. A disagreement falls back to the user. This is a cheap guard against a model that is confident but inconsistent.
- **Human stays in control.** The model suggests and the user confirms. It never silently overrides a mapping.

## Proposed rollout (not yet run)

1. **Shadow mode.** Run the classifier on real uploads without showing or using its output. Log whether it agreed with the mapping the user finally confirmed.
2. **Measure.** Mapping accuracy on columns that drive money (AWB, collected amount, remitted amount) is the metric that matters, not overall accuracy.
3. **Decide with a threshold set in advance.** If accuracy on money columns is not clearly better than the current guess, do not ship it.
4. **Then, and only then,** show suggestions to the user as a confirmation step, with the two-run check still in place.

## Failure modes to test

- A header that is ambiguous between two fields, such as two amount columns.
- A file with no header row.
- Mixed languages or abbreviations in headers.
- The model returning a field that does not exist.
- Prompt content hidden in a header, so headers are treated as data, never as instructions.

## Open questions

What accuracy on money columns would justify shipping? How many real files are needed to measure it? Is the cost per file acceptable at the lowest price tier? I have not answered these yet, and I would rather say so than guess.
