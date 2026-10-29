<<<<<<< HEAD
# MATACS-TH-2026-02: AI Proposal Triage Design

**Name:** Hamza Jabbar
**Reference:** MATACS-TH-2026-02
**Time actually spent:** ["About 1 hours including review and edits"]

## Summary
A design for an AI-assisted proposal triage tool. AI extracts facts and quotes from proposals;
plain code applies the mandatory checks, scores and ranks; people review, override and decide.
Proposals A to E are referred to by letter only.

## Files
- [WRITEUP.md](WRITEUP.md): answers to the five questions (under 1,000 words)
- [diagram/process-flow.png](diagram/process-flow.png): flow from upload to ranked output, showing AI steps, rule steps and human checkpoints
- [PROMPTS.md](PROMPTS.md): the extraction and checklist prompts, as I would send them
- [ranking.json](ranking.json): expected ranking of A to E

## How I used AI tools
I used Claude to help structure the design, draft the prompts, and check the sample ranking against the stated requirements. I reviewed the output rather than accepting it blindly: in particular, I treated Proposal B as ineligible because the bidder itself does not confirm that it holds the required Ontario electrical contractor licence; a subcontractor whose identity is still to be confirmed is not the same evidence. I also kept Proposal D out of the ranked list because its completion schedule is not stated and needs human clarification.

=======
# matacs-th-2026-02
>>>>>>> 7ba724789d0fbc54878b86863aa98a89ee7a043d
