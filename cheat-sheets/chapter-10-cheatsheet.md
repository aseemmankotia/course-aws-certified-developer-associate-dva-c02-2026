## Chapter 10 Cheat Sheet — CloudWatch & X-Ray (Domain: Troubleshooting & Optimization)

### CloudWatch Metrics
| Feature | Detail |
|---|---|
| Custom metric API | `PutMetricData` |
| Max dimensions/metric | 30 |
| Max metric entries per call | 1000 |
| Standard resolution | 60s (StorageResolution=60, default) |
| High resolution | 1s (StorageResolution=1) |
| Retention | 1-min data: 15 days → 5-min: 63 days → 1-hr: 455 days |
| Dimensions | Uniquely identify a metric stream; alarm dimensions must match exactly |

### CloudWatch Logs Insights — Core Commands
| Command | Purpose |
|---|---|
| `fields` | select fields to display |
| `filter` | filter log events (supports `like`, `=`, `in`) |
| `stats ... by` | aggregate (count, avg, sum, pct) grouped by field |
| `sort` | order results |
| `limit` | cap result rows |
| `parse` | extract fields from unstructured text |

Example: `fields @timestamp,@message | filter @message like /ERROR/ | stats count(*) by requestId | sort count(*) desc | limit 20`

### CloudWatch Alarms
| Concept | Detail |
|---|---|
| States | OK, ALARM, INSUFFICIENT_DATA |
| Composite alarms | Combine multiple alarms via AND/OR/NOT (e.g., `ALARM(a) AND ALARM(b)`) — reduces noisy notifications |
| Dashboards | Visualize metrics/alarms across regions/accounts; no native alerting, just display |

### X-Ray Core Concepts
| Concept | Detail |
|---|---|
| Segment | Records work done by one service for one request (trace root) |
| Subsegment | Records a downstream call within a segment (e.g., DynamoDB call) |
| Trace ID format | `1-{8hex epoch}-{24hex unique}` |
| Trace retention | 30 days |
| Default sampling | 1 req/sec reservoir + 5% additional |
| Daemon port | UDP 2000 |

### Annotations vs Metadata — THE key exam differentiator
| | Annotations | Metadata |
|---|---|---|
| Indexed? | YES — searchable/filterable | NO — not searchable |
| Value types | string, number, boolean (simple) | any JSON-serializable object |
| Use case | Filter/search traces (customer_id, status) | Extra debug context only |
| SDK call | `put_annotation(key, value)` | `put_metadata(key, value)` |

### Lambda + X-Ray Instrumentation
1. Enable **Active Tracing** on the Lambda function (no daemon needed — built-in extension handles it).
2. Grant the function IAM permissions: `xray:PutTraceSegments`, `xray:PutTelemetryRecords`.
3. Wrap SDK clients with X-Ray SDK capture (`aws_xray_sdk.core.patch_all()` in Python, or `AWSXRay.captureAWS()` in Node.js) to auto-generate subsegments for AWS SDK calls.
4. Use `put_annotation` for searchable fields (order_id, tenant), `put_metadata` for verbose debug payloads.

### Common Exam Traps
- Confusing annotations (searchable) with metadata (not searchable) — memorize: **A**nnotation = **A**ble to search.
- Assuming alarms auto-match partial dimension sets — they require exact dimension match.
- Forgetting metric data point aggregation/retention tiers (15/63/455 days) when asked about historical granularity.
- Thinking composite alarms evaluate raw metrics — they evaluate the **states of other alarms**.
