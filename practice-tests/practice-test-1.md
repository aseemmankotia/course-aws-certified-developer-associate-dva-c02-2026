# Practice Test 1: AWS Certified Developer Associate DVA-C02 Prep

- **Questions:** 45
- **Time limit:** 90 minutes
- **Passing score:** 70%%

> Answers are in `answer-key-1.md`. Track results in `score-tracker.md`.

---

**Q1.** A developer's Lambda function writes payment records to a legacy on-premises database that can only accept 10 concurrent connections. The Lambda function is invoked by an SQS queue that can receive thousands of messages per minute. What is the BEST way to prevent the function from overwhelming the downstream database?

A) Set the Lambda function's provisioned concurrency to 10
B) Set the Lambda function's reserved concurrency to 10
C) Request an increase to the account's overall concurrent execution limit to 10
D) Set the SQS queue's visibility timeout to 10 seconds

---

**Q2.** Multiple application instances update the same DynamoDB item concurrently. The team must ensure that an update only succeeds if the item has not been modified by another process since it was last read, implementing optimistic concurrency control. Which approach BEST achieves this?

A) Use an UpdateItem call with a ConditionExpression that checks the item's version attribute matches the value last read, incrementing it on success
B) Enable DynamoDB Streams and have a Lambda function detect and resolve conflicting writes after they occur
C) Enable DAX write-through caching in front of the table to serialize writes
D) Add a Global Secondary Index on the version attribute and query it before writing

---

**Q3.** A developer generates an S3 presigned URL using the AWS SDK, signing it with temporary credentials from an assumed IAM role that expire in 1 hour. The developer sets the URL's ExpiresIn parameter to 604800 seconds (7 days). What happens when a user attempts to use the URL 2 hours after it was generated?

A) The URL works because a presigned URL's expiration is completely independent of the credentials used to sign it
B) The URL fails because ExpiresIn cannot exceed 3600 seconds when using SigV4 signing
C) The URL fails because the underlying temporary credentials expired after 1 hour, even though the presigned URL's ExpiresIn value has not yet been reached
D) The URL works, but only for GET requests, not for PUT or POST operations

---

**Q4.** An e-commerce application must process orders in the exact sequence they were placed, but only orders for the SAME customer need to be processed in order relative to each other; orders from different customers can be processed in parallel. Which SQS configuration BEST meets this requirement?

A) A standard queue, using a client-generated MessageDeduplicationId for each order
B) A standard queue with the visibility timeout increased to reduce the chance of duplicate processing
C) A FIFO queue with the MessageGroupId set to the customer ID for each order
D) A FIFO queue with every message assigned the same MessageGroupId

---

**Q5.** A workflow must call three services in sequence, wait for a manual approval step that could take up to several days, automatically retry failed steps with custom backoff logic, and branch to different paths based on the approval outcome. Which AWS service is BEST suited to orchestrate this workflow?

A) AWS Step Functions state machine
B) Chain multiple Amazon EventBridge rules to trigger the next step of the workflow
C) An Amazon SNS topic with multiple Lambda function subscribers
D) An Amazon SQS queue with per-message delay seconds configured for each stage

---

**Q6.** An application's AWS SDK calls to DynamoDB are failing intermittently with ProvisionedThroughputExceededException during traffic spikes. The developer implements exponential backoff for retries. What is the PRIMARY reason to also add jitter (randomization) to the backoff delay?

A) It reduces the total number of retry attempts required before a request eventually succeeds
B) It increases the maximum retry timeout window allowed by the AWS SDK
C) It prevents many clients from retrying at synchronized intervals, which would otherwise create repeated bursts of simultaneous requests (a thundering herd)
D) It guarantees that the retried DynamoDB write is idempotent

---

**Q7.** A serverless order-processing application uses two separate Lambda functions that can both update the same DynamoDB item concurrently: one updates the shipping status and another updates the payment status. The team notices that occasionally one Lambda function's update silently overwrites the other's because both read the item and wrote back a new full copy at nearly the same time. Which solution prevents this lost-update problem?

