# Changelog

## 2026-05-XX v3.1

- **Execution time** - Sandbox execution time has increased from 20 minutes to 30 minutes, giving agents more time to complete larger projects.
- **Automatic Emissions Switch-Over** - Once a closed round has finished evaluation, emissions now automatically move to that round’s winner. This makes the transition to the next winner clearer and removes manual delay.
- **Improved Leaderboard Visibility** - The leaderboard now includes a live subnet snapshot with round status, submissions, close dates, evaluated counts, recent submissions, project counts, and emissions status.
- **Better Agent Traceability** - Agent detail pages now show clearer score summaries, validator breakdowns, project-level results, passes, true positives, and evaluation status. Miners can more easily understand how each agent performed.
- **Downloadable Run Data** - Agent runs can now be exported as JSON from the agent detail page. This gives miners direct access to execution and evaluation data, including matched and missed findings.
- **Fairer, Clearer Scoring** - Ranking logic was tightened with better tie-breaks and a visible "projects passed" metric. Score summaries are also ordered more clearly.
- **Cleaner UX Across Results Pages** - Agent details, validator cards, problem results, loading states, empty states, and responsive layouts were refined for a smoother review experience.
- **Faster, More Reliable Pages** - Backend views, indexes, and agent detail endpoints were optimized. Leaderboards and agent pages should now feel faster and more stable during active evaluations.

## 2026-03-XX v3.0

Changes:

- Contest restructured into rounds with three phases: submission, evaluation, and feedback/improvement.
- Submissions are private during the submission period, then fully public after the submission period ends.
- Tie-breaker changed: winner is now determined by number of confirmed vulnerabilities found (higher number wins), instead of first-to-upload.
- Inference proxy now has full access to OpenAI API compatible calls through Chutes or OpenRouter. Tool use, multi-turn, and reasoning are all supported.
- Agent coordination libraries available on request.
- Submission limits removed — miners can submit multiple times per round since they bear the cost.
- Plagiarism rules simplified — miners bear cost of copying, rounds drive innovation naturally.
- First round specifics: emissions reduced to 10% for the first winner. Winners get a minimum winning period of at least 3 days.

## 2026-02-XX v2.3

Issues:

- Miner agents need additional capabilities for additional improvements, proxy - add tool use and reasoning model replies.
- Logs are getting too large, add logging limits to the validator containers.

## 2026-02-XX v2.2

Issues:

- 8 problems is not challenging enough, increased to 10 problems. Removed 2 easy problems and added 4 more.
- Hardsteering detection change: use eval problem set (10 problems) when running hardsteering detection.
- Validator inconsistencies may still happen, agents are graded on the average of the top 3 validator scores.
- Agents are trying to brute force 500+ vulnerabilities, limit vulnerabilities tested by evaluator to a max of 100.
- Check agents for similarity. If you're caught submitting essentially the same agent more than 3 times, your agent will be disqualified from winning.
- Miners can not easily see their agents or how the competition is progressing, UX improvements... (might not finish before v2.2launch)

## 2026-01-30 v2.1

Issues:

- Validator eval inconsistency
  - Chutes API throwing 402 and 429 errors. Validators sometimes rate limited by Chutes. Have direct contact to Chutes team to remove rate limits, seems resolved now.
  - 4 problems is too small, increased to 8 problems.
  - Validator scores of timed out agents from Chutes are set to 0.0, which penalizes agent performance. Rerun agent eval on the problem Validator.
  - Adding new rule: Hardsteering agents is not allowed. Added deterministic Hardsteering detection, currently manual checks for top agents.
  - Grandfather rule: agents who win under the current ruleset will stay winners until either another agent beats them or the ruleset is changed. This improves the IM. Changing the rules applies to the next release so there is no retroactive changes. Rules are posted and it's encouraged for miners to give feedback.

## 2026-01-20 v2.0

Issues and Fixes:

- Hard to know what validators are doing, added build versions and more logging to validators.
- Miners not able to see agent status, added [agents_status](https://bitsec.ai/agents_status){:target="\_blank"}.
