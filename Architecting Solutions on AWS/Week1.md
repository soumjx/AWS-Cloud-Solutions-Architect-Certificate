#  Course Introduction (Notes)

- Course teaches how to think through customer requirements and select appropriate AWS services.
- Each week begins with a customer call to understand current architecture state, customer need, and implementation requirements. Solutions are built based on these scenarios.

## Week 1
- Customer: Online cleaning products company.
- Need: Improve resiliency and migrate order-processing service to AWS.
- Services: **AWS Lambda**, **Amazon SQS**, **Amazon SNS**, **API Gateway**, and related components.

## Week 2
- Customer: Software company using QR codes for restaurant menus.
- Need: Add data analysis for business intelligence.
- Services: **Amazon Athena**, **Amazon Kinesis**, **Amazon QuickSight**, and other analytics services.

## Week 3
- Customer: Insurance company with on-premises workloads.
- Need: Migrate to a hybrid environment (on-premises + AWS).
- Services: **AWS Direct Connect**, **AWS Database Migration Service**, **NAT gateways**, and networking/migration tools.

## Week 4
- Customer: Marketing agency using a single AWS account.
- Need: Multi-account environment, centralized logging, governance, and security.
- Services: **AWS Organizations**, **IAM Identity Center**, **AWS Service Catalog**, **AWS CloudTrail**, and governance services.

## Course Focus
- Understanding differences between AWS services to choose the right one for each use case.
- Not a deep dive into individual services; emphasis on service selection based on customer needs.
- Additional readings provide deeper service-specific information.


--------

# Customer #1: Use Case and Requirements (Notes)

## Business Context
- Company sells cleaning products globally through a public website, mobile app, and a wholesale buyer portal.
- All orders flow into an on-premises orders service that:
  - Validates, authenticates, accepts, and processes orders.
  - Stores orders in a MySQL database.
  - Calls downstream services (inventory fulfillment, accounting), which are already hosted on AWS.
- Orders service is the last major component remaining on-premises; migration includes a complete rewrite.

## Current Architecture
- Web server and backend application run on the same on-prem server.
- MySQL database stores all order data in a single table.
- Downstream services use their own databases.
- Payment processing is out of scope (handled by external gateway before backend call).

## Database Requirements
- Open to switching databases.
- Must be highly available and durable.
- Current MySQL instance is overkill for minimal data; maintaining it is time-consuming.
- Preference for a simpler, more hands-off AWS-managed database.

## Pain Points and Migration Drivers
- Business logic tightly coupled inside one monolithic service.
- Application slows down or crashes under load.
- Failed downstream calls cause inconsistent order state.
- Overwhelming traffic leads to slow or failed order acceptance.
- Hard to scale on premises; no automatic scaling.
- If app crashes mid-processing, only partial downstream updates occur.
- Need to decouple components and increase resilience.

## Demand Pattern
- Highly spiky traffic:
  - Huge spikes during sales or coupon campaigns.
  - Near-zero traffic at other times.
- On-prem hardware requires overprovisioning.

## Desired Outcomes
- Serverless and cloud-native architecture.
- Automatic scaling up and down based on demand.
- Decoupled downstream processing to prevent cascading failures.
- Reliable, independent delivery of updates to downstream services.
- Reduced operational overhead.
- Ability to fix code issues during rewrite.

## Additional Requirements
- Easy monitoring and logging.
- Consistent logging system across components.
- Optimize for **cost** and **performance**.
- Open to further clarification calls during architecture design.

-----------

# Selecting a Serverless Compute Service (Notes)

## Understanding the Current Compute Setup
- Existing order service runs as a single code package across multiple on-prem virtual machines.
- Goal: Break the application into smaller, decoupled components.
- Need a compute service that supports automatic scaling, high performance, and minimal operational overhead.

## Reviewing AWS Compute Options
- AWS compute categories:
  - **Instances**: EC2, Lightsail, Batch (not serverless).
  - **Containers**: ECS, EKS, AWS Fargate.
  - **Serverless**: AWS Lambda.

### Why Not EC2
- EC2 could support a lift-and-shift migration, but:
  - Not serverless.
  - Higher operational overhead.
  - Not optimized for the customer’s goals (scaling, reduced maintenance).
- Need to evaluate “best fit,” not just “works.”

### Considering Container Services (ECS, EKS, Fargate)
- Fargate provides serverless compute for containers.
- Benefits:
  - Managed scaling.
  - Logs/metrics sent to CloudWatch.
  - Low operational overhead.
  - Potential cost optimization (e.g., Fargate Spot).
- Concern:
  - Customer lacks container expertise.
  - Not currently using container technologies.

### Customer Feedback on Containers
- Customer is familiar only at a high level.
- No in-house container skill set.
- Team already learning cloud-native tools like Lambda and DynamoDB.
- Indicates containers (ECS/EKS/Fargate) are not a good fit.

## Selecting AWS Lambda
- Chosen compute service: **AWS Lambda**.
- Cloud-native, serverless, automatically scales.
- Very low operational overhead.
- Each request runs in a microVM (Firecracker) and shuts down afterward → no idle cost.
- Requires rewriting the code into Lambda-compatible functions.
- Well aligned with customer’s experience and migration direction.

## Adding API Gateway
- **Amazon API Gateway** as the frontend for Lambda:
  - Removes need for a web server.
  - Handles routing of HTTP requests to backend.
  - Can perform authentication and request validation.
  - Reduces custom code needed.
- Logs and metrics for Lambda and API Gateway go to **CloudWatch** for centralized monitoring.

## Resulting Architecture Components (So Far)
1. **API Gateway** – entry point for HTTP requests, handles validation/authentication.
2. **AWS Lambda** – hosts the order service application code, scales automatically.

## Alignment with Requirements
- Serverless (reduced operational burden).
- Managed scaling (handles spiky demand).
- High performance and cost effectiveness.
- Simplifies logging and monitoring.
- No introduction of new complex technology (containers).




-----------



-----------


-----------


-----------


-----------



-----------


-----------