A) Enable DynamoDB Streams and reconcile conflicting writes after they happen in a downstream consumer
B) Enable DynamoDB Accelerator (DAX) in front of the table to cache the item and serialize writes
C) Add a numeric 'version' attribute to the item and use UpdateItem with a ConditionExpression that the version equals the value last read, incrementing it on write
D) Increase the table's write capacity units so both updates can complete without conflict

---

**Q8.** An e-commerce system must process customer orders so that all orders from the same customer are handled in the exact order they were placed, and no order is processed more than once even if a message is delivered twice. Which SQS configuration BEST satisfies these requirements?

A) A standard SQS queue with a longer visibility timeout to reduce the chance of duplicate processing
B) A FIFO SQS queue with MessageGroupId set to the customer ID and deduplication enabled
C) A standard SQS queue combined with a separate DynamoDB table that manually tracks processing order
D) An SNS topic with multiple subscribers, one per customer, to fan out order events

---

**Q9.** A company ingests events from multiple third-party SaaS partners and needs to route them to different targets across several AWS accounts based on the event's source and detail-type, using built-in schema discovery and content-based filtering, without writing custom polling or filtering logic. Which service should the developer use?

A) Amazon SQS standard queue with one queue per partner
B) Amazon SNS topic using subscription filter policies
C) Amazon EventBridge with a custom event bus, rules, and multiple targets
D) AWS Step Functions Standard Workflow with a Choice state

---

**Q10.** A Lambda function is triggered by an SQS queue and writes to an RDS database that can only handle a small number of simultaneous connections. Which Lambda setting should be configured to guarantee that no more than 10 concurrent executions of this function run at any time?

A) Set the SQS queue's visibility timeout to 10 seconds
B) Configure provisioned concurrency of 10 on the Lambda function
C) Add a Lambda destination for asynchronous invocation failures
D) Configure reserved concurrency of 10 on the Lambda function

---

**Q11.** A web application must let browser clients upload files up to 5 GB directly to Amazon S3 without routing the data through the application's backend servers, while restricting uploads to only image content types and enforcing a maximum object size. Which approach BEST meets these requirements?

A) Generate an S3 presigned GET URL and instruct the client to PUT the file to that URL
B) Generate an S3 presigned POST URL with a policy specifying content-type and content-length-range conditions
C) Store long-term AWS credentials in the browser session so the client can call PutObject directly
D) Create an S3 Access Point and share its ARN with the client for direct uploads

---

**Q12.** When implementing retry logic in application code that calls AWS APIs which may return throttling errors, why should jitter be added to the exponential backoff delay calculation?

A) Jitter guarantees that throttling errors will never occur again after the first retry
B) Jitter reduces the total number of retry attempts an application needs to make
C) Jitter prevents multiple clients from retrying at exactly the same synchronized intervals, avoiding a thundering-herd effect on the service
D) Jitter increases the priority of the retried request in the service's internal queue

---

**Q13.** A payment-processing Lambda function is triggered by an SQS standard queue. Because SQS guarantees at-least-once delivery, the same payment message may invoke the Lambda function more than once, and duplicate charges must never be applied. Which approach BEST ensures idempotent payment processing?

A) Increase the SQS visibility timeout so the message cannot be redelivered while it is being processed
B) Switch to a FIFO queue with content-based deduplication and rely on it alone, with no application-level check
C) Before processing, perform a DynamoDB PutItem with a condition expression of attribute_not_exists(paymentId) to atomically record the payment ID, and only proceed with processing if the write succeeds
D) Set the Lambda function's reserved concurrency to 1 so only one invocation can run at a time

---

**Q14.** A development team is building an order fulfillment process that must invoke four Lambda functions in a specific sequence, branch to different paths based on the result of each step, wait for external callbacks that may take hours, and automatically retry failed steps with exponential backoff. The team also needs a visual history of each execution for troubleshooting. Which AWS service should be used to orchestrate this workflow?

A) Amazon EventBridge with a chain of rules routing events between the four Lambda functions
B) Amazon SQS with a single Lambda function that polls four separate queues in order
C) AWS Lambda Destinations, chaining each function's onSuccess destination to invoke the next function
D) AWS Step Functions using a Standard Workflow

