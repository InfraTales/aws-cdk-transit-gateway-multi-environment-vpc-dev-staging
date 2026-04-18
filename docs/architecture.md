# Architecture Notes

## Overview

The CDK TypeScript project provisions multiple environment-specific VPCs (dev/staging/prod) connected via a Transit Gateway for controlled inter-VPC routing without VPC peering mesh sprawl. Each environment runs ECS Fargate workloads behind an Application Load Balancer, with Aurora Global Database providing cross-region replication and Secrets Manager handling credential injection at runtime. AWS Global Accelerator sits in front of the primary ALB, providing anycast routing and static IPs as a single global ingress point. The non-obvious design choice is using SSM Parameter Store alongside Secrets Manager — SSM carries non-sensitive environment config that ECS task definitions need at deploy time, while Secrets Manager handles DB credentials with automatic rotation.

## Key Decisions

- Aurora Global Database on a db.r6g.large with one replica adds roughly $400/month per region cluster compared to a single-region Aurora setup on the same instance class, but cuts RPO from hours (snapshot restore) to under 1 minute for cross-region replication lag [inferred]
- Transit Gateway charges $0.05/GB data processed plus hourly attachment fees per VPC — for low-traffic internal environments like dev, this costs more than VPC peering but eliminates the O(n²) peering mesh as environments grow [inferred]
- Global Accelerator runs at ~$18/month base plus $0.01/GB data transfer — justified for a trading platform where anycast routing and static IPs matter, but wasteful if you only need latency improvement for internal services [inferred]
- ECS Fargate removes EC2 fleet management overhead but makes burst cost unpredictable — a misconfigured task auto-scaling policy during a market open spike can triple compute costs within minutes with no natural ceiling [inferred]
- Sharing a single Transit Gateway across dev/staging/prod keeps the routing config DRY but means a misconfigured route table in dev can inadvertently open a path to prod subnets — environment isolation is a routing policy concern, not a hard network boundary [from-code]