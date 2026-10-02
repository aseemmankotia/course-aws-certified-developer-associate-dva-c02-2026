## Chapter 10: Observability: Amazon CloudWatch and AWS X-Ray — Practice Questions

**Q1.** A development team wants to filter X-Ray traces by customer_id so they can quickly find all requests for a specific customer during an incident. Which instrumentation approach should they use?

A) Add customer_id as an X-Ray annotation using put_annotation()
B) Add customer_id as X-Ray metadata using put_metadata()
C) Log customer_id to CloudWatch Logs only and cross-reference manually
D) Add customer_id as a custom subsegment name

<details>
<summary>Answer</summary>

**Correct: A**

Annotations are indexed by X-Ray and are the only mechanism that supports filtering traces in the console or via GetTraceSummaries filter expressions, e.g. annotation.customer_id = "123".

**Why the others are wrong:**
- Metadata is stored with the trace but is NOT indexed, so it cannot be used in filter expressions — it's for viewing extra debug detail only.
- This works but loses the correlation between the trace and the log entry, and is far slower/manual than using X-Ray's built-in filtering.
- Subsegment names describe downstream calls, not arbitrary business data, and are not designed for ad-hoc key-value filtering.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q2.** An application publishes a custom CloudWatch metric with dimensions Environment=Prod and Service=Checkout. An engineer creates an alarm on the metric but specifies only the dimension Environment=Prod. What happens?

A) The alarm aggregates data from all dimension combinations that include Environment=Prod
B) The alarm fails to create because dimensions must match exactly
C) The alarm shows INSUFFICIENT_DATA forever because dimensions must match exactly what was published
D) CloudWatch automatically creates a second alarm covering the Service dimension

<details>
<summary>Answer</summary>

**Correct: C**

CloudWatch metrics with dimensions are treated as unique metrics per exact dimension combination. An alarm must reference the exact same set of dimension name/value pairs used in PutMetricData, or it will never receive matching data points, resulting in INSUFFICIENT_DATA.

**Why the others are wrong:**
- CloudWatch does not aggregate across partial dimension matches automatically — each dimension combination is a distinct metric stream.
- The alarm creation succeeds without validation against existing published data.
- CloudWatch never auto-generates alarms; this must be done explicitly.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q3.** A team wants an alarm to enter ALARM state only when BOTH the CPU utilization alarm AND the memory utilization alarm are simultaneously in ALARM state, to reduce noisy pages. What should they configure?

A) A CloudWatch composite alarm using ALARM(cpu_alarm) AND ALARM(memory_alarm)
B) Two separate CloudWatch Alarms with SNS actions sent to the same topic
C) A single metric math expression combining CPU and memory into one metric
D) An EventBridge rule that triggers only when two CloudWatch events occur within 1 minute

<details>
<summary>Answer</summary>

**Correct: A**

Composite alarms combine the states of multiple existing alarms using AND, OR, and NOT logic in a rule expression, letting you suppress alerts unless multiple underlying conditions are true simultaneously.

**Why the others are wrong:**
- Two independent alarms would each fire and notify independently — there's no AND logic between them.
- Metric math combines metric values mathematically, not alarm states, and doesn't produce the desired AND-of-alarms behavior directly.
- EventBridge can react to alarm state-change events but doesn't natively provide AND logic across two alarms without significant custom logic — composite alarms are the purpose-built feature.

</details>

---

**Q4.** You need to run an ad-hoc query against a CloudWatch Logs group to count the number of ERROR-level log entries grouped by requestId over the last hour. Which tool should you use?

A) CloudWatch Logs Insights with a stats query
B) CloudWatch Metric Filters
C) AWS X-Ray filter expressions
D) CloudWatch Contributor Insights rules only

<details>
<summary>Answer</summary>

**Correct: A**

CloudWatch Logs Insights provides a purpose-built query language (fields, filter, stats ... by, sort, limit) for ad-hoc analysis of log data, e.g. `filter @message like /ERROR/ | stats count(*) by requestId`.

