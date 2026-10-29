# MATACS-TH-2026-02

**Full name:** Hasnain Ali
**Reference:** MATACS-TH-2026-02 (AI Proposal Triage Design, Associate Software Developer)
**Time actually spent:** "about 3 hours, over 2 days" - be honest, even if it is more than 60 minutes

## Files
- [WRITEUP.md](WRITEUP.md): answers to the five questions
- [PROMPTS.md](PROMPTS.md): extraction prompt and reason-writer prompt, as I would send them
- [ranking.json](ranking.json): expected ranking of proposals A-E
- [diagram/process-flow.png](diagram/process-flow.png): process flow from upload to ranked output, showing AI steps, code steps and human checkpoints

## How I used AI tools
I used Claude to help draft the design, scoring formulas, prompts, write-up and diagram, and I reviewed and edited each file before committing it. one real place where you disagreed with or corrected Claude's output, and why. Example if you made the warranty edit: "Claude's first draft divided workmanship years by 12, which is just the longest warranty in this sample, so the formula was tuned to the data. I changed it to divide by the best valid bid (or an admin cap) so it works for any category. The scores for A-E do not change."
