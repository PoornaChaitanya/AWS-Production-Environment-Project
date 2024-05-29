# AWS-Production-Environment-Project

The following diagram provides an overview of the resources included in this example. I created a VPC with public subnets, private subnets, and a bastion server in multiple Availability Zones. Each public subnet contains a NAT gateway and a load balancer node. The servers run in the private subnets, are launched and terminated by using an Auto Scaling group, and receive traffic from the load balancer. The servers can connect to the internet by using the NAT gateway. The servers can connect to Amazon S3 by using a gateway VPC endpoint. The bastion server provides secure access to the instances in the private subnets.
![1699781752576](https://github.com/PoornaChaitanya/AWS-Production-Environment-Project/assets/84367538/360cee37-03fd-4ac8-95f0-b397f3df630a)

## VPC Structure

- **Public Subnets**: Contain NAT gateway, load balancer node, and bastion server.
- **Private Subnets**: Contain servers launched by an Auto Scaling group.
- **Connectivity**:
  - Servers to Internet: Through NAT gateway.
  - Servers to Amazon S3: Through gateway VPC endpoint.
  - Secure Access: Through bastion server.

## Routing

When I created this VPC using the Amazon VPC console, a route table was created for the public subnets with local routes and routes to the internet gateway. Additionally, a route table was created for the private subnets with local routes, routes to the NAT gateway, egress-only internet gateway, and gateway VPC endpoint.

### Public Subnet Route Table Example

| Destination            | Target     |
|------------------------|------------|
| 10.0.0.0/16            | local      |
| 2001:db8:1234:1a00::/56 | local      |
| 0.0.0.0/0              | igw-id     |
| ::/0                   | igw-id     |

### Private Subnet Route Table Example

| Destination            | Target           |
|------------------------|------------------|
| 10.0.0.0/16            | local            |
| 2001:db8:1234:1a00::/56 | local            |
| 0.0.0.0/0              | nat-gateway-id   |
| ::/0                   | eigw-id          |
| s3-prefix-list-id      | s3-gateway-id    |

## Security

Example security group rules for servers and bastion server:

### Inbound Rules for Servers

| Source                           | Protocol         | Port Range     | Comments                                                  |
|----------------------------------|------------------|----------------|-----------------------------------------------------------|
| ID of the load balancer security group | listener protocol | listener port | Allows inbound traffic from the load balancer on the listener port |
| ID of the load balancer security group | health check protocol | health check port | Allows inbound health check traffic from the load balancer |

### Inbound Rules for Bastion Server

| Source              | Protocol | Port Range | Comments                                         |
|---------------------|----------|------------|--------------------------------------------------|
| Your IP (CIDR)      | SSH      | 22         | Allows SSH access from your IP address           |
| Private Subnet CIDR | SSH      | 22         | Allows SSH access to instances in private subnets |

## Create the VPC

### Steps to Create the VPC

1. I opened the Amazon VPC console at [https://console.aws.amazon.com/vpc/](https://console.aws.amazon.com/vpc/).
2. On the dashboard, I chose **Create VPC**.
3. For **Resources to create**, I chose **VPC and more**.

### Configure the VPC

1. For **Name tag auto-generation**, I entered a name for the VPC.
2. For **IPv4 CIDR block**, I kept the default suggestion or entered the required CIDR block.
3. If using IPv6, I chose **IPv6 CIDR block**, Amazon-provided IPv6 CIDR block.

### Configure the Subnets

1. For **Number of Availability Zones**, I chose 2 or more to improve resiliency.
2. For **Number of public subnets**, I chose 2 or more.
3. For **Number of private subnets**, I chose 2 or more.
4. For NAT gateways, I chose 1 per AZ.

### DNS Options

- I cleared **Enable DNS hostnames**.

### Create the VPC

- I chose **Create VPC**.
![1699781750890](https://github.com/PoornaChaitanya/AWS-Production-Environment-Project/assets/84367538/82d77ac5-7fb8-447b-9b38-72dda562299c)


## Deploy Your Application

- I used Amazon EC2 Auto Scaling to deploy servers in multiple Availability Zones.

### Launch Instances Using Auto Scaling

1. I created a launch template to specify the EC2 instances configuration.
2. I created an Auto Scaling group.
3. I created a load balancer and attached it to the Auto Scaling group.
![1699781750967](https://github.com/PoornaChaitanya/AWS-Production-Environment-Project/assets/84367538/04c19882-c909-4346-b226-22017f5293b6)


### Bastion Server Deployment

1. I launched an EC2 instance to act as the bastion server in one of the public subnets.
2. I ensured the bastion server had a security group that allows SSH access from my IP and access to the private subnet instances.

## Simple Python Page Deployment

I launched a simple Python page on the servers deployed in the private subnets. 

![1699781750804](https://github.com/PoornaChaitanya/AWS-Production-Environment-Project/assets/84367538/7eebe70d-c0ea-485a-97ff-17a70507c24b)

