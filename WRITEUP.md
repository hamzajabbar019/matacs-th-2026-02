# Write-up: AI Proposal Triage (MATACS-TH-2026-02)

**Core idea:** AI reads and extracts. Plain code decides, scores and ranks. People confirm.
The flow is in `diagram/`. Prompts are in `PROMPTS.md`. Expected result is in `ranking.json`.

**Assumptions** (not asked by email, so stated here):
- A "named" lead counts if the proposal says a specific person leads, even if the name is not in the summary text.
- Price and timeline are scored relative to the best eligible bid; warranty and experience use fixed scales.
- Mandatory criteria must be held by the bidder itself, not promised or subcontracted later.

## 1. AI vs. rules

| Step | AI or rules | Why |
|---|---|---|
| Upload checks, category choice, load admin criteria | Rules | Simple and exact |
| PDF to text (OCR only if scanned) | Rules / tool | Deterministic |
| Pull facts and quotes out of messy prose | **AI** | Wording varies too much for fixed patterns |
| Check quotes exist in the source, numbers are sane | Rules | Catches made-up facts |
| Mandatory pass/fail, budget and deadline checks | Rules | Must be exact and explainable |
| Price, timeline, warranty, experience scores | Rules (formulas) | Maths should never be "guessed" |
| Team and approach | **AI** answers a true/false checklist, rules turn it into points | Needs reading, but AI never invents a number |
| Weighted total, ranking, tie-break | Rules | Repeatable |
| Final award | Human | Accountability |

The line: AI is used where the job is **understanding text**. Anything that is a decision or a calculation is code.

## 2. Consistency and explainability

- Temperature 0, fixed model version, fixed prompt version, JSON-only output.
- Extracted facts are **saved** with the proposal. Re-ranking reuses saved facts, so the **ranking is identical** every time. Only a re-extraction could change wording, and the quote check plus the rules catch changes that matter.
- Scoring formulas are plain code (see `prototype/score.py`). Same facts plus same criteria version gives the same score.
- The reviewer sees, for each proposal: score per criterion, the fact used, and the exact quote from the PDF with a link to the page. Reasons are built from those facts, not free-written by AI.
- Example: A ranks first because it passes both mandatory checks, has the most qualifying installs (14), the shortest valid schedule (12 weeks) and a full team plan.

## 3. Human in the loop

Checkpoints:
1. **Before AI runs:** reviewer confirms category and files.
2. **After scoring:** reviewer sees three lists: ranked, flagged, ineligible. They can confirm, **override a fact** (for example fix a wrong number, with a required comment), request clarification from a bidder, or move a proposal between lists.
3. **Final award:** always a person. The tool only recommends an order.

Audit log (append-only): who uploaded what (file hash), criteria version, model and prompt version, raw AI output, validated facts, every score, every flag, every human change with name, time and comment, and the final decision.

## 4. Changing criteria

- Every criteria edit creates a **new version**; old versions are never changed.
- Each score is stamped with the version it used, so history stays truthful.
- Saved facts stay valid, so changing a weight only needs a **quick re-score**. No AI is called unless the admin adds a criterion that needs a new extraction, and then only that criterion is extracted.
- The reviewer sees "scored with v1, current is v2" and chooses to re-score. Rankings are never silently mixed across versions.
- Once an award is confirmed, it is locked to the version used.

## 5. Risks and safeguards

| Risk | Safeguard |
|---|---|
| **Wrong extraction** (misread number, wrong year window) | Evidence quote is required and checked against the source; low confidence or unverified fields are flagged; reviewer sees the quote beside the value |
| **Made-up facts** | "Not stated" is an allowed answer; no outside knowledge; a quote that is not in the text is rejected |
| **Manipulation / prompt injection** | Proposal text is wrapped as untrusted data; a code scan plus the AI report any text addressed to reviewers or AI; the proposal is **flagged and never auto-ranked**; the hidden instruction has zero effect on the score |
| **Bias** (favouring big firms, polished writing, certain names) | Scoring uses facts and formulas, not tone; bidder names are not used in scoring; reviewer can see all inputs; spot-check results periodically |
| **Missing or vague data** | Flagged for clarification, never silently scored as zero or full |
| **Cheap but non-compliant bid skews scores** | Ineligible bids are excluded from the price and timeline baselines |
| **Cost / overload** | One extraction call per proposal; facts cached; re-scoring is free; file size and page limits; retry once only |
| **Service or model outage** | Proposal stays in "pending"; nothing is auto-decided; manual entry of facts remains possible |
| **Privacy** | Only the proposal text is sent; no personal data beyond what the bid contains; logs access-controlled |

## How the sample set comes out

| | Result | Main reason |
|---|---|---|
| A | Rank 1 (98.0) | Strongest on experience, timeline and team |
| E | Rank 2 (85.7) | Best warranty, but less experience and slower |
| C | Flagged | Hidden instruction to reviewers; on merits it would be third (81.8) |
| D | Flagged | No schedule stated, so the 16-week limit cannot be confirmed |
| B | Ineligible | Licensed electrician is an unnamed subcontractor, so it fails the mandatory licence check despite the lowest price |

