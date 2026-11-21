# AWS Web Server in a Custom VPC + 3 Troubleshooting Tickets

A hands-on AWS project built on the Free plan (no charges). I built a custom network and a web server, then broke it on purpose three ways and fixed each one, writing a support ticket for every issue.

**Skills shown:** VPC design, security groups, route tables, network ACLs, EC2, RDS (private), Linux/nginx, troubleshooting, written communication.

## Architecture

- 1 custom VPC (`10.0.0.0/16`) in Mumbai (`ap-south-1`)
- 2 Availability Zones, each with 1 public and 1 private subnet
- Internet gateway, and **no NAT gateway** (to stay inside free credits)
- EC2 `t3.micro` (Amazon Linux 2023) running nginx in a public subnet
- RDS MySQL `db.t3.micro` in private subnets, public access **off**
- Security groups: `web-sg` (HTTP 80, SSH 22) and `db-sg` (MySQL 3306 only from `web-sg`)


## Screenshots

All screenshots (VPC map, working page, and the before/after of each ticket) are in the [`docs/images`](docs/images) folder.

## Security design

- The database has no public IP and accepts traffic only from the web server's security group.
- SSH is limited to my own IP.
- Resources were deleted or stopped after each session to avoid spending credits.

## Troubleshooting tickets

### Ticket 1: Website times out (security group)

| | |
|---|---|
| **Symptom** | Browser showed `ERR_CONNECTION_TIMED_OUT`. EC2 was Running, 3/3 checks passed. |
| **How I checked** | Confirmed the instance was healthy, then opened `web-sg` inbound rules. Only SSH (22) was listed. |
| **Root cause** | The HTTP (80) inbound rule was missing, so the security group blocked web traffic. |
| **Fix** | Added an inbound HTTP rule on port 80 from `0.0.0.0/0`. |
| **Prevention** | Review security group rules after every change. Note: security groups do not cut connections that are already open, so test in a fresh Incognito window. |


### Ticket 2: Website times out (route table)

| | |
|---|---|
| **Symptom** | Site timed out again even though `web-sg` allowed HTTP. |
| **How I checked** | Ruled out the instance and `web-sg`, then opened the public subnet's route table. |
| **Root cause** | The `0.0.0.0/0` route to the internet gateway was missing. |
| **Fix** | Added a route: `0.0.0.0/0` to `althaf-ha-app-igw`. |
| **Prevention** | Check route tables after any network change. |


### Ticket 3: Website times out (network ACL)

| | |
|---|---|
| **Symptom** | Site timed out while the security group and route table were both correct. |
| **How I checked** | Worked down the network path: instance, security group, route table, then the VPC network ACL. |
| **Root cause** | An inbound Deny rule (rule 50) for port 80 in the network ACL. |
| **Fix** | Removed the Deny rule. The default Allow rule (100) stayed in place. |
| **Prevention** | Keep NACLs at the default and track every NACL change. |


## What I learned

- Network problems follow an order: instance, security group, route table, network ACL.
- Security groups are stateful, and network ACLs are stateless with rule numbers.
- Free-plan limits push good habits: no NAT gateway, single-AZ database, and cleanup after every session.

## Cost control

- Zero-spend AWS Budget alert set up first.
- No NAT gateway, no Elastic IP, no Multi-AZ.
- RDS deleted (no snapshot) and EC2 stopped after each session.

## Author

Althafali S | AWS Certified Solutions Architect – Associate (SAA-C03) | github.com/AlthafAli05
