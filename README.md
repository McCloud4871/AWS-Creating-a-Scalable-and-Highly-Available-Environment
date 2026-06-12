# AWS Auto Scaling Web Applications with High Availability

## Project Overview

This project focused on improving the availability, scalability, and resilience of a café web application hosted on AWS. The environment was redesigned to use multiple Availability Zones, private and public subnets, an Application Load Balancer (ALB), a NAT Gateway, and an Auto Scaling Group (ASG). The final solution automatically distributed traffic across multiple web servers and dynamically scaled resources in response to increased demand.

## Skills Demonstrated

- AWS Architecture Design
- High Availability Configuration
- Auto Scaling Configuration
- Application Load Balancer Deployment
- Amazon EC2 Administration
- VPC Networking
- Public and Private Subnet Design
- NAT Gateway Configuration
- Route Table Management
- AWS Systems Manager Session Manager
- Infrastructure Testing and Validation
- Cloud Security Best Practices

## Technology Stack

| Service | Purpose |
|----------|----------|
| Amazon EC2 | Hosted the café web application |
| Amazon VPC | Network isolation and segmentation |
| Public Subnets | Hosted internet-facing resources |
| Private Subnets | Hosted application servers |
| NAT Gateway | Provided internet access for private instances |
| Application Load Balancer | Distributed traffic across web servers |
| Auto Scaling Group | Automatically adjusted server capacity |
| Target Groups | Routed traffic to healthy instances |
| Security Groups | Controlled inbound and outbound traffic |
| AWS Systems Manager | Remote administration and testing |

---

## Solution Architecture

The environment was designed using a multi-tier architecture.

- Application servers were deployed in private subnets.
- An Application Load Balancer was deployed in public subnets.
- A NAT Gateway provided outbound internet access for private instances.
- Auto Scaling automatically increased or decreased server capacity based on CPU utilization.
- Traffic was distributed across multiple Availability Zones to improve resilience and fault tolerance.

[screenshot – architecture overview]<img width="1147" height="629" alt="Screenshot 2026-06-12 134647" src="https://github.com/user-attachments/assets/67586ba8-185f-46d0-8a40-6df2a85e57f5" />


---

## Task 1: Environment Review

The existing café environment was reviewed to identify the current infrastructure and networking configuration.

Key components identified:

- Existing Café Web Application Server
- Custom AMI used for deployment
- Existing VPC configuration
- Public and private subnets
- Existing security groups and IAM roles

[screenshot – original EC2 instance]<img width="1642" height="813" alt="Screenshot 2026-06-12 145109" src="https://github.com/user-attachments/assets/62230ea3-8187-4816-a25d-138365622804" />


---

## Task 2: Configuring Network Infrastructure

### NAT Gateway Deployment

A NAT Gateway was deployed within a public subnet to allow private instances to access the internet for software updates and package installations.

Configuration included:

- Elastic IP allocation
- NAT Gateway deployment in Public Subnet 2
- Route table updates for private subnets

[screenshot – NAT Gateway configuration]<img width="1863" height="576" alt="Screenshot 2026-06-12 151646" src="https://github.com/user-attachments/assets/856ec5b3-381a-457f-a306-ac3aade887a5" />


### Route Table Updates

Private route tables were modified to direct internet-bound traffic through the NAT Gateway.

Configuration:

- Destination: 0.0.0.0/0
- Target: NAT Gateway

[screenshot – updated route table]<img width="1878" height="582" alt="Screenshot 2026-06-12 153044" src="https://github.com/user-attachments/assets/515e1574-fd49-4fd3-8295-cea61d8c9d4e" />


---

## Task 3: Launch Template Creation

A launch template was created from the existing café web server configuration.

Configuration included:

- Existing Café Web Server AMI
- t2.micro instance type
- Existing IAM role
- Existing Café security group
- Key pair configuration

This template provided a consistent deployment standard for Auto Scaling.

[screenshot – launch template details]<img width="1889" height="848" alt="Screenshot 2026-06-12 154932" src="https://github.com/user-attachments/assets/040035f6-967b-4bee-8c06-226cea051d9f" />


---

## Task 4: Auto Scaling Group Deployment

