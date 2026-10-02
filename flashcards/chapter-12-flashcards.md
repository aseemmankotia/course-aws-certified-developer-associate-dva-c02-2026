## Chapter 12: Full Practice Exam Simulation and Test-Taking Strategy — Flashcards

| # | Front (Question) | Back (Answer) |
|---|-----------------|---------------|
| 1 | DVA-C02 exam: number of questions and time limit? | 65 questions, 130 minutes → ~2 min/question. Exam tip: flag uncertain questions and move on rather than stalling; budget the last 10-15 min for review. |
| 2 | DVA-C02 domain weights, highest to lowest? | Development with AWS Services 32%, Security 26%, Deployment 24%, Troubleshooting & Optimization 18%. Development + Security = 58% — review these first if time is short. |
| 3 | IAM evaluation: explicit Deny + explicit Allow, same request — result? | Deny always wins. An explicit deny in ANY applicable identity, resource, SCP, or permissions-boundary policy overrides any Allow. |
| 4 | SQS vs SNS: message delivery model? | SQS = one message consumed by one competing consumer (point-to-point). SNS = fanout, every subscriber gets its own copy. Classic pattern: SNS topic → multiple SQS queue subscribers. |
| 5 | EventBridge vs SNS: when does EventBridge win? | EventBridge supports content-based filtering/routing rules across 100+ AWS/SaaS sources plus a schema registry. SNS is simpler pub/sub with no cross-source rule filtering. |
| 6 | When is Step Functions required over plain Lambda chaining? | When you need visual workflow tracking, branching logic, built-in retry/backoff, parallel states, or long-running orchestration (up to 1 year). Plain chaining has no state tracking. |
| 7 | Max Lambda versions a single alias's routing-config can target? | 2 total (primary + one weighted secondary version). Used for canary/linear shifting via CodeDeploy. |
| 8 | CodeDeploy Lambda config 'Canary10Percent5Minutes' — meaning? | Shift 10% of traffic immediately, wait 5 minutes, then shift the remaining 90%. Pair with CloudWatch alarms for automatic rollback on failure. |
| 9 | Elastic Beanstalk Immutable deployment — rollback behavior? | New version deploys to a separate temporary ASG (double capacity). If health checks fail, the new ASG is terminated instantly with zero impact to the running fleet. Safest, but costs more during deploy. |
| 10 | Exam qualifier 'LEAST operational overhead' points to? | Managed/serverless choices (Lambda, Fargate, managed Step Functions/EventBridge) over self-managed EC2 or manual scripts. It's about ops effort, not cost. |
| 11 | 'MOST cost-effective' vs 'BEST' qualifier — difference? | 'Cost-effective' = lowest cost while still satisfying ALL requirements. 'BEST' = balance cost, performance, complexity, and security together — no single factor dominates. |
| 12 | Strategy for multi-response questions (choose 2 of 5)? | Eliminate clearly wrong options first, then test each remaining pair independently against EVERY stated requirement (cost, security, scale) — both picks must jointly satisfy the scenario. |
