# Write-up

My basic idea: the AI reads the proposals and pulls out the facts. Normal code does the checking, scoring and ranking. A person looks at the result and makes the final decision.

## 1. AI vs. rules

**AI is used in two places:**
- Step 5: reading the proposal text and pulling out facts like licence, price, weeks, warranty and team details. Every company writes differently, so this is hard to do with simple rules.
- Step 11: writing a one-line reason for each position, using numbers the code already calculated.

**Everything else is code or rules:** upload, loading the criteria, getting text from the PDF, checking the AI output, pass/fail checks, scoring, sorting and logging.

I drew the line there because reading messy text is something AI is good at, but pass/fail checks and maths need to give the same answer every time. I also don't want the AI deciding who wins.

## 2. Consistency and explainability
- I set temperature to 0 and save the prompt version. But AI can still change a little between runs, so I don't let it calculate the score.
- The score is calculated in code from the extracted facts. Same facts means same score.
- The scoring rules are simple (weights come from the admin screens):
  - Experience: bands based on how many qualifying installs.
  - Price: lowest eligible price divided by this bidder's price.
  - Timeline: shortest weeks divided by this bidder's weeks.
  - Warranty: half for panel years, half for workmanship years (with a cap).
  - Team: 4-item checklist (named lead, crew, safety plan, monitoring plan).
- If the same proposals are ranked twice, the ranking should be the same. If the AI extracts one fact differently the second time, the tool shows that difference to the reviewer.
- The reviewer sees a table with each criterion, the value found, the quote from the PDF, the points and the weight. So if someone asks why a proposal is 2nd, they can just look at the table.

## 3. Human in the loop
- Step 12: a reviewer checks the evidence and deals with all flagged and ineligible proposals. They can confirm or override any rank, but they have to type a reason.
- Step 13: a person makes the final award decision. The tool never awards anything.
- What I log: who uploaded, file hash, criteria version, prompt version, model name, raw AI output, extracted facts, every score, flags, the original ranking, every override (who, when, why) and the final decision.

## 4. Changing criteria
- Every set of criteria has a version number, and each result saves which version it used.
- Old results are not changed quietly. They stay marked as "scored with v3", for example.
- If an admin edits criteria or weights, the tool tells the reviewer that existing results are out of date and offers to re-score.
- The extracted facts are saved, so re-scoring just runs the code again. No new AI calls are needed. If the admin adds a brand new criterion, the AI only has to extract that one new thing.
- The old and new rankings are both kept so they can be compared.
- My assumption: after the final decision is made, the criteria for that project are locked. The admin would need to make a new version.

## 5. Risks
- **Wrong extraction:** every fact needs an exact quote, and code checks that the quote is really in the PDF text. If not, it gets flagged. The reviewer can also see the quote.
- **Made-up facts:** the prompt tells the AI to use null if something isn't stated. Missing info gets flagged, not guessed.
- **Manipulation (proposal C):** proposal text is treated as data, not as instructions, and the prompt says so. The code also searches for phrases like "rank this first". If found, the proposal is flagged and the text is ignored. Since the code does the scoring, that text can't raise the score anyway.
- **Bias:** the AI only extracts facts linked to the criteria and never gives the score. Company name and size are not used in scoring. The reviewer can check the table.
- **Wrong pass/fail:** these are simple rules. The licence has to be held by the bidder. A subcontractor "to be confirmed" fails (proposal B).
- **Missing data (proposal D):** flag it and ask the bidder for the info. Don't give zero and don't guess.
- **Cost:** one AI call per proposal for extraction, and the results are saved and reused. A smaller model should be enough.
- **Privacy:** only the proposal text is sent, and logs are only visible to authorised users.
- **People trusting the ranking too much:** the screen says it is a suggestion only, and the human decides.

## How the sample set comes out
- A is 1st (98.0). Most experience, fastest, good price.
- E is 2nd (85.7). Best workmanship warranty, but fewer installs than A.
- C is flagged (81.8). It has a hidden note asking to be ranked first, which I ignore. Without it the score would still put C 3rd, but a human should check it.
- D is flagged. It gives no completion time so the timeline can't be scored.
- B is ineligible. The bidder does not hold the licence itself (electrical work goes to a subcontractor). Its low price doesn't matter.

Two small notes: C says "since 2020", which might include a year older than 5 years, so I mention it for the reviewer. And for the price score I use the lowest price among proposals that are not ineligible.
