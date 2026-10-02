# Chapter 11 Lab: Root-Cause Diagnostics and Performance/Cost Optimization

**Estimated time:** 15 minutes

## Exercise

Diagnose a DynamoDB hot-partition scenario from CloudWatch per-partition consumed capacity metrics, then refactor a Lambda handler to initialize the AWS SDK client outside the handler function for connection reuse across warm invocations

## Key takeaway

Most troubleshooting exam questions are root-cause matching problems — pair the exact symptom (502 vs 504, DLQ vs destination, hot partition) with its precise cause.