---

**Q15.** A company must allow a partner AWS account to access resources in their account temporarily, ensuring the partner's role cannot be assumed by an unrelated third party who somehow learns the role ARN. Which approach BEST meets this requirement?

A) Create an IAM user in the company's account and share its long-term access key with the partner
B) Create an IAM role with a trust policy that requires a unique external ID, and have the partner call AWS STS AssumeRole with that external ID
C) Share the AWS account root user credentials with the partner
D) Attach an identity-based policy directly to the partner's IAM users granting cross-account S3 permissions

---

**Q16.** A mobile application must authenticate users with a username and password, and after login the app needs temporary AWS credentials to upload photos directly to an S3 bucket. Which combination of Amazon Cognito components should the developer implement?

A) A Cognito User Pool only, using the ID token directly to sign AWS API requests
B) A Cognito User Pool for authentication, federated into a Cognito Identity Pool to obtain temporary AWS credentials
C) A Cognito Identity Pool only, configured to allow unauthenticated guest access
D) A Cognito User Pool federated with a third-party SAML provider, without configuring any S3 access

---

**Q17.** A company requires that every object uploaded to a specific S3 bucket must be encrypted with AWS KMS (SSE-KMS), and any PutObject request that does not specify SSE-KMS encryption must be rejected outright. Which bucket policy statement satisfies this requirement?

A) Enable S3 default (bucket-level) encryption using SSE-S3 with no bucket policy changes
B) Require the Content-MD5 header to be present on all PutObject requests
C) Enable S3 bucket versioning so unencrypted objects create a new version instead of being denied
D) A Deny statement on s3:PutObject with a condition that s3:x-amz-server-side-encryption StringNotEquals "aws:kms"

---

**Q18.** An application's RDS database credentials must be automatically rotated every 30 days with zero application downtime and no manual intervention. Which AWS service and configuration BEST satisfies this requirement?

A) AWS Systems Manager Parameter Store SecureString parameters paired with a custom-built scheduled Lambda rotation function
B) An AWS KMS customer managed key with automatic annual key rotation enabled
C) Plaintext environment variables on the Lambda function, re-deployed manually every 30 days
D) AWS Secrets Manager with automatic rotation enabled using its built-in RDS rotation Lambda function

---

**Q19.** A developer's application calls the KMS GenerateDataKey API before encrypting a 2GB file locally and uploading it to S3. Which statement correctly describes what happens during this envelope encryption process?

A) KMS returns both a plaintext data key and a copy of that same data key encrypted under the CMK; the application uses the plaintext key to encrypt the file locally and stores the encrypted key alongside the ciphertext
B) KMS encrypts the entire 2GB file directly inside the KMS service and returns the resulting ciphertext to the application
C) The application must first upload the entire file to KMS so that KMS can generate a data key based on the file's contents
D) The plaintext data key returned by KMS is retained by KMS indefinitely so it can decrypt the file later without the application's involvement

---

**Q20.** A developer wants Amazon S3 to automatically invoke a Lambda function whenever a new object is uploaded to a bucket in the same AWS account. What must be configured to grant S3 permission to invoke the function?

A) Attach an identity-based IAM policy to the Lambda function's execution role granting s3:GetObject
B) Add a resource-based policy (Lambda permission) that allows the s3.amazonaws.com service principal to invoke the function, scoped to the specific bucket ARN
C) Create a dedicated IAM user for the S3 service and attach an AdministratorAccess managed policy to it
D) Configure a VPC endpoint policy that allows the S3 prefix list to invoke the Lambda function

---

**Q21.** A company has two AWS accounts: Account A (development) and Account B (production). Developers in Account A need temporary, auditable access to specific resources in Account B without sharing long-term credentials. Which approach BEST meets this requirement?

A) Create an IAM user in Account B and distribute its access keys to developers in Account A
B) Attach an S3 bucket policy in Account B granting Account A access, and rely on this for all resource access
C) Share the root user credentials of Account B with the Account A development team
D) Create an IAM role in Account B with a trust policy that trusts Account A, and have developers call sts:AssumeRole to obtain temporary credentials

