## Chapter 9: Deployment Strategies, Elastic Beanstalk, and Containers — Practice Questions

**Q1.** A team deploys a new Lambda function version and wants to shift 10% of production traffic to it, monitor a CloudWatch alarm for errors, and automatically roll back if the alarm triggers, before gradually increasing traffic. Which combination of AWS services and configuration BEST achieves this?

A) Create a new Lambda alias pointing 100% to the new version and manually update DNS weighting
B) Publish the new code directly over $LATEST and use an EventBridge rule to invoke a rollback Lambda if errors spike
C) Use AWS CodeDeploy with a Lambda deployment application using a Canary or Linear deployment preset and a CloudWatch alarm configured for automatic rollback
D) Create two separate Lambda functions and use an Application Load Balancer to split traffic between them

<details>
<summary>Answer</summary>

**Correct: C**

AWS CodeDeploy natively supports Lambda deployments using Canary (e.g., Canary10Percent5Minutes) or Linear (e.g., Linear10PercentEvery1Minute) presets, and can trigger automatic rollback based on a CloudWatch alarm tied to a Lambda alias with weighted routing.

**Why the others are wrong:**
- Manually shifting alias weights provides no automated monitoring or rollback — it requires manual intervention.
- $LATEST is not a stable production target and has no versioning/rollback guarantees; EventBridge-triggered rollback is a fragile custom solution vs. native CodeDeploy support.
- Lambda functions don't sit behind an ALB directly for traffic splitting the way this scenario requires; this is an over-engineered, non-standard approach.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q2.** A company runs short-lived, bursty batch-processing containers on Amazon ECS. They want zero server management and to pay only for the vCPU and memory consumed while tasks run. Which ECS launch type should they use?

A) EC2 launch type with Auto Scaling group
B) EC2 launch type with Spot Instances only
C) AWS Lambda with container image support instead of ECS
D) Fargate launch type

<details>
<summary>Answer</summary>

**Correct: D**

AWS Fargate is the serverless launch type for ECS — no EC2 instances to manage, and billing is per-task based on vCPU/memory consumed for the task's runtime, matching bursty, short-lived workloads.

**Why the others are wrong:**
- EC2 launch type requires managing and paying for instances continuously, even when idle — poor fit for bursty workloads.
- EC2 Spot still requires managing an underlying EC2 fleet and capacity, contradicting the 'zero server management' requirement.
- The scenario specifies ECS; switching to Lambda changes the orchestration model and isn't what's being asked (also has a 15-minute execution limit unsuitable for some batch jobs).

</details>

---

**Q3.** An application running on Amazon ECS requires GPU-based instances for machine learning inference and needs direct control over instance-level storage and networking configuration. Which ECS launch type is required?

A) EC2 launch type
B) Fargate launch type
C) AWS Batch with Fargate
D) Fargate Spot

<details>
<summary>Answer</summary>

**Correct: A**

The EC2 launch type for ECS lets you choose specific EC2 instance types (including GPU-enabled instances) and gives full control over the underlying host, including storage and networking — capabilities Fargate does not expose.

**Why the others are wrong:**
- Fargate does not support GPU-based tasks and abstracts away the underlying instance, removing the storage/networking control required.
- AWS Batch with Fargate still inherits Fargate's lack of GPU support.
- Fargate Spot is still Fargate under the hood and has the same GPU/instance-control limitations.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q4.** A developer needs an Elastic Beanstalk deployment policy that guarantees zero downtime, deploys the new version to an entirely separate, fully-scaled set of instances, and allows instant traffic cutover with easy rollback by simply terminating the new instances. Which deployment policy is this?

A) Immutable
B) All at once
C) Rolling with additional batch
D) Rolling

<details>
<summary>Answer</summary>

**Correct: A**

Immutable deployments launch a full new set of instances (temporarily doubling capacity) in a new Auto Scaling group, run health checks, then shift traffic. Rollback is simply terminating the new instance set, leaving the original environment untouched — the safest but most resource-intensive EB policy.

