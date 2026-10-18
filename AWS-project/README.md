# AWS Assignment 2: Application Load Balancer

**Objective:** Deploy two EC2 web servers behind an Application Load Balancer (ALB). The ALB handles all incoming traffic, and the instances cannot be reached directly from the internet.

## Architecture

```
Internet
   │  HTTP :80
   ▼
Application Load Balancer (internet-facing, ALB security group)
   │  forwards to target group, runs health checks
   ▼
Target group
   ├── EC2 instance 1 (Apache, page shows its hostname)
   └── EC2 instance 2 (Apache, page shows its hostname)
       EC2 security group: HTTP allowed only from the ALB security group
```

| Security group | Inbound rule                     | Purpose                                  |
|----------------|----------------------------------|------------------------------------------|
| ALB SG         | HTTP (80) from `0.0.0.0/0`       | Accept web traffic from anyone           |
| EC2 SG         | HTTP (80) from the ALB SG only   | Block direct access to the instances     |

## Build steps

### 1. Launch two EC2 instances

Both instances run in the same VPC and serve a simple web page, so I can see which one the ALB sent each request to.

<img width="1353" height="325" alt="Two EC2 instances running" src="https://github.com/user-attachments/assets/2b5bdc93-8160-4de8-9fd1-8f515b0d8933" />

**VPC:**

<img width="1361" height="360" alt="VPC used for both instances" src="https://github.com/user-attachments/assets/938f19be-1be5-4853-aabf-b306153252da" />

**Subnets:**

<img width="1413" height="176" alt="Subnets in the VPC" src="https://github.com/user-attachments/assets/5decf871-b015-4b0c-b0bb-3cdaa48b81bf" />

### 2. Install a web server with user data

I used this script as user data for both instances. It installs Apache and writes a page showing the instance's hostname, so each server's page is different.

```bash
#!/bin/bash
yum update -y
yum install -y httpd
systemctl enable --now httpd
echo "<h1>Welcome to $(hostname -f)</h1>" > /var/www/html/index.html
```

### 3. Create the target group

A target group is the set of destinations the ALB routes requests to. Here, it contains the two EC2 instances.

<img width="776" height="533" alt="Target group settings" src="https://github.com/user-attachments/assets/3f7b6de8-c3e1-489c-bf51-ba52c85f29df" />

**Registered targets:**

<img width="1316" height="353" alt="EC2 instances registered as targets" src="https://github.com/user-attachments/assets/7c56ae61-a03b-4c65-a81c-3bfcd6a29bcc" />

**Result:**

<img width="1390" height="675" alt="Target group created" src="https://github.com/user-attachments/assets/a0b4a7b5-bef0-4441-aeaf-36e77bf30d77" />

### 4. Create the Application Load Balancer

AWS offers two ALB schemes:

- **Internet-facing:** routes requests from the internet to resources inside the VPC.
- **Internal:** routes requests only within the VPC.

I chose **internet-facing** because the web servers need to accept requests from the internet. An internet-facing ALB must sit in public subnets, so the VPC needs an Internet Gateway and a route table that sends `0.0.0.0/0` to it.

### 5. Configure the security groups

**ALB security group:** allows HTTP from anywhere.

<img width="1596" height="490" alt="ALB security group allowing HTTP from anywhere" src="https://github.com/user-attachments/assets/303d236a-a350-47dd-b567-5f3afaa32f74" />

**EC2 security group:** allows HTTP only when the source is the ALB security group. Requests sent straight to an instance's IP are dropped.

<img width="1328" height="194" alt="EC2 security group allowing HTTP only from the ALB security group" src="https://github.com/user-attachments/assets/978a73b8-1324-4188-90f7-9e1482f5dc21" />

**Completed ALB:**

<img width="1334" height="271" alt="Application Load Balancer active" src="https://github.com/user-attachments/assets/cfb2b292-abb6-4822-b0c1-02a4e27681af" />

## Result

Refreshing the ALB's DNS name alternates between the two instances' hostnames, which shows the ALB is spreading requests across both instances.

### Demo

https://github.com/user-attachments/assets/4938907d-682c-47bc-9cf6-c0d542b06b14

### Health checks

Both targets pass the ALB health checks. If one instance failed, the ALB would stop sending it traffic.

<img width="1592" height="692" alt="Both targets healthy in the target group" src="https://github.com/user-attachments/assets/c050dca1-b362-4ca7-9943-c3371eecf8db" />
