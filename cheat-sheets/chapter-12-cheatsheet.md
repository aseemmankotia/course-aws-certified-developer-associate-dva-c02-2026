# Chapter 12 Cheat Sheet: Practice Exam & Strategy

## Time Budget
| Metric | Value |
|---|---|
| Questions | 65 |
| Time | 130 min |
| Per question | ~2 min |
| Strategy | Flag & skip if stuck >90 sec; review flagged items with remaining time |

## Domain Weight & Review Priority
| Domain | Weight | Review Priority |
|---|---|---|
| Development with AWS Services | 32% | 1st |
| Security | 26% | 2nd |
| Deployment | 24% | 3rd |
| Troubleshooting & Optimization | 18% | 4th |
Dev + Security = 58% — revisit these flagged questions first if time runs short.

## Messaging/Orchestration Decision Table
| Need | Service |
|---|---|
| One consumer per message, decoupled queue | SQS |
| Fanout — every subscriber gets a copy | SNS (→ SQS fanout) |
| Cross-source event routing/filtering | EventBridge |
| Multi-step orchestration, branching, retries, long-running | Step Functions |

## IAM Policy Evaluation Order
1. Explicit Deny anywhere (identity, resource, SCP, boundary) → **DENY wins, always**
2. Explicit Allow present, no deny → ALLOW
3. No explicit allow anywhere → implicit DENY (default)

## Deployment Strategy Rollback Comparison
| Strategy | Downtime | Rollback Speed | Extra Capacity Cost |
|---|---|---|---|
| All at once | Yes | Slow (redeploy) | None |
| Rolling | Reduced capacity | Medium | None |
| Rolling + additional batch | No | Medium | Temporary +1 batch |
| Immutable | No | Instant (terminate new ASG) | Double (temporary) |
| Lambda Canary (CodeDeploy) | No | Automatic via CloudWatch alarm | None (traffic-based) |

## Qualifier Word Decoder
| Phrase | Optimize For |
|---|---|
| BEST | Balanced tradeoff — cost, perf, security, complexity |
| MOST cost-effective | Lowest cost while still meeting ALL requirements |
| LEAST operational overhead | Managed/serverless over self-managed |
| MOST secure | Least privilege, encryption, isolation over convenience |

## Final Review Checklist (before submitting)
- [ ] All flagged questions revisited, starting with Development & Security
- [ ] Every multi-response question has the exact number of answers required
- [ ] Qualifier words re-checked (BEST vs cost-effective vs least overhead)
- [ ] No question left blank — guess before submitting
- [ ] Confirm IAM questions accounted for explicit-deny precedence
- [ ] Confirm deployment questions matched rollback speed/cost tradeoffs correctly
