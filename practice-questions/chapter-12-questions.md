## Chapter 12: Full Practice Exam Simulation and Test-Taking Strategy — Practice Questions

**Q1.** A company processes orders through a Lambda function. Multiple downstream systems (billing, inventory, shipping) must each receive a copy of every order event, and the team must be able to add new subscribers without changing the publisher's code. Which solution BEST meets these requirements?

A) SQS standard queue with three consumers polling the same queue
B) Step Functions state machine invoking three Lambda functions sequentially
C) EventBridge custom bus with a single rule targeting one Lambda function
D) SNS topic with three SQS queues subscribed (fanout pattern)

<details>
<summary>Answer</summary>

**Correct: D**

SNS fanout to multiple SQS queues delivers an independent copy of each message to every subscriber, and new subscribers can be added by subscribing a new queue without touching the publisher.

**Why the others are wrong:**
- SQS uses competing-consumer semantics — only one consumer receives each message, not all three.
- Step Functions adds sequential coupling and latency; adding a subscriber requires editing the state machine definition.
- A single rule with one target does not fan out to multiple subscribers as described.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q2.** A developer's IAM identity has an attached policy that ALLOWS s3:GetObject on arn:aws:s3:::finance-bucket/*. The bucket policy explicitly DENIES s3:GetObject to every principal except role FinanceAdminRole. The developer's role is not FinanceAdminRole. What happens when the developer calls GetObject?

A) Access is allowed because the IAM identity policy explicitly allows it
B) Access is allowed because identity-based policies take precedence over resource-based policies
C) Access is denied only if the developer is in a different AWS account
D) Access is denied because an explicit Deny in any applicable policy always overrides an Allow

<details>
<summary>Answer</summary>

**Correct: D**

In AWS policy evaluation, an explicit Deny found anywhere in the applicable identity, resource, SCP, or permissions-boundary policies always overrides any Allow.

**Why the others are wrong:**
- The identity policy's Allow is irrelevant once an explicit Deny applies.
- There is no precedence rule favoring identity policies; explicit deny wins regardless of policy type.
- The account boundary is irrelevant here — the bucket policy denies same-account principals too.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q3.** A critical Lambda function behind API Gateway must shift traffic gradually to a new version, automatically roll back if CloudWatch alarms trigger, and require MINIMUM operational overhead to configure. Which approach BEST meets these requirements?

A) Manually update Lambda alias weights every few minutes while watching CloudWatch dashboards
B) Deploy a new Lambda function with a different name and update the API Gateway integration after manual testing
C) Use an Elastic Beanstalk blue/green environment swap for the Lambda function
D) Use AWS CodeDeploy with a Lambda deployment configuration (e.g., Canary10Percent5Minutes) and alarm-based automatic rollback

<details>
<summary>Answer</summary>

**Correct: D**

CodeDeploy natively supports canary/linear traffic shifting for Lambda and integrates with CloudWatch alarms for automatic rollback, requiring minimal manual configuration.

**Why the others are wrong:**
- Manual weight updates require constant human monitoring — high operational overhead.
- Renaming the function and cutting over is all-or-nothing, offering no gradual shift or automated rollback.
- Elastic Beanstalk does not manage standalone Lambda functions or their aliases; it's the wrong service for this use case.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q4.** A workflow needs to orchestrate multiple Lambda functions with branching logic, built-in retries, and visual monitoring of a business process that can run for up to a year. Which service is the BEST fit?

A) Amazon SQS
B) AWS Step Functions
C) Amazon SNS
D) Amazon EventBridge

<details>
<summary>Answer</summary>

**Correct: B**

Step Functions is purpose-built for stateful orchestration with branching, retries, and visual execution history, and supports long-running executions up to one year.

**Why the others are wrong:**
- SQS only queues messages; it has no orchestration or branching capability.
- SNS only publishes notifications and has no state management or workflow logic.
- EventBridge routes events via rules but does not manage a stateful, multi-step, visually trackable workflow.

</details>

---

**Q5.** A Lambda function needs to read from one specific DynamoDB table and write logs to CloudWatch, and nothing else. Which configuration follows LEAST privilege?

A) Attach the AWS managed policy AmazonDynamoDBFullAccess to the execution role
B) Attach AdministratorAccess to simplify permission management
C) Attach a customer-managed policy scoped to dynamodb:GetItem/Query on the specific table ARN plus AWSLambdaBasicExecutionRole
D) Grant access to all DynamoDB tables in the account via the Lambda function's resource-based policy

<details>
<summary>Answer</summary>

**Correct: C**

A custom policy scoped to only the required actions on the specific table ARN, combined with the basic execution role for CloudWatch Logs, grants exactly what is needed and nothing more.

**Why the others are wrong:**
- Full access grants write, delete, and access to every table — far beyond what's required.
- Administrator access is the direct opposite of least privilege.
- A Lambda resource-based policy controls who can invoke the function; it does not grant Lambda permissions to call DynamoDB, and granting all-table access is over-permissioned regardless.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q6.** When configuring a Lambda alias's routing-config to enable weighted traffic shifting for a canary deployment, what is the MAXIMUM number of function versions the alias can point to, including the primary version?

A) 1
B) 3
C) 2
D) Unlimited

<details>
<summary>Answer</summary>

**Correct: C**

A Lambda alias supports routing traffic to at most one additional version alongside the primary version — a maximum of 2 versions total.

**Why the others are wrong:**
- An alias pointing to only 1 version has no secondary weighted target, so no traffic shifting is possible.
- Aliases do not support a third version target.
- There is no unlimited routing-config support; the limit is fixed at 2.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q7.** An Elastic Beanstalk environment for a customer-facing app cannot tolerate any downtime during deployment, and the team needs to roll back instantly to the untouched previous fleet if health checks fail, accepting temporary double instance capacity as a tradeoff. Which deployment policy should they choose?

A) All at once
B) Immutable
C) Rolling
D) Rolling with additional batch

<details>
<summary>Answer</summary>

**Correct: B**

Immutable deployment launches a fully separate temporary Auto Scaling group with the new version, swaps traffic only after health checks pass, and rolls back instantly by terminating the new group — leaving the original fleet completely untouched, at the cost of double capacity.

**Why the others are wrong:**
- All at once causes downtime and offers no separate fleet for instant rollback.
- Rolling reduces capacity during deployment and does not guarantee zero downtime or a fully isolated rollback fleet.
- Rolling with additional batch avoids capacity reduction but still updates gradually in batches, without an instant full-fleet rollback.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q8.** An exam question asks: 'Which solution is the MOST cost-effective way to meet these requirements?' Several options technically satisfy the functional requirements. What does the qualifier 'MOST cost-effective' instruct you to do?

A) Choose the option with the fewest AWS services involved, regardless of price
B) Choose the option that meets all stated requirements at the lowest overall cost, even if it requires more configuration steps
C) Choose the option AWS documentation labels as a best practice
D) Choose the option that is easiest to implement, since time is money

<details>
<summary>Answer</summary>

**Correct: B**

The qualifier directs you to first confirm each option meets ALL stated requirements, then select the one with the lowest total cost — even if it takes more setup effort.

**Why the others are wrong:**
- Fewer services does not guarantee lower cost; some single-service solutions can be more expensive than a multi-service architecture.
- AWS-recommended best practices don't always equal the lowest-cost option in a given scenario.
- Ease of implementation is the signal for 'LEAST operational overhead,' a different qualifier, not cost.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

