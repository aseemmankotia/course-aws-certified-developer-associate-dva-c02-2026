## Chapter 5: IAM, AWS STS, and Amazon Cognito: Authentication and Authorization — Practice Questions

**Q1.** A Lambda function in Account A (execution role: LambdaExecRole) needs to read objects from an S3 bucket owned by Account B. The bucket owner will not create a cross-account IAM role. What MUST be configured to allow this access?

A) Attach an identity-based policy allowing s3:GetObject to LambdaExecRole only
B) Create an IAM user in Account B and embed its access keys in the Lambda code
C) Enable S3 Transfer Acceleration on the bucket
D) Add a bucket policy on the Account B bucket granting s3:GetObject to LambdaExecRole's ARN, in addition to an identity-based policy on LambdaExecRole allowing s3:GetObject

<details>
<summary>Answer</summary>

**Correct: D**

Cross-account access to a resource requires BOTH sides to allow it: the caller's identity-based policy must allow the action, AND the resource owner must grant access via a resource-based (bucket) policy naming the caller's principal ARN. Identity-based policies alone cannot grant access to a resource in another account.

**Why the others are wrong:**
- An identity-based policy in Account A has no effect on a resource owned by Account B — the resource owner never granted access
- Hardcoding long-term access keys violates least privilege and is never the exam's 'best' answer when a role-based solution exists
- Transfer Acceleration is unrelated to authorization; it optimizes upload speed

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q2.** A mobile app authenticates users with Facebook Login and needs to let those users upload photos directly to an S3 bucket without embedding AWS credentials in the app. Which solution meets this requirement?

A) Create an Amazon Cognito User Pool and use its JWT access token to call S3 directly
B) Create one IAM user per mobile app user and rotate long-term access keys periodically
C) Call sts:GetFederationToken from the mobile app using long-term IAM user credentials embedded in the app
D) Use Amazon Cognito Identity Pool federated with Facebook as the identity provider, which calls STS AssumeRoleWithWebIdentity to vend temporary AWS credentials scoped by an IAM role

<details>
<summary>Answer</summary>

**Correct: D**

Cognito Identity Pools exist specifically to exchange a third-party or User Pool identity token for temporary AWS credentials via STS AssumeRoleWithWebIdentity, scoped by an IAM role with fine-grained S3 permissions — no AWS credentials ever ship in the app.

**Why the others are wrong:**
- A User Pool JWT authenticates the user but is NOT an AWS credential — it cannot sign S3 API calls
- Per-user IAM users do not scale and require long-term credential management, violating least privilege and exam best practice
- GetFederationToken requires long-term IAM user credentials as the caller, which should never be embedded in a distributed mobile app

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q3.** A company's SPA authenticates users through an Amazon Cognito User Pool and calls a REST API hosted on API Gateway. They want API Gateway to validate the JWT and reject invalid tokens without writing any custom authorization code. What should they configure?

A) A Cognito User Pool authorizer on the API Gateway method, referencing the User Pool ARN
B) A Lambda authorizer that manually decodes and validates the JWT signature
C) An IAM authorizer using SigV4-signed requests
D) A resource policy on the API Gateway REST API

<details>
<summary>Answer</summary>

**Correct: A**

API Gateway has a native Cognito User Pool authorizer type: you point it at the User Pool ARN and it automatically validates the incoming JWT's signature, expiration, and audience — zero custom code required.

**Why the others are wrong:**
- A Lambda authorizer works but requires custom code — the question asks for zero custom code, which the native Cognito authorizer provides
- IAM authorizers validate SigV4-signed requests using AWS credentials, not Cognito JWTs — wrong mechanism entirely
- Resource policies control network-level/account-level access (e.g., VPC endpoints, source IPs), not per-user JWT validation

</details>

---

**Q4.** Which AWS service should be used to provide a managed user directory with built-in sign-up, sign-in, MFA, and password policy enforcement for a customer-facing web application?

A) Amazon Cognito Identity Pool
B) AWS STS
C) Amazon Cognito User Pool
D) AWS IAM Identity Center

<details>
<summary>Answer</summary>

**Correct: C**

Cognito User Pools are purpose-built as a user directory: they handle sign-up, sign-in, MFA, password policies, and issue JWTs (ID, access, refresh tokens) upon successful authentication.

**Why the others are wrong:**
- Identity Pools do not store user credentials — they federate an existing identity into temporary AWS credentials
- STS only issues temporary credentials; it has no concept of a user directory or sign-up flow
- IAM Identity Center is designed for workforce (employee) SSO across AWS accounts and business apps, not customer-facing app authentication

