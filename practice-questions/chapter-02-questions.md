## Chapter 2: Amazon API Gateway: Integrations, Authorizers, and Throttling — Practice Questions

**Q1.** A company is building a public-facing API that must support per-customer API keys, request throttling tiers sold to different partners, and request validation against a JSON schema before invoking the backend. Which API Gateway type should they choose?

A) REST API, because it supports usage plans, API keys, and request validation
B) HTTP API, because it is cheaper and has lower latency
C) HTTP API with a Lambda authorizer to simulate usage plans
D) REST API with a VPC Link to a private ALB

<details>
<summary>Answer</summary>

**Correct: A**

Usage plans, API keys, and JSON schema request validation are REST API-only features. HTTP APIs do not support usage plans, API keys, or request validation.

**Why the others are wrong:**
- HTTP API is cheaper/faster but cannot provide usage plans, API keys, or schema validation — it lacks the required feature set
- A Lambda authorizer can enforce auth but cannot replicate usage-plan throttling tiers or API key quotas
- VPC Link is unrelated to the requirement; it only enables private integrations, not usage plans

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q2.** A developer configures a REST API method with Lambda proxy integration. Inside the Lambda function, they try to read a custom mapping template variable that was previously used with a non-proxy setup. What happens?

A) The mapping template still runs and populates the variable before invoking Lambda
B) The mapping template is bypassed; the entire raw request is passed as the event object with no transformation
C) API Gateway throws a 500 error because proxy integration requires a mapping template
D) The variable is available only in the response mapping template, not the request

<details>
<summary>Answer</summary>

**Correct: B**

Lambda proxy integration passes the entire raw HTTP request (headers, query params, body, path params) directly into the Lambda event object. Request and response mapping templates are not used in proxy integration.

**Why the others are wrong:**
- Mapping templates are only used in custom (non-proxy) integration, not proxy integration
- Proxy integration does not require a mapping template — that's the whole point of proxy mode
- Response mapping templates also do not apply to proxy integration; the Lambda function must return the full response object itself

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q3.** An internal microservices API is called only by other AWS services within the same account using the AWS SDK, and requests must be signed. No third-party developers will ever call this API. Which authorization method requires the LEAST custom code to implement?

A) Lambda authorizer with a TOKEN type
B) Amazon Cognito user pool authorizer
C) Lambda authorizer with a REQUEST type
D) IAM authorization using SigV4

<details>
<summary>Answer</summary>

**Correct: D**

IAM authorization lets AWS SDKs sign requests automatically with SigV4 and enforces access via IAM policies attached to the calling principal — no custom authorizer code is needed for AWS-to-AWS calls.

**Why the others are wrong:**
- A Lambda TOKEN authorizer requires writing and maintaining custom validation logic, which is unnecessary overhead here
- Cognito user pools are designed for end-user identity (human sign-in), not machine-to-machine AWS SDK calls
- A REQUEST authorizer also requires custom Lambda code, adding complexity not needed for simple IAM-based SigV4 auth

</details>

---

**Q4.** Which statement correctly compares the default account-level throttle limits enforced by API Gateway per AWS Region?

A) 5,000 requests per second steady-state with a burst capacity of 10,000 requests
B) 10,000 requests per second steady-state with a burst capacity of 5,000 requests
C) 1,000 requests per second steady-state with no burst allowance
D) Unlimited steady-state throughput with a 5,000 request burst cap

<details>
<summary>Answer</summary>

**Correct: B**

The default account-level throttle in API Gateway is 10,000 requests per second (steady-state) with a burst limit of 5,000 requests, using a token bucket algorithm. This can be raised via a service quota increase request.

**Why the others are wrong:**
- This reverses the steady-state and burst values — burst is lower than steady-state, not higher
- There is a defined account-level limit; it is not unbounded
- Unlimited throughput is incorrect — API Gateway always enforces an account-level ceiling

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q5.** A team wants a single API Gateway stage to route to a different downstream Lambda alias depending on whether it's 'dev' or 'prod', without duplicating the API definition. Which feature should they use in the integration configuration?

A) Usage plans with separate API keys per environment
B) Two separate mapping templates selected by a request header
C) A canary deployment configuration on the prod stage
D) Stage variables referenced inside the integration URI

<details>
<summary>Answer</summary>

**Correct: D**

Stage variables act like environment variables for a stage and can be referenced in the Lambda integration ARN/alias (e.g., ${stageVariables.lambdaAlias}), letting one API definition route to different backend resources per stage.

**Why the others are wrong:**
- Usage plans control throttling and quotas, not backend routing per environment
- Custom mapping templates can inspect headers but this is not the standard mechanism for switching Lambda aliases per stage
- Canary deployments shift a percentage of traffic to a new version within the same stage — they don't select environments

</details>

---

**Q6.** A non-proxy (custom) REST API integration needs to transform an incoming JSON request body before sending it to a legacy SOAP backend. Which templating language does API Gateway use for this transformation?

A) JSONPath expressions only
B) AWS CloudFormation intrinsic functions
C) Velocity Template Language (VTL)
D) Jinja2 templates

<details>
<summary>Answer</summary>

**Correct: C**

API Gateway request/response mapping templates for custom (non-proxy) integrations are written in Velocity Template Language (VTL), which can reference context, input, and util variables to transform payloads.

**Why the others are wrong:**
- JSONPath can be used inside VTL via $input.path() but is not itself the templating language
- CloudFormation intrinsic functions apply to infrastructure templates, not API Gateway payload mapping
- Jinja2 is a Python templating engine unrelated to API Gateway mapping templates

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q7.** A company exposes an HTTP API secured with a JWT authorizer backed by an Amazon Cognito user pool. A mobile client calls the API without an Authorization header. What response should the developer expect?

A) 401 Unauthorized, because the JWT authorizer requires a valid bearer token
B) 200 OK, because JWT authorizers are optional by default
C) 403 Forbidden, because IAM policy evaluation denies the request
D) 502 Bad Gateway, because the backend integration failed

<details>
<summary>Answer</summary>

**Correct: A**

When a route is protected by a JWT authorizer and no token (or an invalid one) is presented, API Gateway returns 401 Unauthorized before invoking the backend integration.

**Why the others are wrong:**
- JWT authorizers are enforced per configured route; they are not optional once attached
- 403 is returned for IAM/resource-policy denials or scope mismatches, not for a missing token entirely
- 502 indicates a backend integration error, but the request never reaches the backend if authorization fails first

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q8.** Which pricing and feature statement about REST APIs versus HTTP APIs is accurate?

A) HTTP APIs cost more per million requests than REST APIs but offer lower latency
B) HTTP APIs support usage plans and API keys at a lower price point than REST APIs
C) REST APIs and HTTP APIs have identical feature sets, differing only in name
D) REST APIs support edge-optimized endpoints, caching, and WAF integration; HTTP APIs do not support these features but cost less per request

<details>
<summary>Answer</summary>

**Correct: D**

REST APIs offer the full feature set — edge-optimized endpoints, response caching, AWS WAF integration, usage plans, and request validation — while HTTP APIs strip these out in exchange for significantly lower per-request pricing and reduced latency.

**Why the others are wrong:**
- HTTP APIs are priced lower per million requests than REST APIs, not higher
- Usage plans and API keys are REST API-only features; HTTP APIs do not support them at any price
- The feature sets differ substantially — HTTP APIs are a deliberately reduced feature subset, not a renaming of REST APIs

</details>

---

