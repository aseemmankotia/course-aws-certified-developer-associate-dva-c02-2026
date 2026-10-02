## Chapter 3: Amazon DynamoDB: Key Design, GSIs/LSIs, Streams, and DAX — Flashcards

| # | Front (Question) | Back (Answer) |
|---|-----------------|---------------|
| 1 | DynamoDB GSI vs LSI: which supports strongly consistent reads? | LSI only. GSIs support eventually consistent reads only. Exam tip: strong consistency requirement + alternate sort key = LSI (if table is new). |
| 2 | When can you create a DynamoDB Local Secondary Index? | Only at table creation time — cannot be added later. Max 5 LSIs per table. Exam trap: existing table + need alternate sort key = must use GSI instead. |
| 3 | Default max number of Global Secondary Indexes per DynamoDB table? | 20 GSIs per table (quota can be increased via Service Quotas). LSIs are capped at 5 and cannot be increased. |
| 4 | What causes a DynamoDB 'hot partition'? | Uneven access to one partition key value, since throughput is allocated per physical partition. Fix: add random/high-cardinality suffix to the partition key to spread load (write sharding). |
| 5 | DynamoDB max item size? | 400 KB per item. Exceeding this returns a ValidationException — not related to Query/Scan's 1MB response limit. |
| 6 | Max data returned by a single DynamoDB Query or Scan call? | 1 MB per response. If more data matches, response includes LastEvaluatedKey for pagination. |
| 7 | DynamoDB Streams retention period? | 24 hours. Stream records are then automatically removed. Use with Lambda event source mapping for near-real-time change data capture. |
| 8 | DAX default TTL for cached items? | 5 minutes (300 seconds) for both item cache and query cache. DAX is write-through and eventually consistent — not for strict consistency needs. |
| 9 | Max number of actions in a single DynamoDB TransactWriteItems/TransactGetItems call? | 100 unique items, up to 4 MB total data. Transactions are all-or-nothing (ACID) across one or more tables. |
| 10 | Provisioned vs On-Demand capacity mode: when to choose On-Demand? | Choose On-Demand for unpredictable/spiky traffic with no capacity planning desired — billed per request. Choose Provisioned+Auto Scaling for predictable, steady traffic to optimize cost. |
| 11 | How do you enforce optimistic locking / prevent overwriting concurrent updates in DynamoDB? | Use ConditionExpression on PutItem/UpdateItem (e.g., attribute_not_exists or version-number check). Fails with ConditionalCheckFailedException if condition isn't met. |
| 12 | Query vs Scan: which is cheaper and why? | Query is cheaper — it uses the partition key (and optional sort key/index) to read only matching items. Scan reads the entire table/index, consuming far more RCUs for large tables. |
