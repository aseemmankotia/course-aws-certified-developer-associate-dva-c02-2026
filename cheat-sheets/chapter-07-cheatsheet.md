## Chapter 7 Cheat Sheet: CI/CD with AWS Developer Tools

### CodePipeline Stage Model
| Stage | Purpose | Artifact Flow |
|---|---|---|
| Source | Pull code (Git provider, S3, ECR) | Produces output artifact → S3 |
| Build | Compile/test via CodeBuild | Consumes source artifact, produces build artifact → S3 |
| Test | Optional automated tests | Consumes build artifact |
| Deploy | Push to target (CodeDeploy, ECS, S3, CFN) | Consumes deploy-ready artifact |

Default artifact store = **S3 bucket**. Each stage's output is the next stage's input.

### CodeBuild buildspec.yml Phases
| Phase | Purpose |
|---|---|
| install | Install dependencies, set runtime-versions |
| pre_build | Pre-build tasks (e.g., ECR login, npm install extras) |
| build | Main build/compile/test commands |
| post_build | Cleanup, notifications, packaging prep |

**Not a phase:** top-level `artifacts:` key defines what gets uploaded (files, base-directory, discard-paths). No artifacts key = nothing passed downstream.

### CodeDeploy AppSpec.yml — EC2/On-Premises Lifecycle
```
ApplicationStop → DownloadBundle → BeforeInstall → Install →
AfterInstall → ApplicationStart → ValidateService
```
- `DownloadBundle` & `Install` = reserved for CodeDeploy agent (don't script).
- `hooks:` section maps lifecycle event → script path.
- File location required at **root of revision**.

### CodeDeploy AppSpec.yml — Lambda
```
BeforeAllowTraffic → AfterAllowTraffic
```
- Uses `Resources:` section (function name, alias, version weights) instead of `files:`.
- Deployment type: **Blue/Green only** — no in-place for Lambda.

### Deployment Configs (Lambda traffic shifting)
| Config | Behavior |
|---|---|
| LambdaAllAtOnce | 100% immediately |
| LambdaCanary10Percent5Minutes | 10% → wait 5 min → 100% |
| LambdaLinear10PercentEvery1Minute | +10% every minute |

### EC2 Deployment Types
- **In-place**: update existing instances (app stopped briefly).
- **Blue/Green**: new instance set deployed, traffic shifted, old set terminated after.

### CodeCommit Status (verify current docs)
- As of mid-2024: **no new customer onboarding**. Existing repos/customers unaffected.
- Exam scenario for *new* projects → expect a different Git provider as the "correct" answer.

### CodeArtifact
- Managed repo for **npm, pip, Maven, NuGet, generic, Swift** packages.
- Supports **upstream repositories** and **cross-account sharing** via resource policies.
- Integrates with CodeBuild via package manager config commands in `pre_build`.

### Common Exam Traps
- Confusing buildspec **phases** with the **artifacts** key (artifacts is separate, not a phase).
- Applying EC2-only hooks (BeforeInstall/AfterInstall/ValidateService) to a Lambda AppSpec — invalid.
- Assuming CodeDeploy auto-generates AppSpec.yml — it does not; missing file = failed deployment.
- Assuming CodeCommit is available for brand-new customers/accounts in 2026.
