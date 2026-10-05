# AWS for Network Administrators

A 35-day study path through AWS networking, written for people who already run
production networks and want the AWS model mapped onto what they know. Every day
covers one topic, ordered so each one builds on the last.

No ML, no AI, no serverless deep dives. Networking only.

## How a day works

Each of the 35 day files follows the same structure:

- **Concept** — what it is, and why a network admin should care
- **How it works** — architecture, components, limits
- **Hands-on** — CLI and console walkthrough
- **Key commands** — `aws` CLI quick reference
- **Routing and design notes** — what it means for traffic flow
- **Gotchas** — the traps
- **Cost notes** — what this bills for

## Prerequisites

- Working knowledge of CIDR, TCP/IP, DNS and routing
- An AWS account, free tier is enough
- AWS CLI installed and configured

## Weeks

**Week 1 — Foundations (days 1-7)** · the building blocks

| Day | Topic |
|-----|-------|
| 1 | [Global Infrastructure](day-01-global-infrastructure.md) — regions, AZs, edge locations |
| 2 | [IAM for Network Admins](day-02-iam-for-network-admins.md) — users, roles, policies |
| 3 | [VPC Deep Dive](day-03-vpc-deep-dive.md) — CIDR, default vs custom |
| 4 | [Subnets, Route Tables, IGW](day-04-subnets-route-tables-igw.md) |
| 5 | [NAT Gateway, NAT Instance, Egress-Only IGW](day-05-nat-gateway-egress-only-igw.md) |
| 6 | [Security Groups](day-06-security-groups.md) — stateful firewall |
| 7 | [NACLs, ENI, EIP](day-07-nacls-eni-eip.md) — stateless firewall |

**Week 2 — Compute, Governance, Advanced VPC (days 8-15)** · what runs in it

| Day | Topic |
|-----|-------|
| 8 | [EC2 & Compute Fundamentals](day-08-ec2-compute.md) — instances, AMIs, key pairs, Session Manager |
| 9 | [Core AWS Services](day-09-core-services.md) — S3, EBS, RDS, CloudWatch, CloudTrail |
| 10 | [Organizations, RAM & Config](day-10-organizations-ram-config.md) — governance, compliance |
| 11 | [GWLB & Traffic Inspection](day-11-gwlb-traffic-inspection.md) — appliance mode on TGW |
| 12 | [Container Networking & IaC](day-12-container-networking-iac.md) — ECS, EKS |
| 13 | [VPC Peering](day-13-vpc-peering.md) — cross-account, cross-region |
| 14 | [Transit Gateway](day-14-transit-gateway.md) — hub and spoke |
| 15 | [VPC Endpoints](day-15-vpc-endpoints.md) — PrivateLink |

**Week 3 — Connectivity, DNS, Load Balancing (days 16-22)** · getting in and distributing

| Day | Topic |
|-----|-------|
| 16 | [Site-to-Site VPN](day-16-site-to-site-vpn.md) |
| 17 | [Client VPN](day-17-client-vpn.md) |
| 18 | [Direct Connect](day-18-direct-connect.md) — dedicated, hosted, LAG |
| 19 | [Route 53](day-19-route53.md) — hosted zones, routing policies |
| 20 | [Elastic Load Balancers](day-20-elastic-load-balancers.md) — ALB, NLB, CLB |
| 21 | [CloudFront](day-21-cloudfront.md) — distributions, origins, behaviors |
| 22 | [AWS WAF](day-22-waf.md) — web ACLs, rate limiting |

**Week 4 — Security & Monitoring (days 23-28)** · protecting and watching

| Day | Topic |
|-----|-------|
| 23 | [AWS Shield](day-23-shield.md) — DDoS |
| 24 | [Network Firewall](day-24-network-firewall.md) — stateful inspection |
| 25 | [Global Accelerator](day-25-global-accelerator.md) — anycast |
| 26 | [VPC Flow Logs](day-26-vpc-flow-logs.md) — capture and analyze |
| 27 | [Reachability Analyzer, Network Manager, IPAM](day-27-reachability-analyzer-network-manager-ipam.md) |
| 28 | [Route 53 Resolver](day-28-route53-resolver.md) — inbound/outbound endpoints |

**Week 5 — Hybrid & Capstone (days 29-35)** · putting it together

| Day | Topic |
|-----|-------|
| 29 | [PrivateLink & VPC Lattice](day-29-privatelink-vpc-lattice.md) — service to service |
| 30 | [Multi-Region Architecture](day-30-multi-region-architecture.md) — DR, cross-region |
| 31 | [Hybrid Networking](day-31-hybrid-networking.md) — VPN + DX + TGW together |
| 32 | [Network Security](day-32-network-security.md) — encryption, TLS, compliance |
| 33 | [Edge Networking](day-33-edge-networking.md) — Local Zones, Wavelength, Outposts |
| 34 | [Cost Optimization for Networking](day-34-cost-optimization-networking.md) |
| 35 | [Capstone](day-35-capstone.md) — enterprise multi-VPC, multi-region, hybrid design |

## Also here

| File | Contents |
|------|----------|
| [aws-role-vs-policy.md](aws-role-vs-policy.md) | Roles vs policies, the distinction that trips everyone up |
| `files/` | IAM JSON: trust roles and CloudWatch monitoring policies used in the labs |

## Lab cost

Almost everything here runs on free tier if you tear things down as you go. The
exceptions to watch are NAT Gateways (billed hourly per gateway, not per
request), data transfer out, and Gateway Load Balancers. Days 5, 14 and 11 are
where a forgotten resource costs the most.
