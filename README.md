# Improved Architecture

![Improved Architecture](Improved%20Architecture.png)

# AWS Well-Architected & Cloud Adoption Framework Lab

**Lab Title:** Design and Evaluate an AWS Solution Using the Well-Architected and Cloud Adoption Frameworks

**Repository Overview**  
This repository contains my submission for the AWS lab on migrating a two-tier web application (frontend + backend database) from on-premises to AWS. The work applies the AWS Well-Architected Framework (WAF) five pillars and the AWS Cloud Adoption Framework (CAF) six perspectives to evaluate, recommend improvements, and design a modern, best-practice architecture.

**Key Files in this Repository**

- `aws_waf_caf_assessment.md` — Complete report covering all tasks:
  - Task 1: Review of existing architecture (components & risks/weaknesses)
  - Task 2: WAF evaluation table (five pillars with strengths, improvements, recommendations, and AWS services)
  - Task 3: CAF readiness summary (150–200 words per perspective: Business, People, Governance, Platform, Security, Operations)
  - Task 4: Description of the improved AWS architecture + how it addresses all five WAF pillars
  - Reflection: Brief 150-word summary of key learnings
- `architecture-diagram.drawio` (or `.png` / scanned image) — Visual diagram of the revised architecture (created/adapted in draw.io based on AWS best practices)
- This `README.md` — Explains my overall approach and methodology

**My Approach**
I followed a structured, step-by-step methodology aligned with the lab objectives and AWS best practices:

1. **Task 1 – Existing Architecture Review**  
   Started with the provided background of the on-premises two-tier app. Identified core components (web frontend server + relational database) and highlighted common migration risks: single-AZ deployment (no high availability), no automated backups, open security groups (poor least-privilege), lack of scalability, and no centralized monitoring. This set the baseline for improvements.

2. **Task 2 – Well-Architected Framework Evaluation**  
   Analyzed the hypothetical migrated workload against all **five WAF pillars**:
   - Operational Excellence
   - Security
   - Reliability
   - Performance Efficiency
   - Cost Optimization  
     For each pillar, I noted one realistic strength from the current/on-premises state, one clear area for improvement, a practical recommendation, and a specific AWS service/feature to implement it. Results are summarized in a clean Markdown table for easy reading.

3. **Task 3 – Cloud Adoption Framework (CAF) Analysis**  
   Evaluated organizational readiness across all **six CAF perspectives** (Business, People, Governance, Platform, Security, Operations).  
   Wrote concise ~150–200 word summaries per perspective, identifying current maturity level, key gaps, recommended actions/enablers (e.g., training, CCoE, AWS Landing Zone, policy updates), and how they support successful migration. This shows holistic thinking beyond just technical architecture.

4. **Task 4 – Improved Architecture Design**  
   Designed a revised, cloud-native two-tier architecture that directly addresses weaknesses from Task 1 and gaps from Tasks 2–3:
   - VPC with public/private subnets across multiple Availability Zones
   - Internet Gateway + NAT Gateway for secure connectivity
   - Application Load Balancer (ALB) in public subnets
   - EC2 instances in Auto Scaling Group (ASG) for frontend (multi-AZ)
   - Amazon RDS (Multi-AZ) in private subnets for the database (with automated backups & encryption)
   - Security groups with least-privilege rules, ACM for TLS, IAM roles
   - Optional: CloudFront + S3 for static assets (performance & cost)

   This design explicitly covers **all five WAF pillars**:
   - **Operational Excellence** — IaC (CloudFormation), CloudWatch monitoring
   - **Security** — Private subnets, least-privilege, encryption, WAF potential
   - **Reliability** — Multi-AZ, ASG, RDS failover & backups
   - **Performance Efficiency** — Elastic scaling, load balancing, caching potential
   - **Cost Optimization** — Auto Scaling to match demand, pay-as-you-go

   Diagram created in draw.io (exported as PNG/PDF) — inspired by AWS reference architectures but customized for this two-tier workload.

5. **Reflection**  
   Included a short 150-word reflection on lessons learned: importance of frameworks for critical evaluation, balancing pillars/trade-offs, organizational readiness beyond tech, and communicating decisions clearly.

**Tools & References Used**

- AWS Documentation: Well-Architected Framework, Cloud Adoption Framework, VPC best practices, RDS Multi-AZ, EC2 Auto Scaling
- Diagram tool: draw.io (Lucidchart alternative)
- No live AWS console used (hypothetical design), but design is validated against Well-Architected Tool patterns

This submission demonstrates thinking like a cloud architect: critical evaluation, framework-driven recommendations, best-practice alignment, and clear documentation.

Feel free to clone/fork and explore!
