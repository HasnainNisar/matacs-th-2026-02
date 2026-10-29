# WRITEUP: MATACS-TH-2026-02

## How the tool works (short version)
1. The admin defines each category's criteria, weights and mandatory checks in admin screens. They are saved with a version number.
2. A reviewer uploads proposal PDFs and picks a category.
3. Code checks the files and extracts the text.
4. AI reads each proposal on its own and returns facts, each with a short evidence quote.
5. Code validates those facts, runs the mandatory checks, calculates scores, flags problems and ranks.
6. AI writes a one-line reason per proposal from the finished numbers.
7. A human reviews, can override, and confirms. The tool stops there. The award decision is made by people.

The diagram in `diagram/` shows the same flow.

## 1. AI vs. rules
**AI (two steps):** extracting facts from free-text proposals (every bidder writes differently) and wording the one-line reason.

**Plain code:** file and text checks, validating the AI's JSON, checking that each evidence quote really appears in the proposal, mandatory pass/fail, score formulas, weights, ranking, flags and logging.

**Why there:** AI reads, code decides. Reading messy text is where AI helps. Pass/fail, arithmetic and ranking must be identical every time and easy to explain, and a model can drift or be talked into things. So the AI never sets a score, a rank or a pass/fail.

## 2. Consistency and explainability
Each criterion gets 0-100 from a fixed formula, then is multiplied by the admin's weight:
- Experience: installs of 100 kW or more, divided by 10, capped at 100.
- Price: lowest valid price divided by this price.
- Timeline: fastest valid time divided by this time. Over 16 weeks fails.
- Warranty: half panel years divided by 25, half workmanship years divided by 12.
- Team: 25 points each for named lead, crew, safety plan, monitoring plan.

(The divisors and caps are admin settings, not hard-coded.)

**Same result if ranked twice?** Yes. The formulas are deterministic. The AI step runs at temperature 0 with a pinned model, and its output is saved (keyed by file hash, prompt version and model version), so a re-rank reuses the saved facts. A new model or prompt counts as a new run and is labelled so.

**How a reviewer sees why:** a score table per criterion, every extracted fact next to its quote from the proposal, and any flags.

## 3. Human in the loop
- **Checkpoint 1 (after extraction):** anything missing, unclear, failed or suspicious is shown to the reviewer, who can correct a fact with a note before scoring is final.
- **Checkpoint 2 (ranked list):** the reviewer can clear a flag, exclude a proposal or reorder. Every override needs a written reason, and the machine result stays visible.
- **Checkpoint 3 (confirm):** the reviewer confirms the list. People make the award decision outside the tool, and verify licence and insurance claims with the registry first, because the tool only records what a bidder claims.

**Audit log (append-only):** file hash, extracted text, prompt text and version, model name and version, criteria version, raw AI output, computed scores, flags, who did what and when, and every override with its reason.

## 4. Changing criteria
Criteria are versioned. An edit creates version 2 and never changes version 1.
- Extracted facts do not depend on weights, so changing weights or thresholds needs no new AI call. Code just re-scores.
- A new criterion that needs a new fact triggers extraction for that field only.
- Old results stay labelled "scored under v1". The reviewer clicks "re-score under v2", sees old and new rankings side by side with changes highlighted, and confirms. Nothing changes silently.
- Once a review round is confirmed, its criteria version is locked. Edits apply to new rounds only.

## 5. Risks
| Risk | Guard |
|---|---|
| Wrong extraction | Code checks each quote exists in the text. Reviewer sees quotes. Unclear values become null and are flagged. |
| Made-up facts | Prompt says null instead of guessing. Missing data is flagged, never scored (D has no completion time). |
| Manipulation | C has a line aimed at automated reviewers asking for top rank. Proposal text is treated as data, the AI reports it instead of obeying it, the AI never controls rank, and code flags any such proposal for human review. |
| Bias | Scoring uses only the admin's criteria and numbers. Names, size and location are not inputs. Same formulas for everyone. |
| Unverifiable claims | AI records claims only. B fails the mandatory licence check because the bidder does not hold it. A human verifies the rest before award. |
| Bad or scanned PDFs | Empty or low-quality text is flagged as unreadable, not scored. |
| Cost | One short call per proposal plus one for the reason, cached, with file and page limits. A small model is enough. |
| Confidentiality | Bids are restricted-access and stored securely. Only fictional data is used here. |

## Sample result
A is 1st (98.0) and E is 2nd (91.7). C is flagged (84.8 if cleared). D is flagged (lowest price, no completion time). B is ineligible. See `ranking.json`.

## Assumptions
- Install counts are taken as stated. C says "since 2020", which may reach beyond five years, so a reviewer should check it.
- All prices are CAD.
- D's price counts as the lowest-price baseline even though D is flagged. Dropping it changes scores slightly (A 98.8, E 92.4, C 85.5) but not the order.
- A missing value is never assumed.