**Why the others are wrong:**
- Metric filters convert log patterns into CloudWatch metrics for alarming over time — they aren't designed for one-off ad-hoc grouped queries.
- X-Ray filter expressions operate on trace annotations/metadata, not raw CloudWatch Logs content.
- Contributor Insights surfaces top-N contributors from logs/metrics but doesn't provide the same flexible ad-hoc query syntax as Logs Insights.

</details>

---

**Q5.** Which statement correctly differentiates X-Ray annotations from metadata?

A) Annotations support complex nested objects; metadata only supports strings
B) Metadata is indexed and filterable; annotations are for extra debug context only
C) Annotations are indexed and filterable; metadata is not indexed and used only for extra debug context
D) Both annotations and metadata are indexed but metadata has a larger size limit

<details>
<summary>Answer</summary>

**Correct: C**

Annotations are simple key-value pairs (string, number, boolean) that X-Ray indexes so you can search/filter traces on them. Metadata can hold richer objects but is never indexed — it's only visible when you open a specific trace.

**Why the others are wrong:**
- It's the reverse — metadata supports complex/nested objects, not annotations.
- This reverses the correct roles of annotations and metadata.
- Only annotations are indexed; metadata is explicitly excluded from indexing regardless of size.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q6.** A Lambda function calls DynamoDB and an external HTTP API. Response times are inconsistent, and the team needs to see exactly which downstream call is slow for each invocation. What should they implement?

A) Add more CloudWatch custom metrics for average Lambda duration
B) Enable Lambda active tracing and instrument the code with the X-Ray SDK to create subsegments for each downstream call
C) Increase Lambda memory allocation to reduce cold starts
D) Create a CloudWatch Logs Insights query that parses log timestamps to estimate latency

<details>
<summary>Answer</summary>

**Correct: B**

X-Ray subsegments break a single trace segment into the individual downstream calls (DynamoDB, HTTP API), showing exact latency per call. Enabling active tracing on the Lambda function plus SDK instrumentation is the standard exam-tested pattern for distributed tracing.

**Why the others are wrong:**
- Custom metrics show aggregate duration trends but not per-invocation breakdown of which downstream call was slow.
- Memory changes may help cold start/compute time but don't diagnose which downstream dependency is the bottleneck.
- Manually parsing logs for latency is error-prone and doesn't give the request-level dependency map that X-Ray provides natively.

</details>

---

**Q7.** You need a custom CloudWatch metric to be recorded and displayed at 1-second granularity instead of the standard 1-minute granularity. What must you configure in the PutMetricData call?

A) Set the Unit parameter to 'Seconds'
B) Set the metric's Dimensions to include a Resolution key
C) Set StorageResolution to 1
D) Enable detailed monitoring on the metric namespace

<details>
<summary>Answer</summary>

**Correct: C**

Setting StorageResolution=1 in the PutMetricData API call creates a high-resolution custom metric with 1-second granularity (default is StorageResolution=60, standard resolution).

**Why the others are wrong:**
- Unit describes the measurement unit (Count, Seconds, Bytes) and has no effect on data point granularity.
- Dimensions define what the metric measures/categorizes; there's no built-in 'Resolution' dimension key.
- 'Detailed monitoring' is an EC2/RDS concept for 1-minute vs 5-minute default metrics, not the mechanism for high-resolution custom metrics.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q8.** A company publishes a custom metric with 32 dimension key-value pairs in a single PutMetricData call. What will happen?

A) The call succeeds and CloudWatch stores all 32 dimensions
B) The call fails validation because the maximum is 30 dimensions per metric
C) CloudWatch automatically drops the extra dimensions and stores the first 30
D) The call succeeds but only the last 30 dimensions alphabetically are indexed

<details>
<summary>Answer</summary>

**Correct: B**

CloudWatch enforces a hard limit of 30 dimensions per metric; a PutMetricData call exceeding this limit returns a validation error.

**Why the others are wrong:**
- There is no support for storing more than 30 dimensions per metric — this exceeds the documented limit.
- CloudWatch does not silently truncate dimensions; it rejects the request instead.
- There is no alphabetical truncation behavior — the API call simply fails.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