---

**Q22.** A mobile application allows users to sign in with their Google or Facebook account and then needs to upload photos directly to an S3 bucket using temporary AWS credentials scoped to that user. Which AWS feature should the developer use to provide these temporary AWS credentials?

A) Create an IAM user for every mobile app user at sign-up
B) Amazon Cognito User Pools alone, using the ID token to call S3 directly
C) Manually implement sts:AssumeRoleWithWebIdentity calls in the mobile client for each provider
D) Amazon Cognito Identity Pools configured with Google and Facebook as identity providers

---

**Q23.** A Lambda function must encrypt a 50MB file using envelope encryption with a KMS customer master key (CMK) before storing it. Which sequence of steps correctly implements envelope encryption?

A) Call kms:GenerateDataKey to get a plaintext data key and its encrypted copy, use the plaintext key to encrypt the file locally, then store the encrypted data key alongside the ciphertext and discard the plaintext key
B) Call kms:Encrypt directly on the entire 50MB file using the CMK
C) Call kms:GenerateDataKey once, then store the plaintext data key in DynamoDB for reuse on future files
D) Call kms:Decrypt on the file to encrypt it using the CMK

---

**Q24.** A developer needs to enable server-side encryption on an S3 bucket where AWS fully manages the encryption keys using AES-256, with no additional per-request KMS charges and no need to configure key policies. Which S3 encryption option satisfies this requirement?

A) SSE-KMS (AWS Key Management Service managed keys)
B) SSE-S3 (Amazon S3-managed keys)
C) SSE-C (customer-provided keys)
D) Client-side encryption using a customer-managed KMS key

---

**Q25.** An application stores RDS database credentials that must be automatically rotated every 30 days using a native AWS-provided Lambda rotation template, without the developer writing a custom rotation scheduler. Which service should be used to store these credentials?

A) AWS Systems Manager Parameter Store using SecureString parameters
B) AWS Identity and Access Management (IAM)
C) AWS Secrets Manager
D) AWS Key Management Service (KMS)

---

**Q26.** Which statement correctly differentiates an IAM identity-based policy from a resource-based policy?

A) Identity-based policies always require an explicit Principal element, while resource-based policies never include one
B) Resource-based policies can only be attached to IAM roles and never to services like S3 or SNS
C) Resource-based policies cannot be used to grant cross-account access, unlike identity-based policies
D) Identity-based policies are attached to IAM users, groups, or roles, while resource-based policies are attached directly to a resource (such as an S3 bucket or SNS topic) and can specify a Principal to grant cross-account access

---

**Q27.** A developer configures AWS CodeDeploy to deploy a new Lambda function version. The deployment must shift 10% of traffic immediately, then the remaining 90% linearly every 10 minutes until 100% is reached, so that CloudWatch alarms have time to detect errors before full rollout. Which predefined CodeDeploy deployment configuration should be selected?

A) Canary10Percent5Minutes
B) Linear10PercentEvery10Minutes
C) AllAtOnce
D) Canary10Percent10Minutes

---

**Q28.** A CloudFormation template for a network stack exports a value named 'VPCId' using the Outputs section with an Export field. A separate application stack needs to reference this VPC ID when creating a subnet resource. Which intrinsic function should be used in the application stack to retrieve this value?

A) Fn::GetAtt
B) Fn::ImportValue
C) Fn::Sub
D) Fn::FindInMap

---

**Q29.** A serverless application defined with AWS SAM has five Lambda functions that all require the same 512 MB memory size, 10 second timeout, and identical environment variable for a table name. What is the MOST efficient way to apply these common settings across all functions without repeating them in each AWS::Serverless::Function resource?

A) Define the settings once in the Globals section of the SAM template
B) Create a nested stack containing only the shared configuration values
C) Use a CloudFormation mapping to store the shared property values
D) Add the settings as parameters that must be passed at each sam deploy

---

