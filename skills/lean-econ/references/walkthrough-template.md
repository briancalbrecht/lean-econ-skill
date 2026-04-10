# Walkthrough template

Read this file when writing a walkthrough document. The walkthrough is the
primary review artifact — the user reads it to verify that the Lean theorem
says what the paper says, without trusting the person who wrote the code.

## File location

`docs/<paper>-<claim>-walkthrough.md` in the Lean project root.

## Template

```markdown
# <Claim name>: paper ↔ Lean

A line-by-line correspondence between <paper description> and its Lean 4
formalization.

- **Paper source:** `<tex_path>` (lines N–M)
- **Lean source:** `<lean_file>`
- **Axioms used:** <axiom list> (no `sorry`)

---

## What the theorem actually says

<State the paper's claim in math, then the Lean statement, explain the
≤ vs > flip or other encoding choices>

---

## Side-by-side bijection

| Paper | Lean |
|-------|------|
| <paper sentence or equation> | <tactic or hypothesis that encodes it> |
| ... | ... |

---

## Linear walkthrough of the proof

### Paper
> <quoted paper proof>

### Lean
```lean
<proof with inline comments mapping each tactic to the paper step>
```

---

## The load-bearing hypothesis

<Explain the negative control: which hypothesis, why it matters, what
the witness is>

---

## Verifying it yourself

```bash
cd <lean_project_root>
lake build <module.path>
cat > /tmp/verify_<claim>.lean <<'EOF'
import <full.module.path>
open <namespace>
#check @<theorem_name>
#print axioms <theorem_name>
EOF
lake env lean /tmp/verify_<claim>.lean
```

Expected output: <paste actual output>
```

## Important

- Copy proof text **verbatim from the `.lean` source file**, not from
  `#check` or Lean's pretty-printer. The pretty-printer normalizes Unicode
  (`ℝ` → `R`) and renames lemmas (`le_div_iff₀` → `le_div_iff`).
- `lake env lean` takes a **file path**, not `-c` inline code.
- For "core only" or "logic only" claims, the walkthrough must state
  explicitly what Lean verified vs what it assumed as a hypothesis.
