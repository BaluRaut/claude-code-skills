---
name: vpc-networking
description: VPC decisions for serverless — when a Lambda joins a VPC, endpoints vs NAT, security-group discipline, and EC2 access via SSM instead of SSH.
---

# VPC & networking

## 1. When a Lambda joins a VPC (and when it must not)

- Joins ONLY to reach private resources: RDS/Proxy, ElastiCache, private
  ALBs, EKS-hosted internal services. DynamoDB/S3/SQS/EventBridge/Secrets
  need NO VPC — putting every function in the VPC "for security" adds
  failure modes and cost, not security
- The moment it joins: it loses direct internet. Inventory what ELSE it
  calls BEFORE the move (rds-postgres §4) — third-party APIs now need NAT;
  AWS APIs should use endpoints (§2), not NAT

## 2. Endpoints before NAT (cost + reliability)

- **Gateway endpoints (free): S3, DynamoDB** — always on in any VPC that
  touches them
- Interface endpoints for what the functions call from inside [adapt:
  Secrets Manager, SSM, SQS, events] — per-AZ hourly cost, usually still
  cheaper and more reliable than routing AWS traffic through NAT
- NAT gateway is for genuine third-party egress only; it's per-AZ +
  per-GB — an unexplained NAT bill is usually AWS API traffic that should
  be an endpoint

## 3. Subnets & security groups

- Lambdas/containers in private subnets across ≥2 AZs; public subnets are
  for NAT/ALB/edge only — a workload in a public subnet is a review flag
- SG rules reference SGs, not CIDRs: `db-sg allows 5432 FROM lambda-sg` —
  self-documenting and survives IP churn. `0.0.0.0/0` inbound on anything
  but the public ALB/443 fails review
- One SG per role (lambda-sg, db-sg, proxy-sg) [adapt naming], defined in
  [the infra repo — adapt] and referenced by ID/SSM from serverless.yml

## 4. EC2 — access without SSH

- Access via **SSM Session Manager**: no port 22 open, no key files
  passed around, sessions logged. An SG with 22 open to the world is an
  immediate finding
- EC2 boxes carry instance PROFILES (roles) — no AWS keys on disk; least
  privilege same as Lambda IAM (serverless-v3-config §2)

## 5. Verify

From a dev invoke inside the VPC: DB reachable, S3/Dynamo calls succeed
(via endpoints — check flow logs/endpoint metrics, not NAT), third-party
call works only if NAT was deliberately provisioned. `aws ssm
start-session` works on the EC2 box; port 22 closed.