**Q30.** A team stores private npm packages in AWS CodeArtifact and wants CodeBuild to automatically fetch these dependencies during the build phase without hardcoding credentials in the buildspec.yml. What is the correct approach?

A) Store the CodeArtifact authorization token as a plaintext buildspec environment variable
B) Manually download the packages and upload them to the S3 source bucket before each build
C) Embed the CodeArtifact repository password directly into the package.json file
D) Grant the CodeBuild service role permission to call codeartifact:GetAuthorizationToken and run 'aws codeartifact login' in the buildspec install phase

---

**Q31.** A containerized application running on Amazon ECS with the Fargate launch type needs a blue/green deployment strategy that automatically shifts production traffic between the old and new task sets and can roll back automatically if CloudWatch alarms trigger. Which combination of services should the developer configure?

A) AWS CodeDeploy with an ECS compute platform, integrated with an Application Load Balancer and CloudWatch alarms
B) ECS rolling update deployment type with minimum healthy percent set to 100
C) AWS Elastic Beanstalk with immutable deployment policy for the ECS environment
D) Amazon ECR image tag rollback triggered manually via the AWS CLI

---

**Q32.** An Elastic Beanstalk environment running a critical production web application must ensure that during deployments, at least the full fleet of original, unmodified instances remains serving traffic until an entirely new set of instances passes health checks, with instant rollback by terminating new instances if health checks fail. Which Elastic Beanstalk deployment policy meets this requirement?

A) Immutable
B) Rolling with additional batch
C) All at once
D) Rolling

---

**Q33.** A developer is deploying a new version of an AWS Lambda function using AWS CodeDeploy with a canary deployment configuration. The company requires that a validation Lambda function run BEFORE any traffic is shifted to the new version, and the deployment must automatically roll back if the validation fails. Which AppSpec.yml configuration should the developer use?

A) Define a BeforeInstall hook that invokes the validation Lambda function, since it runs prior to any traffic shift
B) Define an AfterAllowTraffic hook that invokes the validation Lambda function, and configure CloudWatch alarms on the deployment group for automatic rollback
C) Skip AppSpec hooks entirely and manually shift traffic using a Lambda alias with weighted routing
D) Define a BeforeAllowTraffic hook that invokes the validation Lambda function, and configure CloudWatch alarms on the deployment group for automatic rollback

---

**Q34.** A team wants to preview the impact of updating an AWS CloudFormation stack before applying the changes, including seeing which resources will be modified, added, or replaced. Which action should they take?

A) Enable termination protection on the stack and then redeploy it
B) Create a change set for the stack and review it before executing
C) Attach a stack policy to the stack to preview the update
D) Run the update-stack CLI command with the --dry-run flag

---

**Q35.** A developer maintains a SAM template with 15 Lambda functions that all require the same memory size, timeout value, and runtime. To avoid repeating these properties in every AWS::Serverless::Function resource, which SAM template section should the developer use?

A) Globals
B) Mappings
C) Conditions
D) Parameters

---

**Q36.** A company runs a containerized application on Amazon ECS with the Fargate launch type and wants zero-downtime deployments with the ability to automatically roll back based on CloudWatch alarms while traffic is being shifted to the new task set. Which deployment configuration should they use?

A) Configure the ECS service to use the rolling update deployment type with minimum healthy percent set to 100
B) Use AWS Elastic Beanstalk to manage the ECS service with immutable deployments
C) Manually create a second ECS service and shift traffic using Route 53 weighted routing records
D) Configure the ECS service deployment controller as CODE_DEPLOY and set up an AWS CodeDeploy blue/green deployment with an Application Load Balancer and CloudWatch alarms

---

**Q37.** A company uses AWS CodePipeline to build and deploy a Java application. Developers want to store and share internal Maven and npm packages across multiple pipelines and AWS accounts without managing their own package repository infrastructure. Which AWS service should they use?

A) AWS CodeArtifact
B) AWS CodeCommit
C) Amazon ECR
D) Amazon S3 with versioning enabled

---

**Q38.** A developer instruments a Lambda function with the AWS X-Ray SDK and needs to filter traces in the X-Ray console by a custom field called order_status. Which SDK call should be used so this field is indexed and searchable?

