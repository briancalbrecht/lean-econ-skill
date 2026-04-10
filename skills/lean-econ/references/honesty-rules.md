# Honesty rules

These are non-negotiable. They exist because the failure mode of this
skill is not "proof doesn't compile" — it's "user thinks a claim is
verified when it isn't."

## Rule 1: Never say "proven" without the depth

Wrong: "Murphy's claims are Lean-proven."
Right: "5 of 11 Murphy claims are Lean-proven, all in the quadratic
specialization. Both general propositions are gaps."

If someone asks "are the claims proven?" the answer includes the depth
spectrum, the gap count, and what the gaps are. Always.

## Rule 2: Quadratic ≠ general

Proving the quadratic/polynomial specialization of a proposition does NOT
prove the general proposition. If the paper states a result for generic
functions and you prove it for a specific functional form, the status is
**core only**, not **fully verified**. Say explicitly: "The quadratic case
is verified. The general proposition (for generic D, c) is a gap."

## Rule 3: Taking assumptions as hypotheses ≠ proving them

If a Lean theorem takes `(h : r_i > q_i)` as a hypothesis, it has NOT
proven that `r_i > q_i`. It has proven that IF `r_i > q_i` THEN the
conclusion follows. The status is **logic only**. The upstream derivation
of `r_i > q_i` (e.g., from LP duality) is a separate gap.

## Rule 4: After every formalization session, produce a coverage table

Before the user leaves, they must see:

| Claim | Paper statement | Lean status | What Lean checks | What it doesn't |
|-------|----------------|-------------|------------------|-----------------|

The "What it doesn't" column is the most important. If it's empty for
every row, something is wrong — you're not being honest about the gaps.

## Rule 5: Flag when asked "is X proven?"

If the user asks whether something is proven, verified, or covered, and
the answer involves any nuance (core only, logic only, quadratic but not
general, hypotheses taken not derived), lead with the caveat:

"Not fully. Here's what Lean actually checks vs what the paper claims..."

Do not bury the caveat after a paragraph of what IS verified. Lead with
the gap.

## Rule 6: The findings doc is the deliverable

Every paper gets a `proofs/lean-findings.md`. It must contain the coverage
table from Rule 4, the depth spectrum, and the "What Lean checks vs what
it doesn't" breakdown. This file is what the user (or a coauthor, or a
referee) reads to understand the verification status. If this file is
misleading, the whole skill has failed.

## Why these rules exist

The user said: "so we are confident the claims in murphy are lean proven."
The correct answer was "no — here's what's proven and what isn't." The
first response should have been the honest table, not a yes that required
follow-up to correct. These rules ensure the honest answer comes first.
