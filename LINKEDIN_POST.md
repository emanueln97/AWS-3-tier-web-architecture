🚀 Project Drop: AWS 3‑Tier Web Architecture (My First Real‑World Build)

I just completed a production‑style 3‑tier web app on AWS with:
• Internet‑facing ALB → Internal ALB → EC2 Auto Scaling (app tier)
• Private DB subnets with Amazon RDS/Aurora
• Segmented Security Groups + least‑privilege IAM
• NAT for controlled outbound from private subnets
• Multi‑AZ subnets & health checks for resilience

What I learned:
• Designing clean VPC/subnet boundaries and route tables
• Getting ALB/target groups + ASG health checks working smoothly
• Hardening access between tiers (only what’s needed, nothing more)
• Validating DB connectivity from app tier only

Repo (screens, diagram, and notes): 🔗 (https://github.com/kaldurjoy/AWS-3-tier-web-achitecture/tree/main)
If you’re exploring AWS architecture or hiring for cloud roles, I’d love your feedback and suggestions!

#AWS #Cloud #SolutionsArchitect #DevOps #Architecture #VPC #ALB #RDS #EC2 #IaC
