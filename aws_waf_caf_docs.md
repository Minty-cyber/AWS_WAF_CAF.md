# AWS Well-Architected & Cloud Adoption Framework Assessment

**Lab Title:** Design and Evaluate an AWS Solution Using the Well-Architected and Cloud Adoption Frameworks  
**Submitted by:** Jeffrey Mintah  
**Date:** February 2026

The AWS Well-Architected Framework now includes **six pillars**: Operational Excellence, Security, Reliability, Performance Efficiency, Cost Optimization, and **Sustainability** (focused on minimizing environmental impact through efficient resource use, energy optimization, and waste reduction).

## Task 1 – Review the Existing Architecture

### 1. Components of the workload (current lift-and-shift deployment on AWS)

| AWS Resource              | Details                                                                 |
|---------------------------|-------------------------------------------------------------------------|
| Amazon RDS                | Single-AZ MySQL or PostgreSQL instance in a private subnet              |
| EC2 instances             | 1–2 t3.medium or m5.large instances running the web/frontend/application tier |
| Security Groups           | Web SG: ports 80, 443, 22 open from 0.0.0.0/0                           |
| EBS volumes               | Root and data volumes attached to EC2                                   |
| RDS automated backups     | Enabled with default 7-day retention                                    |
| Internet Gateway          | Attached to the VPC; EC2 instances directly reachable from internet     |
| VPC                       | Single VPC with public and private subnets                              |
| Elastic IP / Public DNS   | Directly pointing to EC2 instance (no load balancer)                    |

### 2. Potential risks or weaknesses
- **Single-AZ deployment** — EC2 and RDS in one AZ; Availability Zone failure causes full outage.
- **No load balancer** — Direct traffic to 1–2 EC2 instances; single point of failure, no health checks.
- **No Auto Scaling** — Cannot handle traffic spikes or scale down to save costs.
- **Excessively open security groups** — Port 22 (SSH) and 80/443 open to 0.0.0.0/0 → high risk of compromise.
- **Direct SSH key access** — Hard to audit/manage; violates best practices.
- **Single-AZ RDS** — No automatic failover; potential data loss on failure.
- **Weak backup strategy** — Only default 7-day RDS backups; no manual snapshots or cross-region copies.
- **No web application firewall** — Vulnerable to SQL injection, XSS, etc.
- **No CDN** — Slow global load times; all traffic hits EC2 directly.
- **Limited monitoring** — Basic CloudWatch only; issues often detected by users.
- **Always-on fixed instances** — Paying for idle capacity 24/7 (also increases unnecessary energy consumption).

## Task 2 – Well-Architected Framework Evaluation

| Pillar                  | Observation (Strength)                          | Improvement Needed                          | Recommendation                                      | Supporting AWS Service                  |
|-------------------------|-------------------------------------------------|---------------------------------------------|-----------------------------------------------------|-----------------------------------------|
| **Operational Excellence** | Infrastructure in cloud enables automation potential | Lacks centralized monitoring & IaC          | Implement observability, IaC, and automated ops     | CloudWatch, CloudTrail, AWS Config, CloudFormation |
| **Security**            | Moved away from on-prem unsecured network       | Security groups too open + unencrypted data | Least-privilege rules, encryption, no public SSH    | VPC Security Groups, KMS, IAM, ACM, Secrets Manager |
| **Reliability**         | Easy to introduce backups in AWS                | Single-AZ risks full downtime               | Multi-AZ deployment + automatic failover            | RDS Multi-AZ, Auto Scaling Group, ALB   |
| **Performance Efficiency** | Scalable compute available on-demand          | Manual scaling leads to overload/underuse   | Elastic scaling + caching/CDN                       | Auto Scaling Group, CloudFront, ElastiCache |
| **Cost Optimization**   | Pay-as-you-go eliminates upfront hardware costs | Over-provisioned always-on resources        | Right-size + scale to demand + committed discounts  | Auto Scaling, Compute Savings Plans, Trusted Advisor |
| **Sustainability**      | Cloud infrastructure allows for efficient hardware choices | Always-on fixed instances waste energy; no demand alignment | Maximize utilization, scale dynamically, prefer efficient services/hardware | Auto Scaling Group, Graviton instances, Managed Services (e.g., RDS, ALB), AWS Compute Optimizer |

## Task 3 – Apply the AWS Cloud Adoption Framework (CAF)

### Business Perspective
The organization is motivated to adopt AWS for better scalability, reduced infrastructure costs, and faster delivery. However, maturity remains partial — no formal cloud value roadmap, TCO/ROI model, or defined KPIs (e.g., uptime gains, faster time-to-market, customer satisfaction). Business stakeholders recognize cloud benefits but lack quantified justification or executive alignment.  
**Key actions/enablers:** Establish strong executive sponsorship, develop a business case with AWS TCO/ROI tools, define success metrics, and align cloud goals with revenue growth and competitiveness. This ensures the migration delivers strategic value beyond IT cost savings.

### People Perspective
The IT team has strong on-premises experience but limited AWS/cloud-native skills (e.g., VPC, managed services, IaC). Roles are still traditional; change management is weak.  
**Key actions/enablers:** Invest in upskilling via AWS Skill Builder, certifications, workshops, and labs. Establish a Cloud Center of Excellence (CCoE), mentorship, and agile/DevOps practices. Redefine roles for shared responsibility. Effective training and communication will reduce resistance and enable the team to operate the new architecture confidently.

