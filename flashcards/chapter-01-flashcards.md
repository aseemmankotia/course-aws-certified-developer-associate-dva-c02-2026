## Chapter 1: AWS Lambda: Concurrency, Packaging, and Event Sources — Flashcards

| # | Front (Question) | Back (Answer) |
|---|-----------------|---------------|
| 1 | Lambda function timeout: max value and default? | Max 900 seconds (15 minutes). Default 3 seconds. Exam tip: exceeding timeout terminates the invocation and still bills for the time used. |
| 2 | Lambda memory allocation range? | 128 MB to 10,240 MB, in 1 MB increments. CPU power scales proportionally with memory. Exam tip: raising memory can reduce duration/cost for CPU-bound work. |
| 3 | .zip deployment package size limits? | 50 MB zipped for direct console/API upload; 250 MB unzipped total including all layers. Exam tip: exceeding this means using S3 upload or switching to a container image. |
| 4 | Container image max size for Lambda? | Up to 10 GB. Exam tip: use container images when dependencies exceed .zip limits or you need custom runtimes/libraries. |
| 5 | Max number of layers attachable to one Lambda function? | 5 layers. Combined with function code, the unzipped total must stay under 250 MB. Exam tip: layers = shared dependencies/code reuse across multiple functions. |
| 6 | Environment variables total size limit per function? | 4 KB combined for all variables. Exam tip: encrypted at rest by default with an AWS managed key; specify a customer managed CMK for extra control/auditability. |
| 7 | Reserved concurrency vs Provisioned concurrency? | Reserved = dedicated max/guaranteed concurrency ceiling, taken from the account's shared unreserved pool. Provisioned = pre-initialized warm environments to eliminate cold starts. Exam tip: reserved controls capacity limits; provisioned controls latency. |
| 8 | Which invocation model does S3 use to trigger Lambda? | Asynchronous (Event) invocation. Lambda retries automatically up to 2 times on failure. Exam tip: configure a DLQ or on-failure destination to capture events after retries are exhausted. |
| 9 | Which invocation model do SQS, Kinesis, and DynamoDB Streams use? | Poll-based Event Source Mapping — the Lambda service polls the source and invokes the function synchronously on your behalf. Exam tip: scaling here is tied to shards/partitions or queue depth, not just account concurrency. |
| 10 | How many automatic retries does Lambda perform for async invocations? | 2 retries (3 total attempts), with delays between them. Exam tip: configure a DLQ or Destinations on-failure target to capture events that exhaust retries. |
| 11 | How does Provisioned Concurrency mitigate cold starts? | It keeps a specified number of execution environments pre-initialized and ready to respond instantly, skipping the init phase. Exam tip: pairs with Application Auto Scaling for scheduled or metric-based scaling ahead of predictable spikes. |
| 12 | CLI command to set reserved concurrency on a function? | aws lambda put-function-concurrency --function-name myFunc --reserved-concurrent-executions 10. Exam tip: setting this to 0 effectively disables/throttles the function completely. |
