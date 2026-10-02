## Chapter 9 Cheat Sheet: Deployment Strategies, Beanstalk, Containers

### Deployment Strategy Comparison
| Strategy | How it works | Downtime | Rollback speed | Cost |
|---|---|---|---|---|
| In-place (rolling) | Update instances directly in batches | Possible (reduced capacity) | Slow (redeploy old version) | Low |
| Blue/Green | New full environment stood up, traffic cut over | None | Instant (redirect traffic) | High (2x resources temporarily) |
| Canary | Small % traffic shifted first, then rest after bake time | None | Fast (halt shift, revert) | Medium |
| Rolling w/ additional batch | Extra batch launched before old batches removed | None | Slower than blue/green | Medium |

### CodeDeploy Deployment Configs
| Type | Applies to | Example presets |
|---|---|---|
| Canary | Lambda, ECS | Canary10Percent5Minutes, Canary10Percent10Minutes/15Min/30Min |
| Linear | Lambda, ECS | Linear10PercentEvery1Minute/2Min/3Min/10Min |
| AllAtOnce | Lambda, ECS, EC2/On-prem | Shifts 100% immediately |
| OneAtATime / HalfAtATime | EC2/On-premises only | In-place instance-by-instance |

### Lambda Versions vs Aliases
| Feature | Version | Alias |
|---|---|---|
| Mutability | Immutable snapshot | Mutable pointer |
| Purpose | Fixed code+config record | Named target (e.g., PROD) apps invoke |
| Weighted routing | N/A | Yes — max 2 versions per alias |
| Rollback | N/A (create new version) | Instant (repoint alias) |

### Elastic Beanstalk Deployment Policies
| Policy | Downtime | Capacity impact | Rollback |
|---|---|---|---|
| All at once | Yes | None | Manual redeploy |
| Rolling | No (reduced during deploy) | Temporary reduction | Slow |
| Rolling + additional batch | No | None (extra batch first) | Slower |
| Immutable | No | Temporary doubling | Fast (terminate new ASG) |
| Traffic splitting | No | Temporary doubling | Fast (canary-style) |

### ECS Launch Types
| Feature | Fargate | EC2 |
|---|---|---|
| Server management | None (serverless) | You manage instances |
| Billing | Per task vCPU/memory-second | Per EC2 instance-hour |
| GPU support | No | Yes |
| Custom storage/networking | Limited | Full control |
| Best for | Bursty/variable workloads | Steady-state, GPU, cost-optimized at scale |

### ECS/ECR Core Objects
- **Task Definition**: immutable blueprint (revisions), defines containers, CPU/memory, roles, network mode
- **Task**: running instance of a task definition
- **Service**: maintains desired task count, supports rolling update or CodeDeploy blue/green
- **ECR**: container image registry; supports lifecycle policies to expire old images

### Exam Gotchas
- Canary/Linear presets = **Lambda & ECS only**, NOT EC2/on-prem (those use OneAtATime/HalfAtATime/AllAtOnce)
- Lambda alias weighted routing = max **2 versions**
- Immutable EB deployment **doubles capacity temporarily** — costs more but safest rollback
- Fargate cannot use GPU instances — pick EC2 launch type for ML/GPU workloads on ECS
