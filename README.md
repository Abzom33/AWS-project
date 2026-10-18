# AWS-project



# AWS Assignment 1: VPC and Networking

**Objective:** Build a custom VPC with one public and one private subnet, configure routing so both can reach the internet, and deploy an EC2 instance in each.

## Architecture

```
VPC (10.0.0.0/16)
├── Public subnet   route 0.0.0.0/0 -> Internet Gateway
│   ├── EC2 instance (bastion host)
│   └── NAT Gateway
└── Private subnet  route 0.0.0.0/0 -> NAT Gateway
    └── EC2 instance (no public IP, reached via the bastion)
```

| Subnet  | Default route (0.0.0.0/0) | Inbound access                       |
|---------|---------------------------|--------------------------------------|
| Public  | Internet Gateway          | SSH/HTTP from my IP only             |
| Private | NAT Gateway               | Internal VPC traffic only            |

## Build steps

### 1. Create the VPC

I created a custom VPC with the IPv4 CIDR block `10.0.0.0/16` and left the other settings at their defaults.

<img width="1593" height="575" alt="Creating the custom VPC" src="https://github.com/user-attachments/assets/aa0f55cb-6180-4bca-ace7-43486235ec70" />

### 2. Create the subnets

A subnet is public when its route table sends traffic to an Internet Gateway (IGW). Without that route it is private.

1. Click **Create subnet**.
2. Select the custom VPC.
3. Name the subnet.
4. Pick an IPv4 CIDR block. It must not overlap with any other subnet in the VPC.
5. Choose an Availability Zone.
6. Repeat for the second subnet.

<img width="1105" height="723" alt="Public and private subnets" src="https://github.com/user-attachments/assets/eb463793-7f39-412b-a1a8-7ecc418d0fc0" />

### 3. Create the security groups

**Public EC2 security group:** allows SSH and HTTP from my IP only.

<img width="1599" height="349" alt="Public security group rules" src="https://github.com/user-attachments/assets/1e8d4a3a-422b-4420-934a-394923387131" />

**Private EC2 security group:** allows internal traffic only.

<img width="1596" height="579" alt="Private security group rules" src="https://github.com/user-attachments/assets/9d29c4eb-9245-456d-8cba-a7150c4b58b5" />

### 4. Launch the public EC2 instance

1. Click **Launch instance**.
2. Give it a name.
3. Keep the default AMI and instance type.
4. Under **Network settings**, click **Edit**.
5. Set the VPC to the custom VPC and the subnet to the public subnet.
6. Launch.

<img width="1588" height="703" alt="Public EC2 instance network settings" src="https://github.com/user-attachments/assets/92faa77e-b4fe-4526-afe7-5f58854ab53c" />

### 5. Create the Internet Gateway

The IGW lets resources in the VPC reach the internet.

1. Click **Create internet gateway**.
2. Attach it to the custom VPC.

<img width="1596" height="343" alt="Internet Gateway attached to the VPC" src="https://github.com/user-attachments/assets/09c5a5af-8865-4be1-9f9f-966caecc41dc" />

### 6. Configure the public route table

1. Click **Create route table**, name it, and attach it to the custom VPC.

<img width="1599" height="675" alt="Creating the public route table" src="https://github.com/user-attachments/assets/3be87165-9e29-47e3-8e9a-f4f8988056df" />

2. Add a route for `0.0.0.0/0` with the IGW as the target.

<img width="1252" height="194" alt="Route to the Internet Gateway" src="https://github.com/user-attachments/assets/05cbcc6f-1a23-49ff-87ee-4f36c900b8e0" />

3. Explicitly associate the public subnet with this route table. By default, every subnet uses the main route table created with the VPC, which has no route to the IGW.

<img width="1342" height="276" alt="Public subnet association" src="https://github.com/user-attachments/assets/2d9b61b8-4d3b-46cb-95c6-8e9b492ea377" />

**Result:** the public instance can now reach the internet through the IGW.

<img width="1597" height="712" alt="Public instance with internet access" src="https://github.com/user-attachments/assets/b189a850-e1cc-49bd-8f2c-77ff5ba51f53" />

### 7. Set up the private subnet

1. Launch an EC2 instance in the private subnet.

<img width="1324" height="41" alt="Private EC2 instance" src="https://github.com/user-attachments/assets/9c8a5dab-7a62-4bb2-8ebe-3d1debeae1d0" />

2. Create a NAT Gateway in the public subnet, then add a `0.0.0.0/0` route to the NAT Gateway in the private subnet's route table.

<img width="1599" height="473" alt="NAT Gateway route for the private subnet" src="https://github.com/user-attachments/assets/8ca6f969-c309-440c-bf6c-3bd29f0e919b" />

## Result

The public instance acts as a bastion host. To reach the private instance, I SSH into the bastion first and then SSH from there into the private instance. Both instances have outbound internet access: the public one through the IGW and the private one through the NAT Gateway.

### Demo

https://github.com/user-attachments/assets/d9d3f88f-fc9a-4e27-8abd-e58957cd3910

## Problems I ran into

### EC2 Instance Connect could not reach the public instance

<img width="1597" height="712" alt="Connection attempt" src="https://github.com/user-attachments/assets/b189a850-e1cc-49bd-8f2c-77ff5ba51f53" />

**Cause:** I first tried connecting through EC2 Instance Connect in the AWS console. Those connections come from AWS's own IP range, not from my machine, so the security group rule allowing SSH only from my IP blocked them.

**Fix:** I connected over SSH from my own machine using the key pair, which matches the security group rule.

<img width="810" height="426" alt="Successful SSH connection from my machine" src="https://github.com/user-attachments/assets/cd2bbbb8-85c7-4806-baa2-3b3eb6698e89" />

