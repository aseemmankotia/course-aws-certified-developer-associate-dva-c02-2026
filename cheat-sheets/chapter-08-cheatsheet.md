## CloudFormation Template Anatomy
| Section | Purpose |
|---|---|
| Parameters | Runtime input values (no hardcoding) |
| Mappings | Static key-value lookup tables (e.g., region → AMI ID) |
| Conditions | Boolean logic controlling resource creation |
| Resources | **Required.** The actual AWS resources to create |
| Outputs | Values exposed after stack creation; can be `Export`ed for cross-stack use |
| Transform | Enables macros — required for SAM: `AWS::Serverless-2016-10-31` |

## Intrinsic Functions
| Function | Use | Scope |
|---|---|---|
| `Ref` | Get default value of a resource/parameter | Same template |
| `Fn::GetAtt` | Get a specific resource attribute (e.g., Arn) | Same template |
| `Fn::Sub` | Inline string substitution with `${Var}` | Same template |
| `Fn::Join` | Concatenate list of strings with delimiter | Same template |
| `Fn::ImportValue` | Retrieve an exported Output value | Cross-stack |

## Change Sets
```bash
aws cloudformation create-change-set \
  --stack-name my-stack --change-set-name preview1 \
  --template-body file://template.yaml
aws cloudformation describe-change-set --change-set-name preview1 --stack-name my-stack
aws cloudformation execute-change-set --change-set-name preview1 --stack-name my-stack
```
- Preview only — nothing applied until `execute-change-set`.
- Shows Add / Modify / **Remove-and-Replace** actions (catches destructive updates like RDS replacement).

## Nested Stacks
- Resource type: `AWS::CloudFormation::Stack` (`TemplateURL` points to child template in S3)
- Max nesting depth: **5 levels**
- Use for modular, reusable infra within one deployment (vs. cross-stack exports for independent stacks)

## AWS SAM
- Required header: `Transform: AWS::Serverless-2016-10-31`
- Simplified types: `AWS::Serverless::Function`, `AWS::Serverless::Api`, `AWS::Serverless::SimpleTable`, `AWS::Serverless::Application`
- SAM auto-generates IAM roles, event source mappings, API Gateway integration
- `sam build` → `sam deploy` (wraps CloudFormation create/update-stack)

## AWS CDK
- Imperative code (TypeScript, Python, Java, C#) → `cdk synth` → CloudFormation template → `cdk deploy`
- Building blocks: **App** → **Stack** (1:1 with CFN stack) → **Construct** (L1 raw / L2 curated / L3 pattern)
- CDK does NOT replace CloudFormation — it's a code generator for it

## Key Limits (default, per account/region)
| Limit | Value |
|---|---|
| Template size (direct upload) | 51,200 bytes |
| Template size (via S3) | 1,048,576 bytes (1MB) |
| Resources per stack | 500 |
| Parameters per template | 200 |
| Outputs per template | 200 |
| Nested stack depth | 5 |

## Common Exam Traps
- Using `Ref` when the exam wants a specific attribute → should be `Fn::GetAtt`
- Forgetting `Transform:` in a SAM template → resource type errors
- Trying to delete/update an exported Output still imported elsewhere → blocked by CloudFormation
- Assuming change sets auto-apply — they only PREVIEW until executed
