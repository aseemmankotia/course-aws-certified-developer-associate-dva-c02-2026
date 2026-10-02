## Chapter 6: KMS, Secrets Manager, Parameter Store

### KMS Key Types
| Key Type | Managed By | Policy Control | Rotation |
|---|---|---|---|
| AWS owned | AWS | None (invisible to account) | Automatic, no control |
| AWS managed (aws/service) | AWS | View only | Auto yearly, cannot disable |
| Customer managed (CMK) | Customer | Full key policy control | Optional auto yearly or on-demand |

### Envelope Encryption (KMS)
1. `GenerateDataKey` → plaintext data key + encrypted data key
2. Encrypt data locally (AES-256) with plaintext key
3. Discard plaintext key; store encrypted data key with ciphertext
4. Decrypt: `Decrypt` on encrypted data key → decrypt data locally
- Symmetric CMK direct Encrypt/Decrypt limited to **4 KB** — envelope encryption required beyond that.

### S3 Server-Side Encryption Options
| Option | Key Owner | Audit Trail | Extra Cost | Client Manages Key |
|---|---|---|---|---|
| SSE-S3 | AWS-owned | No | No | No |
| SSE-KMS | Customer CMK | Yes (CloudTrail) | Yes (KMS API calls) | No |
| SSE-C | Customer | No | No | Yes, sent with every request over HTTPS |

- **S3 Bucket Keys**: reduce SSE-KMS cost by caching a bucket-level data key, cutting KMS request volume.
- Encryption in transit: TLS 1.2 minimum required for all AWS API calls.

### Secrets Manager vs Parameter Store
| Feature | Secrets Manager | Parameter Store (SSM) |
|---|---|---|
| Automatic rotation | Yes, native (RDS/Redshift/DocumentDB) or custom Lambda | No — build your own automation |
| Cost | $0.40/secret/month + $0.05/10k API calls | Standard: free; Advanced: $0.05/param/month |
| Max parameters/secrets | No hard count limit (billed per secret) | Standard: 10,000/account/Region; Advanced: 100,000 |
| Max value size | 64 KB | Standard: 4 KB; Advanced: 8 KB |
| Cross-account sharing | Resource policies supported | Limited |
| Best for | Rotating credentials, DB passwords | Static config, feature flags, low-cost secrets |

### Commands
```bash
# SecureString with custom CMK
aws ssm put-parameter --name /app/db/pass --value 'secret' \
  --type SecureString --key-id alias/my-cmk

# Retrieve with decryption
aws ssm get-parameter --name /app/db/pass --with-decryption

# Rotate a Secrets Manager secret
aws secretsmanager rotate-secret --secret-id mySecret
```

### Decision Rule
- Need rotation or native DB integration → **Secrets Manager**
- Static config, cost-sensitive, high volume → **Parameter Store** (Advanced if >10,000 params or >4KB values)
- Need audit trail on encryption key usage in S3 → **SSE-KMS** (add Bucket Keys to control cost)
- Fully customer-controlled key material in S3, no AWS storage of key → **SSE-C**
