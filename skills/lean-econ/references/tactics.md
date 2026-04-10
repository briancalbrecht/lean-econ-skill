# Tactic reference for economics proofs

Read this file when you need tactic guidance for a specific proof pattern.

## Core tactics

Most economics-paper proofs use combinations of:

- `linarith` / `nlinarith` — linear and nonlinear arithmetic over ordered fields
- `ring` / `ring_nf` — ring identities and normalization
- `field_simp` — clear all denominators in a field expression
- `positivity` — prove `0 < expr` or `0 ≤ expr` from structure
- `norm_num` — concrete numeric computation (rationals, naturals)
- `by_cases` / `rcases` — case splits and destructuring
- `subst` — substitute definitional equalities
- `have` — introduce intermediate facts

Try these before heavier automation (`simp`, `omega`, `polyrith`, `decide`).

## Claims involving `max` / `min`

Lean handles `max`/`min` much more cleanly than QE. QE must expand into
4-6 polynomial sub-cases. Lean states the claim directly and proves it:

```lean
simp only [max_def]   -- or [min_def]
split_ifs <;> linarith
```

`split_ifs` generates all if-then-else branches; `linarith` closes each.

## Clearing denominators

**Pattern A — division at top level** (`a ≤ b / c` or `a / c ≤ b`):
```lean
have hd : (0 : ℝ) < denom := by linarith
rw [le_div_iff₀ hd]
nlinarith [...]
```

**Pattern B — division nested inside a larger expression** (e.g.,
`ξ * (expr / denom) - stuff < 0`): `le_div_iff₀` won't match at top
level. Multiply the whole expression by the positive denominator:
```lean
have hd : (0 : ℝ) < denom := by linarith
have hd' : denom ≠ 0 := ne_of_gt hd
suffices h : whole_expr * denom < 0 from by
  rcases mul_neg_iff.mp h with ⟨_, hb⟩ | ⟨ha, _⟩
  · linarith
  · exact ha
have : a * (b / denom) * denom = a * b := by
  rw [mul_div_assoc', div_mul_cancel₀ _ hd']
nlinarith [...]
```

Prefer this over `field_simp` when possible — `field_simp` can produce
goals that `nlinarith` struggles with.

## Factor positivity from hypotheses

`positivity` can't decompose expressions where a factor's sign requires
arithmetic from hypotheses. Pre-derive the sign:

```lean
-- positivity can't see that 1 - m₂*(1-β₁) ≥ 0
have h : 0 ≤ 1 - m₂ * (1 - β₁) := by nlinarith [sq_nonneg (1 - m₂)]
positivity
```

## Reserved identifiers

Lean 4 reserves `λ`, `fun`, `let`, `do`, `if`, `match`, `where`, `return`.
Use `lamPar`, `delPar`, etc. for Greek-letter parameters that collide.
Unicode identifiers like `ξ`, `η`, `βj`, `πk` are fine.
