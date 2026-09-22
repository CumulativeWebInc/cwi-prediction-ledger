# Prediction Ledger v1 — schema & validation rules

Owner: `agent:CWI_Athena` · Standing mandate: **never claim what she hasn't measured.**
Purpose: every Athena (and Athena-agent) prediction is registered **before**
its outcome is knowable, then scored when it resolves. Judgment compounds.

## Entity: prediction

```json
{
  "id": "prd_<ulid>",
  "statement": "plain-language claim with named resolution condition",
  "agent": "agent:CWI_Athena",
  "confidence": 0.75,
  "made_at": "2026-09-22T10:40:00Z",
  "deadline_at": "2026-09-23T14:00:00Z",
  "resolution": "PRD state: <criterion>; evidence required: <what counts>",
  "status": "open | resolved",
  "resolved_at": null,
  "outcome": null,
  "evidence": null,
  "brier": null
}
```

- `confidence`: probability in **[0.01, 0.99]**. Extremes (0.01/0.99) are
  allowed but flagged in the report — overconfidence is a calibration smell.
- `made_at`: set **by the tool from the clock**, never user-supplied.
  A registration whose `deadline_at` is not strictly after `made_at` is
  rejected (anti-backfill: you cannot predict the past).
- `status: resolved` requires `outcome` (0 or 1) **and** non-empty `evidence`
  (a file path, URL, or ledger event id — not vibes). Resolved records are
  **immutable**: `resolve` twice or edit-after-resolve is rejected.
- `brier`: `(confidence − outcome)²`, computed at resolution. Mean Brier per
  agent ≤ 0.25 beats the climatology baseline (always-0.5). Lower is better.

## Commands (`node ledger.js`)

| command | effect |
|---|---|
| `add --statement S --agent A --confidence P --deadline ISO --resolution R` | register; id = `prd_<ulid>` |
| `resolve --id ID --outcome 0|1 --evidence E` | resolve with evidence |
| `report [--agent A]` | scorecard: n resolved, mean Brier, vs baseline, per-agent table, calibration bins |
| `list [--status open\|resolved]` | all predictions, JSON |
| `validate` | re-check every record against the rules; exit non-zero on violation |

## Calibration bins

10 bins (0.0–0.1 … 0.9–1.0). Each bin shows: mean confidence, observed hit
rate, n. A calibrated agent sits on the diagonal; with tiny n the bins are
honest about it (`n` shown, no smoothing).

## Kill rules

- Any validation failure on `validate` → ledger goes advisor-mode: no new
  registrations until the corruption is root-caused and reported plainly.
- A prediction resolved without evidence → **verification failure** → per
  Athena's mandate, immediate advisor mode pending review with KingCode.
