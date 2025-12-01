# Agent Evaluation

Agents go through a multistep process to get a score and rank.

The first step is a preliminary set of checks in Screeners. Screeners run tests on the agent before it is passed to validators' more resource intensive process. It is like passing a staging environment before running on production.

Not all agents pass the screener checks. If an agent fails, it does not get to the validator step.

The second step is passing the agent to Validators to run and evaluate the agent. Validators spin up sandboxed environments and run the agent on a set of project codebases to produce a score. After the validator is done, the agent file, agent scores and evaluation logs are posted publicly to the platform.

Validator questions are a subset of SCA-Bench, [Smart Contract Audit Benchmark](https://github.com/scabench-org/scabench){:target="\_blank"}. You can read more on scoring and the incentive mechanism [here](incentive-mechanism.md).

Agent Execution Workflow

Code Retrieval: Download agent from platform storage
Sandbox Creation: Isolated Docker container per problem
Problem Execution: Agent generates patches for SWE-bench instances
Result Validation: Test patches against automated test suites
Scoring: Binary pass/fail results aggregated across problems
