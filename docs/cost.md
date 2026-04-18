# Cost Model

## Overview

This is a reference cost model. Actual costs vary by usage, region, and configuration.

## Key Cost Drivers

- Aurora Global Database on a db.r6g.large with one replica adds roughly $400/month per region cluster compared to a single-region Aurora setup on the same instance class, but cuts RPO from hours (snapshot restore) to under 1 minute for cross-region replication lag [inferred]
- Transit Gateway charges $0.05/GB data processed plus hourly attachment fees per VPC — for low-traffic internal environments like dev, this costs more than VPC peering but eliminates the O(n²) peering mesh as environments grow [inferred]
- Global Accelerator runs at ~$18/month base plus $0.01/GB data transfer — justified for a trading platform where anycast routing and static IPs matter, but wasteful if you only need latency improvement for internal services [inferred]
- ECS Fargate removes EC2 fleet management overhead but makes burst cost unpredictable — a misconfigured task auto-scaling policy during a market open spike can triple compute costs within minutes with no natural ceiling [inferred]

## Estimated Monthly Cost

| Component | Dev (₹) | Staging (₹) | Production (₹) |
|-----------|---------|-------------|-----------------|
| Compute   | ₹2,000–5,000 | ₹8,000–15,000 | ₹25,000–60,000 |
| Database  | ₹1,500–3,000 | ₹5,000–12,000 | ₹15,000–40,000 |
| Networking| ₹500–1,000   | ₹2,000–5,000  | ₹5,000–15,000  |
| Monitoring| ₹200–500     | ₹1,000–2,000  | ₹3,000–8,000   |
| **Total** | **₹4,200–9,500** | **₹16,000–34,000** | **₹48,000–1,23,000** |

> Estimates based on ap-south-1 (Mumbai) pricing. Actual costs depend on traffic, data volume, and reserved capacity.

## Cost Optimization Strategies

- Use Savings Plans or Reserved Instances for predictable workloads
- Enable auto-scaling with conservative scale-in policies
- Use DynamoDB on-demand for dev, provisioned for production
- Leverage S3 Intelligent-Tiering for infrequently accessed data
- Review Cost Explorer weekly for anomalies
