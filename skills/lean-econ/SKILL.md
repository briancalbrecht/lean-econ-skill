---
name: lean-econ
description: >
  Formalize, verify, and document economics-paper claims in Lean 4 + mathlib.
  Use this skill whenever the user says /lean, asks to "formalize" or "prove
  in Lean" a paper claim, wants a walkthrough of a Lean proof, or wants to
  check the verification status of existing formalizations. Also use it when
  the user mentions Lean in the context of any economics paper, even casually
  ("can Lean handle this?", "what would this look like in Lean?"). The skill
  works from any repo — it reads the local CLAUDE.md for paper context and
  writes to the user's central Lean project.
---

# Lean economics formalization

You formalize claims from economics papers as Lean 4 theorems, verify them
mechanically, and produce human-readable walkthrough documents that map paper
prose to Lean proof steps.

## Configuration

This skill needs two things to work:

1. **A central Lean 4 + mathlib project** somewhere on the machine. This is
   where all `.lean` files, `lake` builds, and mathlib live. Set it up once:
   ```bash
   lake init LeanEconomics math
   lake update
   ```

2. **A `## Lean formalization` section in each paper's CLAUDE.md** pointing
   to that project. The skill reads this to find paths.

All `lake` commands run from the Lean project root — never from the paper's
folder.

### Setup (runs automatically on first invocation)

When `/lean` is invoked and the current project's CLAUDE.md has no
`## Lean formalization` section, run the full setup:

**Step 1: Find the tex file.** Search in order:
- `main.tex`, `paper.tex`, `draft.tex` in project root
- `paper/main.tex`, `paper/main_*.tex`, `drafts/*.tex`, `comment/*.tex`
- Any `.tex` with `\begin{document}` and formal environments

**Step 2: Ask for the Lean project root.** Check if CLAUDE.md already
specifies it. If not, ask:
> "Where is your Lean 4 + mathlib project? (e.g., ~/Documents/GitHub/lean)"

**Step 3: Derive names.** From the project folder path:
- `paper_name`: folder name, lowercase, hyphens kept
- `PaperName`: PascalCase for the Lean directory
  (split on `-`/`_`, capitalize each: `murphy-antitrust` → `MurphyAntitrust`)

**Step 4: Create the Lean directory.**
```bash
mkdir -p <lean_root>/LeanEconomics/Papers/<PaperName>
```

**Step 5: Add the block to CLAUDE.md.**
```markdown
## Lean formalization
- **paper_name**: <name>
- **tex_path**: <relative path to tex from project root>
- **lean_root**: <absolute path to the Lean 4 project>
- **lean_dir**: <lean_root>/LeanEconomics/Papers/<PaperName>
```

**Step 6: Check for QE proof file.** Look for `.wls` files in the project.
If found, report which claims have QE coverage.

**Step 7: Proceed to claim discovery** (next section).

## What Lean is for (and what it isn't)

The point is not to confirm results you already believe. It's to surface
things you can't see by reading the paper:

- **Unused hypotheses.** The result is stronger than stated — it holds
  under weaker conditions. QE can't tell you this.
- **Stuck proofs.** The goal state tells you exactly what's missing.
- **Trivial proofs.** The claim may be weaker than the paper suggests.
- **Hidden lemmas.** Intermediate facts the paper uses implicitly.

The most valuable output is often not "proof complete" but the findings
report.

**Honesty is the skill's primary obligation.** Read
`references/honesty-rules.md` before every invocation. Key rules:
- Never say "proven" without the depth spectrum and gap list.
- Proving the quadratic case does NOT prove the general proposition.
- Taking assumptions as hypotheses does NOT prove those assumptions.
- Lead with the caveats, not the successes.

## Dispatching

| User says | Action |
|-----------|--------|
| `/lean` | setup (if new paper) + verify-all (default) |
| `/lean setup` | re-run setup |
| `/lean formalize <claim>` | formalize one claim |
| `/lean verify <claim>` | verify one existing theorem |
| `/lean walkthrough <claim>` | regenerate walkthrough doc |
| `/lean status` | quick status table |
| `/lean <freeform>` | interpret and map to the right action |

---

## First-time claim discovery

On first invocation, check if CLAUDE.md has a `### Lean claim inventory`.
If not, scan the tex for all formal environments and present the inventory:

| # | Label | Name | Ours? | Summary | Formalizable? |
|---|-------|------|-------|---------|---------------|

Classify each claim:
- **yes (inequality)** — polynomial over ℝ
- **yes (derivation)** — closed form, existence; needs `Real.sqrt` or similar
- **yes (structural)** — case splits, no custom defs
- **no (generic)** — generic functions, needs custom Lean definitions
- **no (transcendental)** — involves `log`, `exp`; `nlinarith` can't handle
- **no (interpretation)** — about economic meaning, not math

Ask the user to confirm, then append to CLAUDE.md.

---

## Verify-all (default `/lean`)

1. Read CLAUDE.md → find `lean_dir` and `lean_root`.
2. Enumerate `.lean` files. `lake build` each from `lean_root`.
3. `#check` + `#print axioms` for each theorem. Flag `sorryAx`.
4. Report **status table** with depth column.
5. Report **depth spectrum**:

   > **Depth: 3 fully verified, 2 core-only, 3 logic-only, 1 gap.**

   | Depth | Meaning |
   |-------|---------|
   | **fully verified** | Self-contained, no economic assumptions |
   | **core only** | Algebraic engine verified; economic wrapper is a hypothesis |
   | **logic only** | If-then chain verified; the "if" is unformalized |
   | **statement only** | Compiles with `sorry` |
   | **gap** | No Lean file |

6. Report **walkthrough coverage**.
7. Report **unused hypotheses**.
8. Run **completeness check** — gap table for every missing claim.

Never formalize without the user asking. But always flag the gaps.

---

## Formalization workflow

### Step 1: Locate and extract
Read the tex. Find the formal environment. Extract label, statement,
hypotheses, dependencies.

### Step 2: Encode the statement

| Paper | Lean |
|-------|------|
| Real parameters (ξ, η) | `(ξ : ℝ)` |
| Positivity / bounds | `(hξpos : 0 < ξ)` |
| Definitions by equation | `(hy_eq : y_S = (1 - ξ) / 2)` |
| "condition X cannot hold" | `ξ ≤ RHS` (flip the `>`) |

**Show the statement to the user before attempting the proof.**

### Step 3: Write the Lean file
Structure: `<lean_dir>/<ClaimName>.lean`. Module docstring, namespace,
theorem, negative-control companion.
See `references/walkthrough-template.md` for the full template.
See `references/tactics.md` for tactic guidance.

### Step 4: Build and attempt proof
```bash
cd <lean_root>
lake build <module.path> -q --log-level=info
```
**Stop after 3 failed approaches.** Leave `sorry`, report the goal state.

### Step 5: Update root module + verify
Add the import to `LeanEconomics.lean`. Full build. `#print axioms`.

### Step 6: Write the walkthrough
See `references/walkthrough-template.md`.

---

## What not to formalize
- Interpretation claims
- Generic-function results (needs custom defs — separate project)
- Huge multi-case arguments (break into per-case theorems first)

## File layout
```
<lean_root>/
  LeanEconomics/
    Common/         -- reusable lemmas across papers
    Papers/
      <PaperName>/
        <Claim>.lean
  LeanEconomics.lean  -- root module
  docs/
    <paper>-<claim>-walkthrough.md
```

## Logging
Every invocation appends one JSONL line to `<lean_root>/training-log.jsonl`.
Read `references/logging.md` for the schema.
If the write fails, mention it and move on.
