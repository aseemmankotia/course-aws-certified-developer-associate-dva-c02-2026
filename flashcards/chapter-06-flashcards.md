## Chapter 6: Encryption and Secrets: AWS KMS, Secrets Manager, Parameter Store — Flashcards

| # | Front (Question) | Back (Answer) |
|---|-----------------|---------------|
| 1 | KMS symmetric CMK direct Encrypt/Decrypt max plaintext size? | 4 KB (4096 bytes). Larger data requires envelope encryption via GenerateDataKey. |
| 2 | Envelope encryption flow with KMS | 1) GenerateDataKey → returns plaintext + encrypted data key. 2) Encrypt data locally with plaintext key. 3) Discard plaintext key, store encrypted data key with ciphertext. 4) Decrypt: KMS Decrypt on encrypted data key, then decrypt data locally. |
| 3 | AWS managed key vs customer managed key — rotation control | AWS managed key (aws/service): auto-rotates yearly, cannot be disabled or customized. Customer managed key: optional automatic yearly rotation or on-demand rotation, full policy control, can be disabled/deleted. |
| 4 | S3 SSE-S3 vs SSE-KMS vs SSE-C — key control | SSE-S3: AWS-owned key, free, no per-object audit trail. SSE-KMS: customer managed CMK, CloudTrail audit trail, per-request KMS charge. SSE-C: customer supplies key on every request over HTTPS; AWS never stores it. |
| 5 | How to lower SSE-KMS cost for many small S3 objects? | Enable S3 Bucket Keys — a bucket-level data key is cached and reused for a period, cutting the number of KMS API calls dramatically. |
| 6 | AWS Secrets Manager pricing | $0.40 per secret per month + $0.05 per 10,000 API calls. More expensive than Parameter Store, justified by built-in rotation. |
| 7 | Parameter Store Standard vs Advanced tier limits | Standard: free, up to 10,000 parameters/account/Region, 4 KB max value size. Advanced: $0.05 per advanced parameter/month, up to 100,000 parameters, 8 KB max value size, supports parameter policies (e.g., expiration TTL). |
| 8 | Which secrets service has native automatic rotation via Lambda? | AWS Secrets Manager. Built-in rotation Lambda for RDS, Redshift, and DocumentDB; custom Lambda needed for other secret types. Parameter Store has NO built-in rotation. |
| 9 | How is a Parameter Store SecureString encrypted? | With KMS — default alias/aws/ssm key unless a customer managed CMK is specified with --key-id. |
| 10 | CLI: create SecureString parameter with a custom CMK | aws ssm put-parameter --name /app/db/pass --value 'secret' --type SecureString --key-id alias/my-cmk |
| 11 | When to choose Secrets Manager over Parameter Store? | Choose Secrets Manager for: automatic rotation, native DB integration (RDS/Redshift/DocumentDB), cross-account resource policies. Choose Parameter Store for: static config, no rotation, lower cost. |
| 12 | Minimum TLS version for AWS API calls (encryption in transit)? | TLS 1.2 minimum is enforced for AWS API endpoints. This is separate from at-rest encryption handled by KMS. |
