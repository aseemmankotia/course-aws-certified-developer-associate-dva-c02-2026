## Chapter 6: Encryption and Secrets: AWS KMS, Secrets Manager, Parameter Store — Practice Questions

**Q1.** A company stores sensitive objects in S3 and must be able to prove via CloudTrail exactly which IAM principal used the encryption key for each object access, while also retaining control over key rotation and key policy. Which S3 encryption option BEST meets these requirements?

A) SSE-KMS with a customer managed key
B) SSE-S3 with AWS-owned keys
C) SSE-C with a customer-supplied key
D) Client-side encryption using a locally generated AES key

<details>
<summary>Answer</summary>

**Correct: A**

SSE-KMS with a customer managed CMK logs every Encrypt/Decrypt/GenerateDataKey call to CloudTrail with the calling principal, and the customer controls the key policy and rotation settings.

**Why the others are wrong:**
- SSE-S3 uses an AWS-owned key that is not visible in the customer's account and generates no per-use CloudTrail entries tied to a customer key.
- SSE-C requires the customer to supply the key on every request and AWS never stores it, so there is no KMS-based audit trail or managed rotation.
- Client-side encryption with a self-managed key adds operational overhead and still provides no CloudTrail audit trail unless KMS is used underneath.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q2.** A team needs credentials for an Amazon RDS MySQL database to rotate automatically every 30 days without writing or maintaining custom rotation code. Which solution meets this requirement with the least development effort?

A) Store the credentials as a Parameter Store SecureString and rotate manually every 30 days
B) Store the credentials in Secrets Manager and enable rotation using the built-in RDS rotation support
C) Store the credentials in Parameter Store Advanced tier with a parameter policy set to a 30-day TTL
D) Store the credentials in an S3 object encrypted with SSE-KMS and rotate via a scheduled EventBridge rule

<details>
<summary>Answer</summary>

**Correct: B**

Secrets Manager provides native rotation integration for RDS, Redshift, and DocumentDB — AWS supplies the rotation Lambda function, so no custom code is required.

**Why the others are wrong:**
- Parameter Store has no built-in rotation mechanism; manual rotation does not satisfy an automatic requirement.
- A parameter policy TTL only marks a parameter as expired/expiring — it does not rotate the credential value itself.
- Storing credentials in S3 with a scheduled EventBridge rule requires building the entire rotation Lambda from scratch, which is far more effort than native Secrets Manager rotation.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q3.** An organization needs to store 50,000 static application configuration values with no rotation requirement, while minimizing monthly cost. Which option satisfies the parameter count while being the most cost-effective?

A) Systems Manager Parameter Store Advanced tier
B) AWS Secrets Manager
C) Systems Manager Parameter Store Standard tier
D) Amazon DynamoDB table with SSE-KMS encryption

<details>
<summary>Answer</summary>

**Correct: A**

Parameter Store Standard tier is limited to 10,000 parameters per account per Region, so 50,000 parameters require Advanced tier, which still costs far less than Secrets Manager per-secret pricing.

**Why the others are wrong:**
- Secrets Manager charges $0.40 per secret per month, making 50,000 secrets extremely expensive for static config that needs no rotation.
- Parameter Store Standard tier caps out at 10,000 parameters, so it cannot hold 50,000 values.
- A DynamoDB table adds unnecessary operational complexity (schema, IAM, encryption setup) compared to the purpose-built Parameter Store service.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q4.** An application must use AWS KMS to encrypt a 500 MB file. Which approach correctly implements this using KMS?

A) Use envelope encryption: call GenerateDataKey, encrypt the file locally with the returned plaintext data key, then discard the plaintext key
B) Call the KMS Encrypt API directly on the 500 MB file
C) Increase the KMS customer managed key size to support larger payloads
D) Split the file into 4 KB chunks and call Encrypt on each chunk individually

<details>
<summary>Answer</summary>

**Correct: A**

KMS symmetric CMK Encrypt/Decrypt calls are limited to 4 KB of data, so envelope encryption is required: GenerateDataKey returns a plaintext data key and an encrypted copy; the plaintext key encrypts the large payload locally and is then discarded.

