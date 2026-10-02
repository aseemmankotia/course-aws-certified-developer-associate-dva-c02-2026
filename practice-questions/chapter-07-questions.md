## Chapter 7: CI/CD Pipelines with AWS Developer Tools — Practice Questions

**Q1.** A company runs a pipeline with Source, Build, and Deploy stages in AWS CodePipeline. The Build stage uses AWS CodeBuild. Developers notice that files generated in the 'post_build' phase are not available to the Deploy stage. What is the MOST likely cause?

A) The pipeline's S3 artifact bucket has versioning disabled
B) CodeBuild phases must be run in a Docker container to produce artifacts
C) The Deploy stage must use the same IAM role as the Build stage to read artifacts
D) The buildspec.yml does not define an 'artifacts' section listing the output files

<details>
<summary>Answer</summary>

**Correct: D**

CodePipeline passes artifacts between stages via S3, but CodeBuild only uploads what is explicitly listed under the top-level 'artifacts' key in buildspec.yml (files, discard-paths, base-directory). Without this, no output artifact is produced regardless of what happens in post_build.

**Why the others are wrong:**
- Versioning affects rollback/history, not whether artifacts are passed between stages
- CodeBuild can run natively without Docker in some configurations, but this is unrelated to missing output files
- IAM roles control permissions, not whether the artifact zip contains the right files

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q2.** A team wants to run a validation Lambda function automatically before AWS CodeDeploy shifts traffic to the new version of a Lambda-based application during a canary deployment. Which AppSpec.yml hook should they use?

A) AfterInstall
B) ValidateService
C) BeforeAllowTraffic
D) BeforeInstall

<details>
<summary>Answer</summary>

**Correct: C**

For Lambda deployments, AppSpec.yml supports only two hooks: BeforeAllowTraffic (runs before traffic shifts to the new version) and AfterAllowTraffic (runs after). BeforeAllowTraffic is used to run pre-traffic validation Lambda functions.

**Why the others are wrong:**
- AfterInstall and BeforeInstall are EC2/on-premises hooks and do not apply to Lambda deployments
- ValidateService is also an EC2/on-premises-only hook, not available in Lambda AppSpec files

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q3.** A CodeBuild project needs to install project dependencies before compiling source code, and the compiled artifact must be uploaded once the build finishes successfully. In which buildspec.yml phases should these two actions occur, respectively?

A) install for dependencies; artifacts section for uploading (not a phase)
B) pre_build for dependencies; artifacts section for uploading (not a phase)
C) build for dependencies; post_build for uploading
D) install for dependencies; post_build phase for uploading

<details>
<summary>Answer</summary>

**Correct: A**

The 'install' phase is meant for installing dependencies and runtime versions. Uploading build output is not a phase at all — it is controlled by the top-level 'artifacts' section, which lists files CodeBuild packages and sends to the pipeline artifact store (typically S3).

**Why the others are wrong:**
- pre_build is typically used for tasks like logging into a container registry or running tests before the main build, not primary dependency installation
- The 'build' phase runs the actual compile/build commands, not dependency installation
- post_build is a phase for cleanup/notification tasks, not the mechanism that uploads artifacts — that is the separate 'artifacts' key

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q4.** A company wants a fully managed private repository to store and version npm and Maven packages that are consumed by multiple CodeBuild projects across different AWS accounts. Which service BEST meets this requirement?

A) AWS CodeArtifact
B) AWS CodeCommit
C) Amazon S3 with versioning enabled
D) AWS Systems Manager Parameter Store

<details>
<summary>Answer</summary>

**Correct: A**

AWS CodeArtifact is a managed artifact repository service purpose-built for storing and sharing software packages such as npm, pip, Maven, NuGet, and generic packages, with cross-account sharing support via resource policies.

**Why the others are wrong:**
- CodeCommit is a Git source repository for code, not a package/dependency repository, and is no longer onboarding new customers as of mid-2024
- S3 can store files but lacks native package-manager protocol support (npm/Maven metadata, versioning semantics) that CodeArtifact provides
- Parameter Store is for configuration/secrets, not binary package artifacts

</details>

---

**Q5.** A company currently uses AWS CodeCommit for source control and is starting a brand-new AWS account for a new project team in 2026. What should the team do regarding source control, based on current AWS guidance?

