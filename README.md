# lean-econ

A Claude Code skill for formalizing economics-paper claims in Lean 4 + mathlib.

## What it does

Type `/lean` in any paper project. The skill:

1. **Sets up** the project for Lean formalization (finds the tex, creates directories, adds config to CLAUDE.md)
2. **Discovers claims** — scans the tex for all theorems, propositions, and lemmas; classifies each as formalizable or not
3. **Formalizes** claims as Lean 4 theorems with negative-control companions
4. **Verifies** — runs `lake build`, checks `#print axioms` for `sorry`
5. **Reports honestly** — depth spectrum (fully verified / core only / logic only / gap), unused hypotheses, coverage gaps

## What it surfaces that reading the paper can't

- **Unused hypotheses.** Lean warns when a theorem doesn't use an assumption. That means the result holds under weaker conditions than stated.
- **Stuck proofs.** The goal state tells you exactly what's missing — a gap in the argument, a missing hypothesis, or a wrong encoding.
- **Trivial proofs.** When Lean closes a claim in one tactic, the claim may be weaker than the paper suggests.
- **Hidden lemmas.** Intermediate facts the paper uses implicitly but never names.
- **Depth of verification.** Not all "verified" is equal. Proving the quadratic case does not prove the general proposition.

## Install

```bash
claude plugin marketplace add https://github.com/briancalbrecht/lean-econ-skill.git
claude plugin install lean-econ@briancalbrecht
```

Or for a single session:
```bash
claude --plugin-dir /path/to/lean-econ-skill
```

## Prerequisites

- [Lean 4](https://leanprover.github.io/lean4/doc/setup.html) installed via `elan`
- A Lean 4 + mathlib project (one-time setup):
  ```bash
  mkdir ~/lean-econ && cd ~/lean-econ
  lake init LeanEconomics math
  lake update
  ```
- [Claude Code](https://claude.ai/code) CLI

## Usage

In any economics paper project:

```
/lean                        # setup + verify-all + report gaps
/lean formalize <claim>      # formalize a specific claim
/lean verify <claim>         # check one existing theorem
/lean walkthrough <claim>    # regenerate the bijection doc
/lean status                 # quick overview
```

On first invocation, `/lean` automatically:
- Finds your tex file
- Asks where your Lean project lives (or reads it from CLAUDE.md)
- Creates the paper's directory in the Lean project
- Scans the tex for all formal claims
- Reports which are formalizable and which aren't

## How it works with your paper

The skill adds a `## Lean formalization` section to your paper's CLAUDE.md:

```markdown
## Lean formalization
- **paper_name**: waterbed
- **tex_path**: comment/comment.tex
- **lean_root**: /Users/you/lean-econ
- **lean_dir**: /Users/you/lean-econ/LeanEconomics/Papers/Waterbed
```

Lean files live in the central Lean project, not in the paper repo. The paper repo just has the pointer in CLAUDE.md.

## Depth spectrum

Not all verification is equal. The skill reports depth honestly:

| Depth | Meaning |
|-------|---------|
| **fully verified** | Self-contained proof, no economic assumptions needed |
| **core only** | Algebraic engine verified; economic wrapper is a hypothesis |
| **logic only** | If-then chain verified; the "if" comes from an unformalized upstream result |
| **statement only** | Compiles with `sorry`; statement well-formed, proof incomplete |
| **gap** | No Lean file at all |

"9 of 9 verified" without this breakdown is misleading. The skill always leads with the caveats.

## Tested on

- **Waterbed** (IO theory) — 11 fully verified polynomial inequality proofs, all KKT sub-cases
- **Kirzner** (entrepreneurship) — 9 fully verified, discovered WLOG ordering is unnecessary for profit non-negativity
- **Know-how** (firm theory) — 7 verified at varying depth (1 fully, 2 core, 3 logic, 1 gap)
- **Murphy antitrust** (gains from trade) — 5 quadratic-case + 1 general interior-max using real calculus
- **Buyer-optimal** (information design) — setup + claim discovery (5 of 10 transcendental, low Lean ROI)

## Reusable Common/ library

The skill builds up `LeanEconomics/Common/` with lemmas reusable across papers:

- **`Optimization.lean`** — interior maximum on closed intervals (3 layers: topology → witnesses → derivative signs). The `interior_max_of_deriv_signs` theorem is the abstract version of "F'(a) > 0, F'(b) < 0, so the optimum is interior" that appears in most constrained optimization papers.
- **`Algebra.lean`** — ring identities, smoke tests

## Works with

- [leanprover/skills](https://github.com/leanprover/skills) — the official Lean 4 skills plugin (proof methodology, mathlib build, bisection). `lean-econ` handles the economics layer; leanprover skills handle Lean mechanics.
- [qe-prove](https://github.com/briancalbrecht/quantifier-elimination) — QE verifies polynomial claims; Lean goes further (unused hypotheses, hidden lemmas, non-polynomial claims).

## License

MIT
