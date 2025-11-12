# Step-by-step Implementation of CICD of ECS Fargate Deployment using Github Actions:

created a github workflow "cicd-workflow.yml" in the following repo: https://github.com/RajalakshmiNagendran/sample-app.git 

securing credentials for the pipeline: In Github as Repository secrets (Eg: AWS_ACCESS_KEY, AWS_SECRET_ACCESS_KEY)

  <img width="1500" height="600" alt="image" src="https://github.com/user-attachments/assets/eda99855-e9e8-40cc-8986-63115187a0fb" />


By this way Github actions will connect with AWS 

**steps involved in Github Actions for CICD:**
1. Checkout source
2. Login to Amazon ECR
3. Build, tag, and push image to Amazon ECR
4. Fill in the new image ID in the Amazon ECS task definition
5. Deploy Amazon ECS task definition (Implement PRE-PROD and PROD Environments)

Now if there is any push happened automatically the Github workflow will run and deploy the application in to the ECS cluster.

# Infrastructure security:

🔐 **1. Network & Infrastructure Security**
VPC Design

Private Subnets for Tasks — Run ECS tasks in private subnets; only ALBs/NLBs in public subnets.

No Public IPs — Disable public IP assignment for Fargate tasks.

NAT Gateways — Use NAT gateways for outbound internet access if required.

VPC Flow Logs — Enable for all VPCs to log accepted/rejected traffic.

Security Groups

Principle of Least Privilege — Restrict inbound/outbound ports.

Only ALB-to-Fargate Access — Only allow traffic from your ALB or specific SGs.

Egress Filtering — Restrict outbound to known destinations if possible.

Network ACLs

Use NACLs as an extra layer (e.g., deny all inbound except ALB CIDRs).

Service-to-Service Communication

Use AWS Cloud Map for internal service discovery.

Prefer PrivateLink for private access to AWS-managed services (e.g., S3, Secrets Manager).

🔑 **2. IAM (Identity and Access Management)**
Task Execution Role

Assign minimal permissions to ECS Task Execution Role (pull images, fetch secrets).

Example:

ecr:GetAuthorizationToken, ecr:BatchGetImage, logs:CreateLogStream, logs:PutLogEvents

Task Role (Runtime Permissions)

Use a separate Task Role for the container’s AWS API access (S3, DynamoDB, etc.).

Avoid wildcard permissions (*).

Least Privilege & Separation

Different roles per microservice.

Use IAM policies scoped to specific resources and actions.

IAM Boundaries

Implement permissions boundaries to control privilege escalation.

🧱 **3. Container Image Security**
Image Source Control

Use Amazon ECR (private registry).

Enable ECR image scanning (enhanced scanner).

Block unscanned/unapproved images using ECR lifecycle policies or CI/CD checks.

Image Hardening

Use minimal base images (e.g., alpine, distroless).

Remove unused tools, compilers, shells.

Set non-root users (USER directive in Dockerfile).

Multi-stage builds to avoid sensitive data in images.

Image Signing

Use AWS Signer or Notary v2 / Cosign for image signing & verification.

🧩 **4. ECS Task & Runtime Security**
Task Definition Security

Set "readonlyRootFilesystem": true where possible.

Set "user": "1000" or non-root users.

Use "privileged": false (never run privileged containers).

Limit container capabilities using "linuxParameters" (drop all unnecessary).

Set memory and CPU limits (prevents resource exhaustion).

ECS Task Networking Mode

Use awsvpc mode (each task gets its own ENI for isolation).

Avoid host networking unless absolutely necessary.

Secrets Management

Use AWS Secrets Manager or SSM Parameter Store for credentials.

Reference secrets in ECS task definitions (secrets block).

Do not bake secrets into environment variables or images.

🔐 **5. Data Security**
Encryption at Rest

EBS (Fargate storage): Enabled by default.

ECR images: Encrypted with AWS-managed keys.

S3 buckets: Enforce encryption (SSE-KMS).

Secrets Manager & SSM: Encrypted with KMS.

