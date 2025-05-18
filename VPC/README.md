# AWS VPC and Networking Components: Detailed Technical Guide with Real-Time Scenario

---

## 1. What is a VPC?

A **Virtual Private Cloud (VPC)** is a logically isolated virtual network in AWS. It lets you launch AWS resources in a virtual network defined by you, controlling IP address ranges, subnets, route tables, gateways, and security settings.

---

## 2. Key VPC Components and Their Roles

| Component                  | Description                                                                                      | Real-Time Use Case Example                                                                                     |
| -------------------------- | ------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------- |
| **Subnet**                 | Segments within a VPC's IP range; can be public or private depending on routing.                 | Public subnet (`10.0.1.0/24`) hosts internet-facing ALB; private subnet (`10.0.2.0/24`) hosts app servers.     |
| **Route Table**            | Directs network traffic between subnets and external networks.                                   | Private subnet’s route table sends internet-bound traffic to NAT Gateway; public subnet’s to Internet Gateway. |
| **Internet Gateway (IGW)** | Enables communication between instances in the VPC and the internet.                             | ALB in public subnet uses IGW to serve user requests from internet.                                            |
| **NAT Gateway**            | Allows instances in private subnets to initiate outbound internet connections securely.          | Backend servers in private subnet use NAT Gateway to download patches or call external APIs.                   |
| **VPC Peering**            | Private network connection between two VPCs allowing traffic with private IPs.                   | Dev VPC peers with Logging VPC to securely send logs without internet exposure.                                |
| **Transit Gateway**        | Central router that connects multiple VPCs and on-premises networks allowing transitive routing. | Multiple VPCs in an enterprise connect via Transit Gateway for simplified network management.                  |
| **Network Firewall**       | Managed firewall to monitor and control inbound/outbound traffic at VPC boundary.                | PCI-compliant app inspects all outbound traffic for threats using AWS Network Firewall.                        |
| **Security Group (SG)**    | Instance-level, stateful firewall controlling inbound and outbound traffic.                      | App servers’ SG only allows inbound HTTP/HTTPS traffic from the ALB SG.                                        |
| **Network ACL (NACL)**     | Stateless subnet-level firewall that applies allow/deny rules for inbound/outbound traffic.      | Public subnet’s NACL blocks known malicious IP addresses and restricts traffic rates to prevent DoS attacks.   |

---

## 3. Private IP Address Classes (RFC 1918)

| Class | CIDR Range       | IP Range                      | Default Mask | Number of Hosts | Use Case                           |
| ----- | ---------------- | ----------------------------- | ------------ | --------------- | ---------------------------------- |
| **A** | `10.0.0.0/8`     | 10.0.0.0 – 10.255.255.255     | 255.0.0.0    | \~16 million    | Large corporate networks, AWS VPCs |
| **B** | `172.16.0.0/12`  | 172.16.0.0 – 172.31.255.255   | 255.240.0.0  | \~1 million     | Medium-sized networks              |
| **C** | `192.168.0.0/16` | 192.168.0.0 – 192.168.255.255 | 255.255.0.0  | \~65,000        | Small home or office networks      |

**Best practice:** AWS VPCs commonly use Class A (`10.0.0.0/16`) for flexibility and scalability.

---

## 4. Real-Time Example Scenario: Deploying a Secure 3-Tier Web Application

### Step 1: **Design the VPC**

* Create a VPC with CIDR: `10.0.0.0/16` (Class A private IP range).
* This allows up to 65,536 IPs for subnetting and growth.

### Step 2: **Create Subnets**

* Public Subnet: `10.0.1.0/24` (hosts ALB, NAT Gateway).
* Private Subnet: `10.0.2.0/24` (hosts backend app servers).
* Database Subnet: `10.0.3.0/24` (hosts RDS MySQL instance).

### Step 3: **Set Up Internet Gateway**

* Attach IGW to VPC.
* Associate public subnet route table to route `0.0.0.0/0` → IGW.

### Step 4: **Set Up NAT Gateway**

* Place NAT Gateway in public subnet.
* Private subnet route table routes `0.0.0.0/0` → NAT Gateway.

### Step 5: **Configure Route Tables**

* Public Subnet Route Table:

  * Local route for `10.0.0.0/16`
  * Default route `0.0.0.0/0` → IGW
* Private Subnet Route Table:

  * Local route for `10.0.0.0/16`
  * Default route `0.0.0.0/0` → NAT Gateway

### Step 6: **Configure Security Groups**

* ALB SG: Allow inbound HTTPS (443) from anywhere.
* App Server SG: Allow inbound HTTP (port 8080) only from ALB SG.
* RDS SG: Allow inbound MySQL (3306) only from App Server SG.

### Step 7: **Implement Network ACL**

* Public subnet NACL: Allow HTTP/HTTPS inbound/outbound; deny blacklisted IP ranges.
* Private subnet NACL: Restrictive rules, only allow required ports.

### Step 8: **Set Up VPC Peering**

* Peer this VPC with a Logging VPC (`172.16.0.0/16`) for secure log aggregation without internet exposure.

### Step 9: **Use Transit Gateway**

* Connect multiple VPCs and on-premises networks via a Transit Gateway for central routing and management.

### Step 10: **Deploy Network Firewall**

* Deploy AWS Network Firewall in dedicated subnet to monitor outbound traffic for threat detection.

### Step 11: **Configure Private Hosted Zones**

* Create Route 53 Private Hosted Zone: `db.internal.example.com`
* Maps to RDS private IP (`10.0.3.x`), hidden from the public internet.

---

## 5. Summary Diagram (Conceptual)

```
[Internet]
    ↓
[Internet Gateway]
    ↓
[Public Subnet 10.0.1.0/24] -- ALB, NAT Gateway
    ↓
[Private Subnet 10.0.2.0/24] -- Application Servers (ECS/EC2)
    ↓
[Database Subnet 10.0.3.0/24] -- RDS MySQL
```

---

## Why This Matters

* **Security:** Private subnets isolate backend servers and databases from direct internet access.
* **Scalability:** Using large CIDR (Class A) enables multiple subnets and VPC expansions.
* **Connectivity:** Peering and Transit Gateway enable communication between VPCs and on-premises networks.
* **Management:** Route tables, SGs, NACLs, and firewalls allow precise traffic control and monitoring.

