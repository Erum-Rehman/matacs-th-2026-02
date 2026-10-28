# Write-up

## 1. AI vs rules

AI is used for one step only: reading each proposal and extracting facts (licence, insurance, installs, price, weeks, warranty, team items) into a fixed JSON format, with the exact sentence that supports each value. Proposals are written in free text and in different styles, so fixed rules would break easily. This is where AI adds value.

Everything else is plain code: checking the AI output, flag checks, mandatory pass/fail, weighted scoring, ranking and the reason text.

The line is: **AI reads and describes, code decides.** Decisions must be repeatable, explainable and hard to manipulate. AI output can vary between runs and can be influenced by text inside a proposal, so it should never decide eligibility or scores.

## 2. Consistency and explainability

- The AI runs at temperature 0 with a fixed prompt version and a fixed JSON format.
- Extracted facts are saved. Re-ranking uses the saved facts, so the same proposals always give the same result. If the AI is run again, the new output is compared with the saved one and any difference is flagged.
- Scoring is a fixed formula. The best bid on each criterion gets full points and others get a proportional share:
  - Experience (30): 30 × installs ÷ highest installs
  - Price (25): 25 × lowest price ÷ this price
  - Timeline (15): 15 × fewest weeks ÷ these weeks
  - Warranty (15): 15 × workmanship years ÷ longest workmanship years
  - Team (15): 3.75 each for named lead, crew, safety plan, monitoring plan
- Every score is shown per criterion, and every fact links to its evidence sentence. A reviewer can go from the rank, to the points, to the fact, to the sentence in the proposal.

## 3. Human in the loop

- **Admin** sets the criteria. Each change is saved as a new version.
- **Reviewer** checks the extracted facts against the evidence before results are used, and must review every flagged and ineligible proposal.
- **Flagged** proposals are not scored until a reviewer decides to clear them, ask the bidder for missing information, or exclude them.
- **Ineligible** results must be confirmed by a reviewer before a bidder is excluded.
- A reviewer can **override** any fact, status or rank, but must write a reason.
- The **final award** is always a human decision. The tool only recommends.

The audit log records: criteria version, prompt and model version, raw AI output, validation results, flags, scores, every override (before value, after value, reason), and who did what and when.

## 4. Changing criteria

- Criteria are versioned, never overwritten. Every score is linked to the version used.
- Because facts are saved, re-scoring under a new version is instant and needs no AI call. If a new criterion needs a new fact, only that field is extracted.
- The tool shows the old and new rankings side by side, marks the old one as replaced, and a reviewer must confirm the new one.
- Changing weights after results are visible can be used to favour a bidder, so the change needs a written reason and is clearly logged.
- The formulas compare bids with the best bid, so adding or removing a bidder (for example, clearing a flagged one) changes other scores too. The tool always re-scores the whole set together.

## 5. Risks and safeguards

| Risk | Safeguard |
|---|---|
| Wrong extraction | Every value needs an evidence sentence; reviewer checks facts before use |
| Made-up facts | Code checks each evidence sentence really exists in the proposal; if not, flag. Missing facts are null, never guessed |
| Manipulation inside a proposal | Proposal text is treated as data, not instructions. AI reports instruction-like text and the proposal is flagged. AI never scores, so it cannot be talked into high marks |
| Unclear or conditional wording | Prompt defines each field (for example, the licence must be held by the bidder). Unclear items go to `notes` for the reviewer |
| Bias | AI does not judge quality; code scores only extracted facts with the same formula for every bidder. A well-written proposal should not score higher just because of its writing |
| Poor PDF text (scans, tables) | If extracted text is empty or very short, flag for OCR or manual entry |
| Cost | One AI call per proposal; results are saved, so re-scoring is free |
| Confidentiality | Use an AI provider that does not train on submitted data; keep API keys out of the code |
| Over-trust in the tool | Results are labelled as recommendations; humans confirm every exclusion and make the award |

## Sample result

- **Ranked:** A (98), E (86), C (77)
- **Flagged:** D. It contains an instruction aimed at automated reviewers and gives no completion timeline, while the project must finish within 16 weeks.
- **Ineligible:** B. The bidder does not hold an electrical licence itself; the work depends on a subcontractor to be confirmed after award.

See `ranking.json` for details.

## Assumptions

- "Last 5 years" means from late 2021. C says "since 2020", so not all 9 installs may count. The count is used as stated and a note is added for the reviewer.
- A missing timeline is a flag, not an automatic fail, because the bidder may be able to provide it.
- The four team items are worth equal points.
- Panel warranty is 25 years for every bidder, so only workmanship warranty is scored.
