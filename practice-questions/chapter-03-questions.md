## Chapter 3: Amazon DynamoDB: Key Design, GSIs/LSIs, Streams, and DAX — Practice Questions

**Q1.** A DynamoDB table stores IoT sensor readings with partition key = DeviceID and sort key = Timestamp. A small subset of devices sends 100x more data than others, causing throttling. Which change BEST resolves this?

A) Increase the table's provisioned RCUs and WCUs
B) Add a random suffix or high-cardinality attribute to create a composite partition key that distributes hot device writes across more partitions
C) Convert the sort key to a Local Secondary Index
D) Switch the table to Scan-based access instead of Query

<details>
<summary>Answer</summary>

**Correct: B**

Hot partitions occur when a single partition key value receives disproportionate traffic. Adding a randomized/high-cardinality suffix (write sharding) spreads writes across more partitions, since RCU/WCU capacity is allocated per partition, not per table.

**Why the others are wrong:**
- Raising overall table throughput doesn't fix uneven distribution across a single hot partition, which has its own internal throughput ceiling
- LSIs share the same partition key as the base table, so they don't help distribute writes across partitions
- Scan is less efficient than Query and unrelated to partition key hot-spotting

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q2.** A developer needs to query a DynamoDB table using an alternate sort key on an existing table that already contains 50GB of data and must support strongly consistent reads on the new key. Which index type should they use?

A) Global Secondary Index
B) DynamoDB Streams with a materialized view
C) Local Secondary Index
D) DAX with a custom query

<details>
<summary>Answer</summary>

**Correct: C**

LSIs support strongly consistent reads and use an alternate sort key with the same partition key. However, this scenario is a trap — LSIs can ONLY be created at table creation time, so on an existing table with data, an LSI cannot actually be added; the exam tests recognizing this constraint.

**Why the others are wrong:**
- GSIs support only eventually consistent reads, which fails the strong-consistency requirement
- Streams capture changes but don't provide alternate query patterns
- DAX caches reads but doesn't create new query access patterns

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q3.** An application requires microsecond-latency reads for a leaderboard read thousands of times per second, with occasional stale data being acceptable. Which AWS service should be added in front of DynamoDB?

A) Amazon DynamoDB Accelerator (DAX)
B) Amazon ElastiCache for Redis
C) Amazon CloudFront
D) AWS Global Accelerator

<details>
<summary>Answer</summary>

**Correct: A**

DAX is a DynamoDB-native, write-through in-memory cache providing microsecond read latency and API compatibility with the DynamoDB SDK, with a default 5-minute TTL for cached items.

**Why the others are wrong:**
- ElastiCache requires custom application logic to manage cache population/invalidation for DynamoDB, whereas DAX is purpose-built and API-compatible
- CloudFront caches HTTP content at edge locations, not DynamoDB API calls
- Global Accelerator improves network routing, not data-layer read latency

</details>

---

**Q4.** A workload has completely unpredictable, spiky traffic ranging from 0 to 40,000 requests per second within minutes, and the team wants to avoid capacity planning entirely. Which DynamoDB capacity mode should they choose?

A) Provisioned capacity with auto scaling enabled
B) Provisioned capacity with a high fixed RCU/WCU baseline
C) On-demand capacity mode
D) Reserved capacity mode

<details>
<summary>Answer</summary>

**Correct: C**

On-demand mode instantly accommodates traffic spikes without capacity planning, charging per request, and is ideal for unpredictable or highly variable workloads.

**Why the others are wrong:**
- Auto scaling reacts to CloudWatch alarms with a delay and can still throttle during sudden extreme spikes
- A fixed high baseline wastes money during low-traffic periods and still risks throttling on unexpected surges
- There is no 'reserved capacity mode' for DynamoDB throughput (Reserved Capacity applies to pricing commitments, not a distinct mode)

</details>

---

**Q5.** A banking application must transfer funds between two DynamoDB items atomically, ensuring both the debit and credit succeed or neither does. Which DynamoDB feature should be used?

A) TransactWriteItems
B) BatchWriteItem
C) ConditionExpression on a single PutItem call
D) DynamoDB Streams with a Lambda rollback function

<details>
<summary>Answer</summary>

**Correct: A**

TransactWriteItems provides ACID transactions across up to 100 actions and multiple items/tables, ensuring all-or-nothing execution — exactly what's needed for atomic fund transfers.

**Why the others are wrong:**
- BatchWriteItem has no atomicity guarantee; individual item writes can succeed or fail independently
- A single ConditionExpression only guards one item's write, not coordinated multi-item atomicity
- Streams+Lambda is asynchronous and eventually consistent, unsuitable for enforcing atomicity before commit

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q6.** A developer runs a Query returning 5MB of matching data. What will the API response actually contain?

A) All 5MB of results in a single response
B) An error because the item exceeds the 400KB limit
C) Up to 1MB of results plus a LastEvaluatedKey for pagination
D) The first 100 items regardless of size

<details>
<summary>Answer</summary>

**Correct: C**

DynamoDB Query and Scan operations return at most 1MB of data per call. If more matching data exists, the response includes a LastEvaluatedKey that the client uses to paginate with subsequent calls.

**Why the others are wrong:**
- DynamoDB never returns more than 1MB in a single Query/Scan response regardless of matched data size
- The 400KB limit applies to individual items, not aggregate query results
- Result truncation is based on the 1MB data size limit, not a fixed item count

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q7.** Which statement correctly differentiates a DynamoDB GSI from an LSI?

A) GSIs must be created at table creation time; LSIs can be added anytime
B) GSIs share the base table's provisioned throughput; LSIs have their own dedicated throughput
C) LSIs support up to 20 indexes per table; GSIs support up to 5
D) LSIs support strongly consistent reads and share the base table's partition key; GSIs use eventually consistent reads and can have a different partition key

<details>
<summary>Answer</summary>

**Correct: D**

LSIs use the same partition key as the base table with an alternate sort key and support strongly consistent reads; GSIs can use entirely different partition/sort keys, have their own provisioned throughput, and only support eventually consistent reads.

**Why the others are wrong:**
- This reverses the actual rule — LSIs, not GSIs, must be created at table creation and cannot be added later
- This reverses throughput ownership — GSIs have their own dedicated throughput settings; LSIs share the base table's provisioned throughput
- This reverses the index count limits — the default max is 20 GSIs and 5 LSIs per table

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q8.** A company wants to trigger a Lambda function whenever an item is updated in DynamoDB, sending only the new item image to downstream processing, and needs the trigger to fire within seconds. Which feature should they configure?

A) Amazon EventBridge scheduled rule polling the table every minute
B) DynamoDB Streams with StreamViewType set to NEW_IMAGE, mapped to a Lambda event source
C) S3 Event Notifications on table exports
D) CloudTrail data events for DynamoDB API calls

<details>
<summary>Answer</summary>

**Correct: B**

DynamoDB Streams captures a near-real-time, ordered sequence of item-level changes (24-hour retention) and can trigger Lambda directly via an event source mapping, with StreamViewType controlling which item data (e.g., NEW_IMAGE) is included.

**Why the others are wrong:**
- Polling on a schedule introduces latency and doesn't capture item-level change data efficiently
- S3 export notifications relate to bulk exports, not real-time change data capture
- CloudTrail logs API calls for auditing, not structured item change payloads for processing

</details>

---

