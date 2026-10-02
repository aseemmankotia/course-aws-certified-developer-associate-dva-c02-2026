## Chapter 8: Infrastructure as Code: AWS CloudFormation, SAM, and CDK — Practice Questions

**Q1.** A team is about to update a production CloudFormation stack that contains an RDS database. They are concerned the update might cause the database to be replaced (destroyed and recreated), resulting in data loss. What should they do BEFORE applying the update?

A) Enable a stack policy to protect the resource
B) Create and review a change set before executing it
C) Enable Drift Detection on the stack
D) Convert the stack into a nested stack

<details>
<summary>Answer</summary>

**Correct: B**

Change sets show a preview of exactly which resources will be added, modified, or REPLACED before any change is actually applied, letting the team catch a destructive RDS replacement in advance.

**Why the others are wrong:**
- A stack policy restricts what CAN be updated, but it doesn't preview the impact of a specific update.
- Drift detection compares actual resource state to the template's expected state — it does not preview a future update.
- Nested stacks organize infrastructure into reusable modules; they don't preview changes.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q2.** A networking stack exports its VPC ID using an Output with an Export name. A separate application stack needs to reference that VPC ID. Which intrinsic function should the application stack use?

A) Fn::ImportValue
B) Fn::GetAtt
C) Fn::Sub
D) Ref

<details>
<summary>Answer</summary>

**Correct: A**

Fn::ImportValue retrieves values exported by another stack's Outputs section, enabling cross-stack references.

**Why the others are wrong:**
- Fn::GetAtt only retrieves attributes of resources defined within the SAME template.
- Fn::Sub performs string substitution using parameters/resources in the current template, not cross-stack values.
- Ref resolves a resource or parameter's default value within the SAME template only.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q3.** A developer writes a CloudFormation template containing an AWS::Serverless::Function resource, but the stack creation fails with an error that the resource type is invalid. What is most likely missing from the template?

A) Metadata: AWS::CloudFormation::Interface
B) A Mappings section
C) Transform: AWS::Serverless-2016-10-31
D) Capabilities: CAPABILITY_IAM in the CLI command

<details>
<summary>Answer</summary>

**Correct: C**

The Transform declaration is required to invoke the SAM macro, which expands simplified resource types like AWS::Serverless::Function into full CloudFormation resources.

**Why the others are wrong:**
- Metadata Interface only affects how parameters are grouped in the console UI, not resource resolution.
- Mappings provide static lookup tables and have no relation to SAM resource type resolution.
- CAPABILITY_IAM is required for IAM resource creation, but it wouldn't cause an 'invalid resource type' error.

</details>

---

**Q4.** A team wants to build a serverless API with Lambda, API Gateway, and DynamoDB using simplified resource declarations that automatically wire up permissions and event sources. Which tool BEST fits this requirement?

A) AWS SAM
B) Plain AWS CloudFormation
C) AWS Elastic Beanstalk
D) AWS OpsWorks

<details>
<summary>Answer</summary>

**Correct: A**

AWS SAM provides simplified resource types (AWS::Serverless::Function, Api, SimpleTable) specifically designed to reduce boilerplate for serverless applications.

**Why the others are wrong:**
- Plain CloudFormation requires verbose, manual definitions for IAM roles, permissions, and event source mappings.
- Elastic Beanstalk is for deploying traditional web apps on managed EC2/containers, not serverless-first architectures.
- OpsWorks is a configuration management service (Chef/Puppet) unrelated to serverless IaC.

</details>

---

**Q5.** A platform team wants to define infrastructure using TypeScript with loops, conditionals, and reusable classes, and have it synthesized into CloudFormation templates automatically. Which tool should they use?

A) AWS SAM CLI
B) AWS CDK
C) CloudFormation StackSets
D) AWS Config

<details>
<summary>Answer</summary>

**Correct: B**

AWS CDK allows writing infrastructure imperatively in TypeScript, Python, Java, etc., and running 'cdk synth' to generate CloudFormation templates.

**Why the others are wrong:**
- SAM CLI works with YAML/JSON templates using the SAM transform, not general-purpose imperative code.
- StackSets deploy the same CloudFormation stack across multiple accounts/regions — it doesn't provide a programming language interface.
- AWS Config is a resource compliance/auditing service, not an IaC authoring tool.

</details>

---

**Q6.** A developer needs to build a readable string that embeds a resource's logical ID value directly inline, such as an ARN pattern. Which intrinsic function is generally preferred for this over manually concatenating a list of strings?

A) Fn::Join
B) Fn::GetAtt
C) Fn::Sub
D) Fn::ImportValue

<details>
<summary>Answer</summary>

**Correct: C**

Fn::Sub lets you embed variables directly in a template string using ${VariableName} syntax, which is more readable than assembling a Fn::Join array.

**Why the others are wrong:**
- Fn::Join works but requires building an array of strings and a delimiter, which is harder to read for embedded variables.
- Fn::GetAtt only retrieves a single attribute value; it doesn't build a composite string.
- Fn::ImportValue only retrieves values exported by other stacks and doesn't build strings itself.

</details>

---

**Q7.** A template needs to reference a reusable child template stored in S3 to keep infrastructure modular. Which resource type is used to declare this in the parent template?

A) AWS::CloudFormation::StackSet
B) AWS::CloudFormation::Stack
C) AWS::Serverless::Application
D) AWS::CloudFormation::WaitCondition

<details>
<summary>Answer</summary>

**Correct: B**

AWS::CloudFormation::Stack defines a nested stack by pointing to a child template's TemplateURL, letting the parent stack manage it as a resource.

**Why the others are wrong:**
- StackSets deploy an entire stack across multiple accounts/regions — a different multi-account use case, not nested modularity within one stack.
- AWS::Serverless::Application is the SAM-specific equivalent for nested serverless apps but requires the SAM transform; it's not the generic CloudFormation answer.
- WaitCondition is used to pause stack creation until a signal is received; unrelated to nesting templates.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q8.** What is the primary purpose of a CloudFormation change set?

A) Preview the proposed changes to a stack's resources before they are applied
B) Automatically roll back a failed stack deployment
C) Detect drift between the stack's template and the live resource configuration
D) Enforce IAM permission boundaries during a deployment

<details>
<summary>Answer</summary>

**Correct: A**

A change set computes and displays what CloudFormation intends to do (Add/Modify/Remove, including replacements) without actually executing those changes until you explicitly run execute-change-set.

**Why the others are wrong:**
- Rollback behavior is controlled separately by stack failure/rollback settings, not change sets.
- Drift detection is a distinct feature comparing live resource state to the template — unrelated to previewing future updates.
- IAM permission boundaries are enforced by IAM policies/roles, not by the change set mechanism.

</details>

---

