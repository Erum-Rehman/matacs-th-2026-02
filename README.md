# MATACS-TH-2026-02: AI Proposal Triage Design

- **Name:** ERUM REHMAN
- **Reference:** MATACS-TH-2026-02
- **Time spent:** About 2 hours

## How I used AI tools

I used Claude to help understand the brief, structure the scoring table, draft the prompt and write-up, and generate the flow diagram with Graphviz. I checked every extracted fact against the sample proposals myself and reviewed all text before committing.

One place where you disagreed with or corrected the AI's output, and why.

The AI first suggested scoring price as lowest price ÷ this price. I disagreed: with bids this close, it gave only 0.6 points of difference across a 25% criterion, so price barely mattered. I changed it to score savings against the budget ceiling, which better reflects the admin's "lower total cost within the ceiling" goal. The ranking stayed the same, but the scores now show the real price gap.

## Files

- [WRITEUP.md](WRITEUP.md): answers to the five questions, assumptions and sample result
- [PROMPTS.md](PROMPTS.md): the AI extraction prompt and how its output is checked
- [ranking.json](ranking.json): expected ranking of proposals A to E
- [diagram/flow.png](diagram/flow.png): process flow from upload to ranked output

## Design in one line

AI only reads proposals and extracts facts with evidence; plain code checks, scores and ranks them; a human reviews flags, can override with a reason, and makes the final decision.
