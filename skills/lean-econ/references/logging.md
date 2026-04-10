# Training log specification

The training log feeds an optional optimization script (GEPA's
`optimize_anything`) to improve this skill automatically. Every invocation
appends one JSONL line. Failures are the most valuable training signal.

## File location

`<lean_root>/training-log.jsonl`

where `<lean_root>` is the Lean project root from the paper's CLAUDE.md
(`lean_root` field under `## Lean formalization`).

If the write fails (permissions, wrong cwd), mention it and move on.

## Schema

```json
{
  "timestamp": "2026-04-09T12:00:00Z",
  "action": "formalize",
  "paper": "waterbed",
  "claim": "beta_neg_at_eta_zero",
  "module": "LeanEconomics.Papers.Waterbed.BoundaryEta0",
  "build_exit": 0,
  "sorry_free": true,
  "walkthrough": true,
  "error_summary": null,
  "tactics_used": ["linarith", "nlinarith", "mul_div_assoc'"],
  "iterations": 3,
  "notes": "Needed Pattern B for nested denominator"
}
```

## Fields

- `action`: `"formalize"` | `"verify"` | `"walkthrough"` | `"status"` | `"discovery"`
- `build_exit`: exit code from `lake build` (0 = success), or `null`
- `sorry_free`: `true` / `false` / `null` (from `#print axioms`)
- `walkthrough`: whether a walkthrough doc was produced or exists
- `error_summary`: `null` on success; one-line description on failure
- `tactics_used`: mathlib tactics in the final proof (empty for non-formalize)
- `iterations`: edit-build cycles (1 = first try)
- `notes`: free-text observation about what was surprising or hard
