# Chapter 9 Lab: Deployment Strategies, Elastic Beanstalk, and Containers

**Estimated time:** 20 minutes

## Exercise

Read an AWS CLI/CodeDeploy configuration shifting Lambda alias traffic 90/10 for a canary release, then compare an Amazon ECS task definition for the Fargate launch type against one for the EC2 launch type

## Key takeaway

Match the deployment strategy to the risk tolerance and rollback speed required, and pick EC2 launch type over Fargate only when you need instance-level control.