**Why the others are wrong:**
- All at once updates every existing instance simultaneously, causing downtime and no easy rollback.
- Rolling with additional batch launches one new batch before taking old batches down but updates instances in place across batches — it doesn't create a fully isolated parallel environment.
- Rolling updates existing instances in batches in place, causing reduced capacity and a slower, harder rollback than Immutable.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q5.** Which AWS CodeDeploy deployment configuration name correctly represents a linear deployment that shifts 10% of traffic every minute?

A) Canary10Percent1Minute
B) Linear10PercentEvery1Minute
C) RollingUpdate10PercentPerMinute
D) AllAtOnce10PercentEvery1Minute

<details>
<summary>Answer</summary>

**Correct: B**

CodeDeploy's built-in linear presets follow the naming pattern LinearX PercentEveryY Minute(s), e.g., Linear10PercentEvery1Minute, shifting a fixed percentage at fixed time intervals until 100%.

**Why the others are wrong:**
- Canary presets use a single upfront percentage shift then a wait, followed by the rest — named like Canary10Percent5Minutes, not per-minute incremental shifts.
- 'RollingUpdate' is not a CodeDeploy preset naming convention; rolling updates apply to EB/ECS services, not CodeDeploy Lambda/ECS deployment configs.
- AllAtOnce presets shift 100% of traffic immediately with no percentage/interval parameters.

</details>

---

**Q6.** How many Lambda function versions can a single alias route traffic to when using weighted alias routing for a canary release?

A) Up to 5 versions with custom percentages
B) Unlimited versions, each with an equal automatic split
C) Exactly 2 versions — a primary and a secondary with a specified weight
D) Only $LATEST and one published version

<details>
<summary>Answer</summary>

**Correct: C**

A Lambda alias's weighted routing configuration supports exactly two function versions at a time — a primary version and one additional version with an assigned percentage weight for canary-style traffic shifting.

**Why the others are wrong:**
- Weighted routing is not supported across 5 versions; this exceeds the actual limit of two.
- There is no unlimited or automatic even-split routing feature — weights must be explicitly assigned across exactly two versions.
- Aliases point to published (numbered) versions, not $LATEST, for production weighted routing — using $LATEST with an alias is not supported for weighted traffic shifting.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q7.** What is the key difference between a Lambda function version and a Lambda alias?

A) A version is mutable and can be updated in place, while an alias is an immutable snapshot of code and configuration
B) Versions are used only for testing, while aliases are used only in production environments
C) Aliases store the function code, while versions store only the configuration
D) A version is an immutable snapshot of code and configuration, while an alias is a mutable pointer that can be repointed to different versions

<details>
<summary>Answer</summary>

**Correct: D**

A published Lambda version is a fixed, immutable snapshot of code and configuration at a point in time. An alias is a mutable named pointer (like 'PROD' or 'DEV') that can be repointed to any version, including weighted routing across two versions.

**Why the others are wrong:**
- This reverses the actual behavior — versions are immutable, aliases are mutable.
- Both versions and aliases can be used in any environment; there's no such restriction — this is a fabricated distinction.
- Both versions and aliases reference the same underlying code and configuration; aliases don't store separate code — they simply point to a version.

</details>

---

**Q8.** A production application is deployed using a blue/green strategy. After the green environment receives full production traffic, monitoring reveals a critical bug. What is the primary advantage of blue/green over a rolling deployment in this situation?

A) Blue/green deployments use less compute resources during the deployment window
B) Blue/green deployments do not require health checks before promoting new code
C) Blue/green allows instant rollback by simply redirecting traffic back to the still-running, unmodified blue environment
D) Blue/green deployments automatically fix bugs using CodeDeploy's built-in error correction

<details>
<summary>Answer</summary>

**Correct: C**

In blue/green, the original (blue) environment remains fully intact and untouched while green receives traffic. Rollback is nearly instantaneous — just redirect traffic (via ALB, Route 53, or CodeDeploy) back to blue — unlike rolling deployments where instances have already been updated in place and reverting requires re-deploying the old version to each batch.

**Why the others are wrong:**
- Blue/green typically uses MORE resources temporarily since both environments run in parallel, the opposite of this claim.
- Health checks are a standard part of blue/green deployments before traffic cutover, not something they skip.
- There is no such 'error correction' feature — CodeDeploy detects failures via alarms/hooks but does not automatically fix code bugs.

</details>

---