A) Continue using CodeCommit since it remains fully available for all new and existing accounts
B) Migrate all existing CodeCommit repositories immediately, as CodeCommit will be shut down within 90 days
C) Use AWS CodeArtifact as a drop-in replacement for Git source control
D) Use a third-party or self-hosted Git provider (or another supported source), since AWS CodeCommit is not onboarding new customers as of mid-2024

<details>
<summary>Answer</summary>

**Correct: D**

AWS announced in mid-2024 that CodeCommit is no longer accepting new customers, though existing customers can continue using it. New projects/accounts should choose a different supported source provider (e.g., a third-party Git service integrated with CodePipeline).

**Why the others are wrong:**
- New customers cannot start using CodeCommit — this contradicts current AWS guidance
- AWS has not announced an imminent shutdown deadline for existing CodeCommit customers
- CodeArtifact is a package repository, not a source control (Git) service, and cannot replace CodeCommit's function

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q6.** A CodeDeploy deployment to an EC2 Auto Scaling group fails during the 'ApplicationStart' hook. Which statement about the deployment lifecycle is TRUE?

A) ApplicationStart runs before BeforeInstall in the deployment lifecycle
B) ApplicationStart is a Lambda-only hook and should not appear in an EC2 AppSpec.yml file
C) If ApplicationStart fails, CodeDeploy automatically retries the entire deployment group indefinitely
D) ApplicationStart is used to start services or applications after files have been installed, and it runs after AfterInstall and before ValidateService

<details>
<summary>Answer</summary>

**Correct: D**

For EC2/on-premises deployments, the lifecycle order is: ApplicationStop, DownloadBundle, BeforeInstall, Install, AfterInstall, ApplicationStart, ValidateService. ApplicationStart is used to restart services after the new application revision is installed.

**Why the others are wrong:**
- ApplicationStart runs after BeforeInstall, not before it, in the defined lifecycle order
- ApplicationStart is a valid and commonly used EC2/on-premises hook, not Lambda-only
- CodeDeploy does not automatically retry indefinitely; a failed hook fails the deployment (or the specific instance, depending on failure policy) without infinite retries

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q7.** A developer configures a CodePipeline with a Source stage pulling from a Git repository and a Deploy stage targeting CodeDeploy. Builds are skipped entirely. What must be true for the Deploy stage to function correctly?

A) CodePipeline requires at least one Build stage before any Deploy stage can be added
B) CodeDeploy will automatically generate an AppSpec.yml if one is not provided in the source
C) The Source stage's output artifact must contain a valid AppSpec.yml along with the application files CodeDeploy needs
D) The Deploy stage must convert the source artifact into a Docker image before deployment

<details>
<summary>Answer</summary>

**Correct: C**

CodeDeploy requires an AppSpec.yml file describing deployment hooks and file mappings. If no Build stage transforms the source, the Source stage's artifact (passed via S3) must already include this AppSpec.yml alongside deployable application files.

**Why the others are wrong:**
- A Build stage is not mandatory — pipelines can go directly from Source to Deploy if no compilation/packaging is needed
- CodeDeploy does not auto-generate AppSpec.yml; deployments fail without one present in the artifact
- Docker image conversion is unrelated to standard EC2/on-premises or Lambda CodeDeploy deployments unless using ECS, which is a separate deployment type with its own AppSpec format

</details>

---

**Q8.** Which statement correctly describes how AWS CodePipeline passes artifacts between the Source, Build, and Deploy stages by default?

A) Artifacts are passed directly between compute instances used by each stage via a private VPC endpoint
B) Artifacts are streamed in-memory between stages using AWS Step Functions state transitions
C) CodePipeline requires an EFS file system to share artifacts between stages
D) Each stage stores its output artifact in an S3 bucket, and the next stage retrieves it as an input artifact from that same bucket

<details>
<summary>Answer</summary>

**Correct: D**

CodePipeline uses an S3 bucket (the pipeline's artifact store) to persist the output artifact of one stage; the next stage in the pipeline specifies that artifact as an input to consume it.

**Why the others are wrong:**
- There is no direct compute-to-compute artifact transfer; S3 is the intermediary storage mechanism
- CodePipeline is not built on Step Functions and does not stream artifacts in-memory
- EFS is not used by CodePipeline for artifact sharing between stages; S3 is the default and required mechanism

</details>

---

