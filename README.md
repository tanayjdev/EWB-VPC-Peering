# AWS VPC Peering — Secure Private Cross-VPC Communication

A hands-on AWS networking project completed as part of the **Amazon Web Services (AWS) Cloud Computing Internship** with EWB Edutech Pvt. Ltd. The project demonstrates private IPv4 connectivity between two non-overlapping Amazon Virtual Private Clouds (VPCs), using VPC Peering, three EC2 instances, explicit route-table configuration, restrictive security groups, and bastion-based SSH access.

> **Project status:** Connectivity was successfully validated during testing. The three project EC2 instances were subsequently terminated, and the project VPCs were removed. This repository documents the tested architecture and its recorded results; it is not a currently running deployment.

---

## Table of Contents

- [Project Information](#project-information)
- [Project Objectives](#project-objectives)
- [Architecture](#architecture)
- [Network Design](#network-design)
- [AWS Resource Inventory](#aws-resource-inventory)
- [Routing Configuration](#routing-configuration)
- [Security Configuration](#security-configuration)
- [SSH Access Through the Bastion](#ssh-access-through-the-bastion)
- [Testing and Results](#testing-and-results)
- [Baseline Evidence Disclosure](#baseline-evidence-disclosure)
- [Architecture and Networking Concepts](#architecture-and-networking-concepts)
- [Cleanup and Cost Considerations](#cleanup-and-cost-considerations)
- [Repository Structure](#repository-structure)
- [Reproducing the Lab](#reproducing-the-lab)
- [Mandatory Viva Questions](#mandatory-viva-questions)
- [Learning Outcomes](#learning-outcomes)
- [Author](#author)

---

## Project Information

| Field | Details |
|---|---|
| Student | Tanay Jain |
| College | Birla Institute of Technology, Mesra — Jaipur Campus |
| Internship | Amazon Web Services (AWS) Cloud Computing Internship |
| Organization | EWB Edutech Pvt. Ltd. |
| AWS Region | Asia Pacific (Mumbai) — `ap-south-1` |
| Availability Zone | `ap-south-1a` |
| Completion date | 19 October 2026 |
| Primary AWS services | Amazon VPC, EC2, VPC Peering, Internet Gateway, Route Tables, Security Groups |
| Evidence | 22 screenshots documenting configuration, validation, and cleanup |

## Project Objectives

The project was designed to demonstrate how two independent VPCs can communicate privately through a VPC Peering connection.

The main objectives were to:

1. Design two VPCs with non-overlapping IPv4 CIDR blocks.
2. Create public and private subnets according to the prescribed architecture.
3. Deploy a public bastion host and two private application instances.
4. Configure an Internet Gateway for the public subnet in VPC A.
5. Establish and accept a VPC Peering connection between VPC A and VPC B.
6. Configure the required routes in all three route tables.
7. Restrict administrative access through security-group rules.
8. Access private EC2 instances using OpenSSH `ProxyJump`.
9. Validate private connectivity using ICMP Echo Request tests in both directions.
10. Demonstrate controlled network isolation through a temporary security-group rule change, then restore connectivity.
11. Capture evidence and clean up the project resources after testing.

---

## Architecture

![AWS VPC Peering Architecture](02_Architecture/EWB_VPC_Peering_Architecture.jpg)

*Figure 1. Project architecture at validation time, including both VPCs, their subnets, EC2 instances, the Internet Gateway, the VPC Peering connection, and the associated routing design.*

### Traffic flow

**Administrative access:**

`Administrator workstation → Bastion public IPv4 → Bastion private IP → App-A or App-B`

**Private workload communication:**

`App-A (10.10.2.10) ↔ VPC Peering ↔ App-B (10.20.1.10)`

The application instances were configured without public IPv4 addresses. Their administrative SSH connections were routed through the bastion host.

**Important:** The architecture represents the configuration during testing. The project resources were removed during subsequent cleanup; the diagram does not indicate that the environment is still running.

---

## Network Design

The two VPCs use non-overlapping IPv4 address ranges, allowing their address spaces to communicate through VPC Peering.

### VPC A — `ewb-vpc-a`

| Property | Value |
|---|---|
| CIDR block | `10.10.0.0/16` |
| Public subnet | `ewb-a-public` |
| Public subnet CIDR | `10.10.1.0/24` |
| Private subnet | `ewb-a-private` |
| Private subnet CIDR | `10.10.2.0/24` |
| Internet Gateway | `ewb-igw-a` |
| Internet Gateway ID | `igw-0eb224e3fb0322445` |

VPC A contains the public bastion host and App-A in a private subnet. Its public route table includes the local VPC route, an Internet Gateway default route, and a route to VPC B through the peering connection.

### VPC B — `ewb-vpc-b`

| Property | Value |
|---|---|
| CIDR block | `10.20.0.0/16` |
| Private subnet | `ewb-b-private` |
| Private subnet CIDR | `10.20.1.0/24` |
| Internet Gateway | None configured for this project |

VPC B contains App-B in a private subnet. Its route table includes the local VPC route and a route to VPC A through VPC Peering.

---

## AWS Resource Inventory

The following identifiers describe the resources at the time of project testing. They are historical resource identifiers, not evidence that those resources remain active.

### VPCs and subnets

| Resource | Name | AWS identifier | CIDR |
|---|---|---|---|
| VPC A | `ewb-vpc-a` | `vpc-0a6ce73ff02b1503f` | `10.10.0.0/16` |
| VPC B | `ewb-vpc-b` | `vpc-075df51206347c58f` | `10.20.0.0/16` |
| Public subnet A | `ewb-a-public` | `subnet-0484e07086d26edb5` | `10.10.1.0/24` |
| Private subnet A | `ewb-a-private` | `subnet-0d7771459ff9a714d` | `10.10.2.0/24` |
| Private subnet B | `ewb-b-private` | `subnet-0b71be209ac84f3f3` | `10.20.1.0/24` |

### Route tables

| Resource | Name | AWS identifier |
|---|---|---|
| Public route table A | `ewb-rt-a-public` | `rtb-07eb6affd6f3a0e73` |
| Private route table A | `ewb-rt-a-private` | `rtb-0e0dde41748dfa436` |
| Private route table B | `ewb-rt-b-private` | `rtb-0dfff1ec80beab2f9` |

### Peering and Internet Gateway

| Resource | Name | AWS identifier | Recorded state |
|---|---|---|---|
| Internet Gateway | `ewb-igw-a` | `igw-0eb224e3fb0322445` | Attached during testing |
| VPC Peering connection | `ewb-a-b-peer` | `pcx-03d90d239a4ae9a9b` | `Active` during validation |

### Security groups

| Resource | Name | AWS identifier |
|---|---|---|
| Bastion security group | `ewb-sg-bastion` | `sg-0dd786e889c8dc6c9` |
| App-A security group | `ewb-sg-app-a` | `sg-06ebed5e7197ab5d5` |
| App-B security group | `ewb-sg-app-b` | `sg-0ef25547ef3d026e5` |

### EC2 instances

All three instances used the `t3.micro` instance type in Availability Zone `ap-south-1a`.

| Instance | Instance ID | Private IPv4 | Public IPv4 during testing |
|---|---|---|---|
| `ewb-bastion` | `i-0a96fe6409821ab86` | `10.10.1.10` | `13.232.128.100` |
| `ewb-app-a` | `i-07875dafbce902beb` | `10.10.2.10` | None |
| `ewb-app-b` | `i-077af8fdd52fd6b4a` | `10.20.1.10` | None |

The bastion public IP is the value recorded during testing. Public IPv4 addresses assigned dynamically can change after an instance is stopped and started, and an address can later be reassigned. Do not assume that the historical address remains allocated or belongs to this project.

---

## Routing Configuration

VPC Peering does not automatically add routes to VPC route tables. Routes were explicitly configured to direct traffic for the peer VPC's CIDR through the peering connection.

### Public route table — `ewb-rt-a-public`

| Destination | Target | Purpose |
|---|---|---|
| `10.10.0.0/16` | `local` | Communication within VPC A |
| `0.0.0.0/0` | `igw-0eb224e3fb0322445` | Internet routing for eligible public-subnet traffic |
| `10.20.0.0/16` | `pcx-03d90d239a4ae9a9b` | Traffic to VPC B |

### Private route table — `ewb-rt-a-private`

| Destination | Target | Purpose |
|---|---|---|
| `10.10.0.0/16` | `local` | Communication within VPC A |
| `10.20.0.0/16` | `pcx-03d90d239a4ae9a9b` | Traffic to VPC B |

### Private route table — `ewb-rt-b-private`

| Destination | Target | Purpose |
|---|---|---|
| `10.20.0.0/16` | `local` | Communication within VPC B |
| `10.10.0.0/16` | `pcx-03d90d239a4ae9a9b` | Traffic to VPC A |

The subnet associations were configured so that the public subnet used the public route table, while the private subnets used their respective private route tables.

**Routing principle:** The forward path and return path must both have valid routes. A correct route in only one VPC is not sufficient for bidirectional communication.

---

## Security Configuration

Security groups were configured to limit access to the required sources and protocols.

### Bastion — `ewb-sg-bastion`

| Protocol | Port | Source |
|---|---|---|
| TCP | 22 — SSH | Administrator's public IPv4 address with `/32` prefix |

The bastion was the only instance configured with a public IPv4 address.

### App-A — `ewb-sg-app-a`

| Protocol | Port / Type | Source |
|---|---|---|
| TCP | 22 — SSH | `10.10.1.10/32` |
| ICMPv4 | Echo Request | `10.20.1.10/32` |

### App-B — `ewb-sg-app-b`

| Protocol | Port / Type | Source |
|---|---|---|
| TCP | 22 — SSH | `10.10.1.10/32` |
| ICMPv4 | Echo Request | `10.10.2.10/32` |

The application security groups permit SSH from the bastion's private address and allow the specific ICMP Echo Request traffic required for the connectivity tests.

Security groups are stateful: responses to permitted connections are allowed by the security group without requiring an equivalent inbound rule for each response packet. Routing and other network controls must still permit the traffic.

---

## SSH Access Through the Bastion

OpenSSH `ProxyJump` was used to connect from the administrator workstation to the private instances through the bastion.

The repository's `configs/ssh_config` contains the sanitized host configuration. It references a private key stored locally on the administrator's workstation; the private key itself is not included in this repository.

### Example SSH commands

```powershell
ssh ewb-bastion
ssh ewb-app-a
ssh ewb-app-b
```

The private-instance host entries use the bastion as the jump host. A private EC2 connection therefore proceeds in two SSH stages:

1. The workstation establishes SSH to the bastion's public IPv4 address.
2. The bastion opens the onward SSH connection to the target's private IPv4 address.

For the onward connection, App-A and App-B see the bastion's private address, `10.10.1.10`, as the SSH source.

### Key-handling precautions

- Keep `ewb-lab.pem` on the local workstation.
- Restrict access to the private key file.
- Do not upload private keys, AWS credentials, passwords, or access tokens to GitHub.
- Verify the current bastion address before attempting to recreate the lab.
- The recorded host IPs and resource IDs are historical and should not be reused blindly in a new deployment.

---

## Testing and Results

The connectivity tests were executed while the instances, routes, security groups, and peering connection were configured for the lab.

### SSH validation

| Test | Recorded result |
|---|---|
| `ssh ewb-bastion` | Successful |
| `ssh ewb-app-a` through `ProxyJump` | Successful |
| `ssh ewb-app-b` through `ProxyJump` | Successful |

### Forward and reverse private connectivity

**Test 1 — App-A to App-B**

Executed on App-A:

```bash
ping -c 4 10.20.1.10
```

Recorded result:

```text
4 packets transmitted, 4 received, 0% packet loss
```

**Test 2 — App-B to App-A**

Executed on App-B:

```bash
ping -c 4 10.10.2.10
```

Recorded result:

```text
4 packets transmitted, 4 received, 0% packet loss
```

These results demonstrate successful private IPv4 connectivity in both directions during validation.

### Controlled negative test and recovery

The App-B inbound ICMP Echo Request rule from `10.10.2.10/32` was temporarily removed to demonstrate the effect of security-group filtering.

| Test | Recorded result |
|---|---|
| App-A ping after removing the App-B ICMP rule | 0 received, 100% packet loss |
| App-A ping after restoring the rule | 4 received, 0% packet loss |

The rule was restored after the controlled failure. The associated screenshots are included in `03_Evidence_Screenshots/`.

---

## Baseline Evidence Disclosure

The original pre-peering baseline ping failure was **not captured before the peering connection was activated**. It has not been fabricated retrospectively.

The evidence archive instead contains a separate controlled negative test performed after peering was configured: the App-B inbound ICMP Echo Request rule was removed, the ping failed, and connectivity was restored after the rule was added back.

This demonstrates the effect of the destination security-group rule on reachability. It is not presented as a substitute for the original pre-peering baseline test.

---

## Architecture and Networking Concepts

### 1. Why are return routes necessary?

IP communication is bidirectional. The destination VPC must know how to route response packets back to the source CIDR. The relevant route table in VPC B therefore needs a route for `10.10.0.0/16` through the peering connection, while VPC A needs the corresponding route for `10.20.0.0/16`.

### 2. Why does App-B see the bastion's private IP during SSH?

With `ProxyJump`, the workstation connects to the bastion first. The bastion then establishes a separate SSH connection to App-B. Consequently, App-B sees the onward connection originating from the bastion's private IP, `10.10.1.10`, rather than the workstation's public IP.

### 3. Why does `Active` peering not guarantee successful ping?

`Active` indicates that the peering connection is available. It does not automatically configure routes or security rules. Both VPCs still need appropriate routes, the security groups and network ACLs must permit the traffic, and the instances and host networking must be functioning.

### 4. Why cannot VPC B automatically use VPC A's Internet Gateway?

VPC Peering is non-transitive. It does not provide a general transit path through VPC A's Internet Gateway. VPC B would require an explicitly designed and supported egress architecture if Internet access were needed.

---

## Cleanup and Cost Considerations

After validation, the supplied cleanup evidence shows:

- All three project EC2 instances in the `Terminated` state.
- The project VPCs absent from the displayed VPC list, with the default VPC remaining.

The evidence archive does not contain post-cleanup screenshots for EBS volumes, snapshots, Elastic IP allocations, or the AWS Billing pages. Therefore, this repository does **not** claim that every possible chargeable resource has been removed or that the account's billing balance is zero.

For a complete cost review, verify the following in the AWS Console, particularly in `ap-south-1`:

1. **EC2 → Volumes:** check for unattached EBS volumes belonging to the lab.
2. **EC2 → Snapshots:** check for lab-created snapshots that are no longer needed. Do not delete unrelated or AMI-owned snapshots without verifying ownership and dependencies.
3. **EC2 → Elastic IPs:** check for unused addresses and release only those confirmed to belong to this lab.
4. **EC2 → Instances:** confirm that the three project instances are terminated.
5. **VPC resources:** confirm that the project VPCs, peering connection, custom route tables, subnets, security groups, and Internet Gateway have been removed where applicable.
6. **Billing and Cost Management:** review usage and charges. Billing data may take time to update, and terminating the EC2 instances does not retroactively remove charges already incurred.

Do not delete unrelated pre-existing practice resources.

---

## Repository Structure

```text
EWB_VPC_Peering_Submission/
├── README.md
├── configs/
│   └── ssh_config
├── 01_Report/
│   ├── Tanay_Jain_EWB_VPC_Peering.docx
│   └── Tanay_Jain_EWB_VPC_Peering.pdf
├── 02_Architecture/
│   └── EWB_VPC_Peering_Architecture.jpg
└── 03_Evidence_Screenshots/
    ├── 01-ec2-instances.PNG
    ├── 02-vpcs-cidrs.PNG
    ├── 03-subnets-cidrs.PNG
    ├── 04-peering-active.PNG
    ├── 05-route-a-public.PNG
    ├── 06-route-a-private.PNG
    ├── 07-route-b-private.PNG
    ├── 08-association-a-public.PNG
    ├── 09-association-a-private.PNG
    ├── 10-association-b-private.PNG
    ├── 11-sg-bastion.PNG
    ├── 12-sg-app-a.PNG
    ├── 13-sg-app-b.PNG
    ├── 14-ssh-bastion.PNG
    ├── 15-ssh-app-a.PNG
    ├── 16-ssh-app-b.PNG
    ├── 17-ping-app-a-to-app-b.PNG
    ├── 18-ping-app-b-to-app-a.PNG
    ├── 19-controlled-ping-failure.PNG
    ├── 20-controlled-ping-recovery.PNG
    ├── 21-cleanup-evidence.PNG
    └── 21-cleanup-evidence-2.PNG
```

The report is the consolidated project deliverable. The architecture image and evidence screenshots provide supporting documentation for the recorded configuration and results.

---

## Reproducing the Lab

This repository documents a completed and cleaned-up lab; it does not deploy AWS resources automatically.

To reproduce the project in an AWS account:

1. Review the official lab instructions and confirm the intended Region.
2. Create VPC A and VPC B with the CIDRs described above.
3. Create the three subnets and associate each with the appropriate route table.
4. Create and attach the Internet Gateway to VPC A.
5. Create the bastion and two private EC2 instances using the required private IPs, security groups, and key pair.
6. Create the VPC Peering connection and accept it.
7. Add the three peering routes and verify their targets.
8. Configure a local OpenSSH key and sanitized `ProxyJump` host entries.
9. Validate SSH access, both ping directions, and the controlled security-group failure/recovery test.
10. Capture fresh evidence and clean up the resources after the exercise.

**Cost warning:** AWS resources can incur charges while running or allocated. Check current pricing before deployment, keep the lab running only as long as needed, and verify cleanup afterwards. The exact charges depend on the resources, region, duration, and applicable pricing.

---

## Learning Outcomes

This project provided practical experience with:

- IPv4 CIDR planning and subnet design.
- VPC Peering between non-overlapping networks.
- Explicit route-table configuration and return routing.
- Internet Gateway routing for a public subnet.
- Restrictive stateful security-group rules.
- Private EC2 administration using a bastion host and OpenSSH `ProxyJump`.
- Bidirectional ICMP connectivity validation.
- Controlled security-group failure and recovery.
- Evidence capture, technical reporting, and AWS resource cleanup.

## Author

**Tanay Jain**  
BCA Student — Birla Institute of Technology, Mesra (Jaipur Campus)

This project was completed as part of the EWB Edutech Pvt. Ltd. AWS Cloud Computing Internship.

---

*Documentation reflects the recorded lab configuration and supplied evidence. Historical AWS identifiers and IP addresses are included for traceability; the original project resources have been cleaned up and should not be assumed to remain available.*