**Why the others are wrong:**
- The Encrypt API rejects payloads larger than 4 KB for symmetric CMKs, so this call would fail.
- KMS key size cannot be modified to support larger direct-encrypt payloads; the limit is fixed by the service.
- Chunking and calling Encrypt repeatedly is inefficient, costly at scale, and is not how AWS recommends handling large objects — envelope encryption is the documented pattern.

</details>

---

**Q5.** Which KMS key type allows a customer to define a custom key policy, enable or disable the key, schedule deletion, and choose whether automatic annual rotation is enabled?

A) AWS owned key
B) AWS managed key
C) S3 Bucket Key
D) Customer managed key

<details>
<summary>Answer</summary>

**Correct: D**

Customer managed keys (CMKs) are created and fully controlled by the customer, including key policy, enable/disable state, deletion scheduling, and optional automatic rotation.

**Why the others are wrong:**
- AWS owned keys are not visible in the customer's account at all and offer zero customer control.
- AWS managed keys (aws/service) rotate automatically every year with no option to disable rotation, and their policy cannot be customized.
- An S3 Bucket Key is not a KMS key type — it is a caching mechanism that reduces KMS API calls for SSE-KMS.

</details>

---

**Q6.** A bucket receives millions of PUT requests for small objects using SSE-KMS, and the team wants to reduce the resulting KMS API cost without switching encryption types. What should they configure?

A) Switch every object to SSE-S3
B) Enable S3 Bucket Keys on the bucket
C) Switch to SSE-C for all objects
D) Reduce the number of objects stored in the bucket

<details>
<summary>Answer</summary>

**Correct: B**

S3 Bucket Keys generate a time-limited bucket-level data key that is reused for many object encryptions, drastically reducing the number of calls made to KMS and lowering cost while still using SSE-KMS.

**Why the others are wrong:**
- Switching to SSE-S3 removes the audit trail and customer key control, changing the encryption type rather than optimizing cost, which the question says to avoid.
- SSE-C shifts key management burden entirely to the client and does not use KMS at all, again changing the encryption approach.
- Reducing object count is not a practical or intended solution to a per-request KMS billing problem.

</details>

---

**Q7.** A team creates a Parameter Store SecureString parameter and must ensure it is encrypted with a specific customer managed KMS key instead of the default AWS managed key. Which action accomplishes this?

A) Specify the --key-id option with the CMK's key ID or alias when creating the parameter via put-parameter
B) Enable Parameter Store Advanced tier, which encrypts all SecureStrings with a customer managed key automatically
C) Enable automatic KMS key rotation on the default alias/aws/ssm key
D) Move the value to Secrets Manager, since Parameter Store cannot use customer managed keys

<details>
<summary>Answer</summary>

**Correct: A**

The put-parameter API accepts a --key-id parameter to specify the customer managed CMK (by ID or alias) used to encrypt the SecureString value; without it, the default alias/aws/ssm key is used.

**Why the others are wrong:**
- The Advanced tier only affects size/count limits and parameter policies, not which KMS key is used by default.
- Rotating the default AWS managed key does not change which key was used — it remains the AWS managed key, not a customer managed key.
- Parameter Store fully supports customer managed KMS keys for SecureString parameters; moving to Secrets Manager is unnecessary.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q8.** What is the primary difference between S3 SSE-S3 and SSE-C encryption?

A) SSE-S3 uses an AWS-owned key fully managed by AWS, while SSE-C requires the customer to supply and manage the encryption key with every request
B) SSE-S3 bills per KMS request while SSE-C is always free
C) SSE-C provides a CloudTrail audit trail while SSE-S3 does not
D) SSE-S3 requires HTTPS for uploads while SSE-C does not

<details>
<summary>Answer</summary>

**Correct: A**

SSE-S3 uses an AWS-owned key that AWS manages entirely with no customer involvement, whereas SSE-C requires the customer to provide the actual encryption key with each PUT/GET request; AWS never stores that key.

**Why the others are wrong:**
- SSE-S3 does not use KMS at all, so there is no per-request KMS billing; SSE-C also has no KMS charge since it doesn't use KMS.
- Neither SSE-S3 nor SSE-C provides a CloudTrail-based key-usage audit trail — only SSE-KMS does, because it goes through KMS.
- SSE-C actually requires HTTPS (TLS) because the key is transmitted in the request headers; SSE-S3 has no such customer-key transmission requirement.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