A) Call segment.put_metadata() to attach the field
B) Add the field to the segment's HTTP block
C) Log the field to CloudWatch Logs and enable Logs Insights
D) Call segment.put_annotation() to attach the field

---

**Q39.** A DynamoDB table uses a partition key of order_date (e.g., '2026-06-01'). During flash sales, the table experiences heavy write throttling even though the table's aggregate provisioned throughput is far from being exhausted. What is the MOST likely root cause?

A) The table's provisioned RCUs are set too low
B) The low-cardinality partition key concentrates all of a day's writes onto a single partition, exceeding its per-partition limit
C) The table needs a global secondary index to spread the load
D) DynamoDB Accelerator (DAX) has not been enabled for the table
E) placeholder

---

**Q40.** An S3 PUT event asynchronously invokes a Lambda function that occasionally fails due to a downstream service outage. The team needs failed events automatically captured together with the invocation result (Lambda response payload, error type, and stack trace) for automated reprocessing, without adding any custom error-handling code to the function. What should the developer configure?

A) A Lambda on-failure destination pointing to an SQS queue or SNS topic
B) An SQS dead-letter queue attached to the function's event source mapping
C) Increase the function's reserved concurrency
D) Wrap the function code in a try/catch block that publishes to SNS on failure

---

**Q41.** A Lambda proxy integration behind API Gateway returns HTTP 502 'Bad Gateway' to clients, while CloudWatch Logs show the Lambda function executing successfully and returning a value with no errors logged. What is the MOST likely cause?

A) The Lambda function's timeout is configured lower than the API Gateway integration timeout
B) The API Gateway resource policy is denying the invocation
C) The Lambda function returned a response that does not match the required proxy integration format (missing statusCode, headers, or body)
D) The Lambda function's execution role lacks apigateway:POST permission

---

**Q42.** An application performs read-heavy GetItem and Query calls directly against a DynamoDB table. The team wants to add a caching layer that delivers microsecond read latency while requiring minimal changes to existing application code. Which solution BEST meets this requirement?

A) Amazon DynamoDB Accelerator (DAX)
B) Amazon ElastiCache for Redis using a cache-aside pattern implemented in application code
C) Amazon ElastiCache for Memcached using write-through caching
D) Enable DynamoDB auto scaling to reduce read latency

---

**Q43.** A developer needs to run a CloudWatch Logs Insights query against a Lambda function's log group to count ERROR-level log entries grouped into 5-minute time bins. Which query syntax accomplishes this?

A) SELECT COUNT(*) FROM logs WHERE level='ERROR' GROUP BY bin(5m)
B) filter @message like /ERROR/ | aggregate count(*) interval 5m
C) filter @message like /ERROR/ | stats count(*) by bin(5m)
D) grep ERROR logs.txt | wc -l | group 5m

---

**Q44.** A company stores user session data in a DynamoDB table using on-demand capacity mode. The partition key is 'session_type', which only has three distinct values: 'web', 'mobile', and 'api'. During peak traffic, the application starts receiving ProvisionedThroughputExceededException errors even though the table's total consumed capacity is far below the account-level service quota. Which change BEST resolves this issue?

A) Switch the table to provisioned capacity mode with auto scaling enabled
B) Change the partition key to a high-cardinality attribute such as session_id
C) Add a global secondary index on session_type to distribute the load across more partitions
D) Enable DynamoDB Accelerator (DAX) in front of the table

---

**Q45.** A developer instruments a Lambda function with the AWS X-Ray SDK. The developer needs to filter traces in the X-Ray console using a filter expression based on a custom field called customer_id, and also wants to attach a large, nested JSON object containing full request details to each trace segment for debugging purposes. The JSON object does not need to be searchable. How should the developer add these two pieces of data to the segment?

A) Add customer_id and the JSON payload both as annotations
B) Add customer_id as metadata and the JSON payload as an annotation
C) Add customer_id as an annotation and the JSON payload as metadata
D) Add both customer_id and the JSON payload as metadata

---