### Governance Perspective
No cloud-specific policies exist (tagging, budgeting, compliance), risking uncontrolled costs and security gaps.  
**Key actions/enablers:** Define mandatory tagging, budget alerts, SCPs via AWS Organizations, and lifecycle policies. Use AWS Control Tower for landing zone guardrails and Trusted Advisor for best practices. Involve finance/procurement in variable pricing education. Mandate periodic Well-Architected reviews. Strong governance will support secure, compliant scaling post-migration.

### Platform Perspective
Current setup is functional but not highly available, automated, or cloud-native (single-AZ, manual ops).  
**Key actions/enablers:** Adopt Multi-AZ, load balancing, Auto Scaling, and IaC (CloudFormation). Standardize VPC patterns (public/private subnets, NAT), use managed services (RDS Multi-AZ, ALB), and implement CI/CD pipelines. This creates a repeatable, resilient platform foundation for the improved architecture.

### Security Perspective
Security maturity is low — still using perimeter-based thinking instead of cloud-native controls. Open ports and manual SSH persist from on-prem habits.  
**Key actions/enablers:** Embrace Shared Responsibility Model. Enforce least-privilege IAM roles, encrypt data (KMS/ACM), use private subnets, Security Groups/NACLs, and AWS WAF on ALB. Enable CloudTrail, GuardDuty, Security Hub for visibility. Define compliance early (e.g., GDPR/PCI). This shifts to proactive, secure-by-design posture.

### Operations Perspective
Operations rely on manual maintenance; monitoring is reactive.  
**Key actions/enablers:** Move to proactive CloudWatch metrics/alarms/dashboards, automated patching (Systems Manager), runbooks, and incident response plans. Define RPO/RTO and test DR. Automate deployments and reviews. This reduces downtime, improves reliability, and aligns with Operational Excellence.

## Task 4 – Design an Improved Architecture

### Improved Architecture Description (see architecture-diagram.png / draw.io file)
The revised design modernizes the lift-and-shift setup into a secure, highly available two-tier web application:
- Custom **VPC** with public and private subnets across **multiple Availability Zones** (e.g., ap-southeast-1a/b).
- **Internet Gateway** for public inbound traffic.
- **NAT Gateway** (in public subnet) for secure outbound from private resources.
- **Application Load Balancer (ALB)** in public subnets with **ACM Certificate** for HTTPS (domain routing via Route 53 recommended).
- **Frontend tier**: EC2 instances in **Auto Scaling Group (ASG)** across AZs, behind ALB, in public or private subnets (private preferred for extra security).
- **Backend database**: **Amazon RDS Multi-AZ** (MySQL/PostgreSQL) in private subnets with automated backups, encryption (KMS), and failover replica.
- **Security Groups**: Least-privilege — ALB allows 80/443 from internet; frontend allows traffic only from ALB; RDS allows only from frontend (e.g., port 3306/5432).
- Route tables: Public → IGW; Private → NAT.
- Optional: **CloudFront + S3** for static assets (images/CSS/JS) to improve global performance.

This matches AWS best practices for two-tier apps (public-facing ALB → private compute → private DB).

### How the new design aligns with the six WAF pillars
- **Operational Excellence** — IaC (CloudFormation) for repeatable setup; Session Manager (no open port 22); CloudWatch monitoring/alarms.
- **Security** — Private subnets for RDS & backend; least-privilege SGs (ALB → EC2 only, EC2 → RDS only); ACM TLS; no public SSH; potential WAF on ALB.
- **Reliability** — Multi-AZ across ALB, ASG, RDS (automatic failover); health checks remove unhealthy instances.
- **Performance Efficiency** — ALB intelligent routing; ASG elastic scaling; CloudFront CDN for static content reduces latency/EC2 load.
- **Cost Optimization** — ASG scales down during low traffic (min 1–2, can go to 0); Savings Plans for predictable usage; pay only for used resources.
- **Sustainability** — Dynamic scaling (ASG) minimizes idle resources/energy waste; managed services (RDS Multi-AZ, ALB) reduce operational overhead; potential use of efficient hardware (e.g., Graviton instances); CloudFront reduces data transfer energy; right-sizing and stopping unused assets align with demand to lower overall environmental impact.

The design fully addresses Task 1 risks (single-AZ, open ports, no scaling, no CDN, etc.) and supports CAF perspectives (managed services reduce People/Operations effort; standardized pattern aids Governance/Platform).

## Brief Reflection
After earning my Cloud Practitioner certification, this lab connected theory to practice. I finally saw how the **six Well-Architected pillars** — now including Sustainability — apply to real workloads, not just exam memorization. The “aha” moment was realizing a simple lift-and-shift isn’t enough; high availability, least-privilege security, cost awareness, **and environmental efficiency** must be built in from day one. The CAF section showed migration success depends as much on people, governance, and business alignment as on technology. Drawing the improved architecture felt like real cloud architect work — balancing pillars and trade-offs, including reducing energy waste through scaling and managed services. I now understand why AWS stresses “build it right the first time.” This transformed my CCP knowledge into practical thinking. Thank you for the opportunity!

**References**
- AWS Well-Architected Framework (including Sustainability Pillar): https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html
- AWS Cloud Adoption Framework
- AWS VPC / ALB / RDS / Auto Scaling / Sustainability best practices documentation
