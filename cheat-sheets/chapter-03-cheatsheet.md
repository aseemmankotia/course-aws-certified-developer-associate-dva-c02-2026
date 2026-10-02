## DynamoDB Quick Reference (DVA-C02)

### GSI vs LSI
| Feature | GSI | LSI |
|---|---|---|
| Partition key | Can differ from base table | Must match base table |
| Sort key | Optional, own choice | Alternate sort key required |
| Consistency | Eventually consistent only | Strong or eventual |
| Throughput | Own dedicated RCU/WCU (or on-demand) | Shares base table throughput |
| Creation time | Anytime | Only at table creation |
| Max per table | 20 (default quota) | 5 (fixed) |

### Capacity Modes
| Mode | Best for | Billing |
|---|---|---|
| Provisioned + Auto Scaling | Predictable/steady traffic | Pay for provisioned RCU/WCU |
| On-Demand | Spiky/unpredictable traffic | Pay per request |

### Key Limits
- Max item size: **400 KB**
- Max Query/Scan response: **1 MB** per call → use `LastEvaluatedKey` to paginate
- Streams retention: **24 hours**
- TransactWriteItems/TransactGetItems: **100 items**, **4 MB** max
- DAX default TTL: **5 minutes** (item & query cache)
- 1 RCU = 1 strongly consistent read/sec (≤4KB) or 2 eventually consistent reads/sec
- 1 WCU = 1 write/sec (≤1KB)

### Partition Key Design
- Choose high-cardinality attributes for even distribution.
- Avoid low-cardinality keys (e.g., status flags) as sole partition key → hot partitions.
- Fix hot partitions: add random/calculated suffix (write sharding), or use composite key.

### DynamoDB Streams
- Captures item-level changes (INSERT/MODIFY/REMOVE).
- `StreamViewType`: `KEYS_ONLY`, `NEW_IMAGE`, `OLD_IMAGE`, `NEW_AND_OLD_IMAGES`.
- Integrates with Lambda via event source mapping — near real-time CDC.

### DAX
- Sits in front of DynamoDB, API-compatible with DynamoDB SDK.
- Microsecond read latency; write-through cache.
- Use when: read-heavy, latency-sensitive, tolerant of brief staleness.

### Conditional Writes & Transactions
- `ConditionExpression` on Put/Update/Delete — prevents overwrites, enables optimistic locking.
- `TransactWriteItems` — ACID guarantees across multiple items/tables (all-or-nothing).
- `TransactGetItems` — atomic multi-item reads.

### Query vs Scan
| | Query | Scan |
|---|---|---|
| Access pattern | Uses partition key (+ sort key/index) | Reads entire table/index |
| Cost | Lower — reads only matches | Higher — reads all items |
| Use case | Known key access pattern | Rare/ad-hoc analysis (avoid in hot paths) |

### Common Exam Traps
- Adding an LSI to an existing table → **not possible**, must recreate table.
- Assuming GSI gives strong consistency → **false**, always eventually consistent.
- Assuming Scan is fine for production hot paths → causes throttling/cost issues.
- Forgetting on-demand mode still has per-partition throughput ceilings (not unlimited).