</details>

---

**Q5.** An on-premises application already authenticates employees against a corporate Active Directory Federation Services (AD FS) SAML 2.0 identity provider. The application now needs temporary AWS credentials to call S3. Which AWS mechanism should be used?

A) Amazon Cognito Identity Pool with a SAML provider configured
B) IAM user access keys distributed to each employee
C) STS AssumeRoleWithWebIdentity
D) STS AssumeRoleWithSAML, using an IAM role whose trust policy trusts the SAML provider

<details>
<summary>Answer</summary>

**Correct: D**

AssumeRoleWithSAML is the direct STS API for exchanging a SAML assertion from a corporate IdP for temporary AWS credentials, using an IAM role whose trust policy specifies the SAML provider as a trusted principal.

**Why the others are wrong:**
- Cognito Identity Pools CAN support SAML but add unnecessary complexity when STS AssumeRoleWithSAML directly solves the requirement — exam favors the most direct native mechanism
- Distributing IAM user access keys is long-term credential sprawl and violates least privilege
- AssumeRoleWithWebIdentity is for OIDC/web identity providers (Facebook, Google, Cognito), not SAML 2.0 assertions

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q6.** A developer needs an STS AssumeRole session to remain valid for 8 hours (28,800 seconds) instead of the 1-hour default. What must be configured?

A) Nothing — all AssumeRole sessions default to 12 hours automatically
B) Set the role's MaxSessionDuration attribute to at least 28800 seconds, then pass DurationSeconds=28800 in the AssumeRole call
C) Set DurationSeconds=28800 in the AssumeRole call; MaxSessionDuration is irrelevant
D) Use sts:GetSessionToken instead, since it supports durations up to 8 hours by default

<details>
<summary>Answer</summary>

**Correct: B**

The DurationSeconds parameter passed to AssumeRole cannot exceed the role's configured MaxSessionDuration (default 1 hour, adjustable up to 43200 seconds/12 hours). Both must be set: raise MaxSessionDuration on the role, then request that duration.

**Why the others are wrong:**
- The default AssumeRole session is 3600 seconds (1 hour), not 12 hours
- Requesting a DurationSeconds value greater than the role's MaxSessionDuration causes the call to fail — the role setting is not irrelevant
- GetSessionToken is for MFA-protected IAM user sessions, not role assumption, and its default duration is 12 hours (43200s), not directly relevant here

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q7.** An IAM role's identity-based policy allows s3:* on a bucket. The bucket's resource-based policy contains an explicit Deny statement for that same role's ARN on s3:PutObject. What happens when the role attempts s3:PutObject on that bucket?

A) The request succeeds because identity-based Allow takes precedence over resource-based Deny
B) The request succeeds because s3:* in the identity policy is more specific than the bucket policy
C) AWS evaluates only the resource-based policy when both exist, ignoring the identity-based policy
D) The request is denied because an explicit Deny in any applicable policy always overrides any Allow

<details>
<summary>Answer</summary>

**Correct: D**

IAM policy evaluation logic is strict: an explicit Deny in ANY applicable policy — identity-based, resource-based, SCP, or permissions boundary — always wins, regardless of any Allow statements elsewhere.

**Why the others are wrong:**
- There is no precedence given to identity-based Allow; explicit Deny always overrides Allow
- Specificity of the action wildcard does not matter — Deny beats Allow unconditionally
- AWS evaluates ALL applicable policies together (identity-based AND resource-based); it does not ignore either one

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q8.** Which statement correctly distinguishes resource-based policies from identity-based policies in AWS IAM?

A) Resource-based policies are attached to a resource (e.g., an S3 bucket or SQS queue), include a Principal element, and can grant cross-account access without requiring the caller to assume a role
B) Resource-based policies never include a Principal element, while identity-based policies always do
C) Identity-based policies can grant access to any AWS account by default, while resource-based policies cannot
D) Resource-based policies can only be used with IAM users, not IAM roles

<details>
<summary>Answer</summary>

**Correct: A**

Resource-based policies are attached directly to the resource, must specify a Principal (who is being granted access), and are the mechanism that enables cross-account access to a resource without the accessing principal needing to assume a role in the resource's account.

**Why the others are wrong:**
- This reverses the fact — resource-based policies REQUIRE a Principal element; identity-based policies never include one
- Identity-based policies only grant access within the account of the attached principal unless a resource-based policy on the other side also grants access — they don't grant cross-account access by default
- Resource-based policies apply to resources, not to IAM users/roles directly — this statement misdescribes the concept entirely

</details>

---