Encryption in Transit

Use HTTPS everywhere (ALB listeners, internal communication).

Enforce TLS 1.2+.

Use ACM certificates for managed certs.

🧭 **6. CI/CD Pipeline Security**
Build Pipeline Hardening

Use CodeBuild or GitHub Actions with least privilege IAM roles.

Scan Dockerfiles for vulnerabilities (Trivy, Grype, Anchore).

Validate images via ECR scan before deploy.

Automate infrastructure as code (IaC) using Terraform/CloudFormation.

Run IaC scanners (Checkov, tfsec, cfn-nag).

Deployment Controls

Use ECS deployment circuit breakers and ALB health checks.

Require manual approval for production promotions.

🛡️ **7. Monitoring, Logging & Detection**
AWS CloudWatch

Capture ECS task logs via FireLens → CloudWatch Logs / OpenSearch.

Monitor task metrics (CPU, memory, restarts).

AWS CloudTrail

Enable CloudTrail in all regions for API activity.

Store logs in secure S3 bucket with access logging and encryption.

GuardDuty

Detect anomalous or malicious behavior (network scans, crypto mining).

AWS Security Hub

Aggregate security findings (GuardDuty, Inspector, Config).

AWS Inspector

Scans container images in ECR for vulnerabilities continuously.

AWS Config

Monitor compliance with AWS best practices (ECS tasks in private subnets, encryption, etc.).

🧰 **8. Runtime & Threat Protection**

Integrate runtime protection tools:

AWS Inspector (for ECR)

AWS GuardDuty (for network threats)

Falco or Datadog Runtime Security for container behavior anomalies.

Monitor for:

Unexpected outbound network calls.

Privilege escalation.

Unexpected binary execution.

🧑‍💻 **9. Access Control & Operations**

MFA for all AWS users.

No long-lived IAM user access keys — use IAM roles & SSO.

Restrict ECS Exec Access — only allow trusted roles for debugging.

CloudTrail alerts for ECS Task definition updates or role changes.

🧮 **10. Compliance & Governance**

AWS Config Rules:

ecs-task-definition-no-root-user

ecs-container-insights-enabled

fargate-tasks-use-private-subnet

iam-policy-no-statements-with-admin-access

Audit Logs Retention — 365+ days for compliance.

Tag Resources for accountability (env, owner, cost center).

Implement SCPs (Service Control Policies) in AWS Organizations.

🧯 **11. Backup & DR**

Use EFS/EBS snapshots for stateful workloads (if any).

Backup configuration (CloudFormation/Terraform state).

Store critical secrets in multi-region Secrets Manager replication.

🧩 **12. Cost & Performance Hardening (Bonus)**

Set task resource limits to prevent cost spikes.

Use Spot Fargate for non-prod workloads.

Set budget alerts and CloudWatch anomaly detection.

✅ **Example Security Stack Summary**
| Layer              | Tool / Service               | Purpose                             |
| ------------------ | ---------------------------- | ----------------------------------- |
| **Network**        | VPC, SGs, NACLs, PrivateLink | Isolation                           |
| **IAM**            | Task Roles, Boundaries       | Least privilege                     |
| **Image Security** | ECR + Inspector              | Vulnerability scanning              |
| **Secrets**        | Secrets Manager              | Secure secret handling              |
| **Runtime**        | Fargate + Falco              | Runtime anomaly detection           |
| **Monitoring**     | CloudWatch + GuardDuty       | Observability & intrusion detection |
| **Compliance**     | Config + Security Hub        | Continuous compliance               |


Notes:
* espicially for fargate supported regions are available: https://docs.aws.amazon.com/AmazonECS/latest/developerguide/AWS_Fargate-Regions.html
* Amazon ECS service that uses Service Discovery - https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-discovery.html
* Apply container scanning using Trivy.
* Plan for roll back deployment.
* ECS Fargate security setup - https://chatgpt.com/share/6914349c-5a6c-8008-926d-9d62baec32c2(reference for ecs service discovery is really required).
* 