An Auto Scaling Group was created using the launch template.

Configuration:

| Setting | Value |
|----------|----------|
| Minimum Capacity | 2 |
| Desired Capacity | 2 |
| Maximum Capacity | 6 |
| Availability Zones | us-east-1a, us-east-1b |

The Auto Scaling Group distributed instances across multiple Availability Zones to improve availability and fault tolerance.

[screenshot – Auto Scaling Group configuration]<img width="940" height="431" alt="image" src="https://github.com/user-attachments/assets/43407749-cf00-463d-b36e-0e2a2ba96ae8" />


### Scaling Policy Configuration

A Target Tracking Scaling Policy was configured.

| Setting | Value |
|----------|----------|
| Metric | Average CPU Utilization |
| Target Value | 25% |
| Warmup Period | 60 seconds |

This policy automatically adjusted capacity based on server load.

<img width="813" height="526" alt="image" src="https://github.com/user-attachments/assets/53f65b6a-5919-40b9-b88b-b37d328e809d" />




---

## Task 5: Application Load Balancer Deployment

An internet-facing Application Load Balancer was deployed.

Configuration:

| Setting | Value |
|----------|----------|
| Type | Application Load Balancer |
| Scheme | Internet-facing |
| Protocol | HTTP |
| Port | 80 |
| Availability Zones | Public Subnet 1, Public Subnet 2 |

[screenshot – ALB configuration]<img width="940" height="342" alt="image" src="https://github.com/user-attachments/assets/cd3c5e17-79ba-45bb-b74b-8f238a573014" />


### Security Group Configuration

A dedicated security group was created for the load balancer.

Inbound Rules:

| Protocol | Port | Source |
|----------|----------|----------|
| HTTP | 80 | 0.0.0.0/0 |

[screenshot – ALB security group]<img width="940" height="308" alt="image" src="https://github.com/user-attachments/assets/182cc4c8-18c9-4597-97af-be09cd0162bf" />


### Target Group Configuration

A target group was created and attached to the load balancer.

Configuration:

| Setting | Value |
|----------|----------|
| Protocol | HTTP |
| Port | 80 |
| Target Type | Instance |

The Auto Scaling Group was then attached to the target group.

<img width="940" height="452" alt="image" src="https://github.com/user-attachments/assets/52ecc2ea-86ef-41f1-95ba-1ce892e60804" />


---

## Task 6: Validation and Testing

### Application Load Testing

The application was tested through the load balancer DNS endpoint.

The café application was successfully accessible through:

```text
http://<ALB-DNS-Name>/cafe
```

This confirmed:

- Load balancer functionality
- Target group registration
- Application availability

<img width="940" height="767" alt="image" src="https://github.com/user-attachments/assets/2e1b94fe-1ff1-40bd-a0fb-f8a39ff43aed" />


---

### Auto Scaling Validation

AWS Systems Manager Session Manager was used to connect to a running web server instance.

A CPU stress test was executed:

```bash
sudo amazon-linux-extras install epel -y
sudo yum install stress -y
stress --cpu 1 --timeout 600
```

The increased CPU utilization triggered the Auto Scaling policy.

Results:

- Initial capacity: 2 instances
- Maximum capacity reached: 6 instances
- New instances launched automatically
- All instances successfully passed health checks

<img width="940" height="766" alt="image" src="https://github.com/user-attachments/assets/2904e200-11e0-4880-97c0-a935f34e6699" />



<img width="940" height="425" alt="image" src="https://github.com/user-attachments/assets/d3335936-aa29-406f-95ae-a43866a31c27" />


---

## Results

The café application environment was successfully transformed into a scalable and highly available architecture.

Key outcomes:

- High availability across multiple Availability Zones
- Load-balanced web traffic
- Automated capacity management
- Improved resilience and fault tolerance
- Secure deployment using private application subnets
- Successful validation of automatic scaling under load

---

## Key Learnings

- Designing highly available AWS architectures
- Implementing Auto Scaling Groups
- Configuring Application Load Balancers
- Managing private subnet internet access using NAT Gateways
- Configuring route tables and network segmentation
- Monitoring scaling behaviour under load
- Validating cloud infrastructure performance and resilience
