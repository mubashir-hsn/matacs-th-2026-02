# MATACS-TH-2026-02: AI Proposal Triage Design

**Name:** Mubashar Hassan
**Reference:** MATACS-TH-2026-02
**Time spent:** 40 Minutes

## Files

- [WRITEUP.md](WRITEUP.md): answers to the five questions
- [PROMPTS.md](PROMPTS.md): the prompts I would use
- [ranking.json](ranking.json): my ranking of proposals A to E
- [diagram/process-flow.png](diagram/process-flow.png): process flow diagram

I did not build a prototype. This is design only.

## How I used AI

I used AI to get a first draft of the flow and the prompts, then I went through it and changed things. One thing I disagreed with: the AI first suggested that the model should give each proposal a final score out of 100. I didn't keep that, because the same proposal could get a different score if you run it twice, and nobody could explain why. So now the AI only pulls out facts with quotes, and normal code does the scoring. I also worked out the sample scores by hand to check the numbers in ranking.json.

## Assumptions

- Proposal C has a hidden note asking to be ranked first. I ignore it and flag the proposal.
- If a proposal has no timeline, I flag it. I don't give it zero.
