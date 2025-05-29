## 🚀 3 TIER APPLICATION 🚀
This repository is created to learn and deploy Resilient Three-Tier Architecture web app on AWS.
### Prerequisites
📌 AWS account

📌 Basic knowledge of Linux

## ⚙️List of AWS services Used

-🌍 Amazon CloudFront

-🌐 Amazon Route 53

-💻 Amazon EC2

-⚖️ Amazon Autoscaling

-🪪 Amazon Certificate Manager

-🪣 Amazon Backup service

-🗄️ Amazon RDS

-☁️ Amazon VPC

-🔐 Amazon WAF

-👁️ Amazon CloudWatch

## 🤔 What is Three-tire architecture

Three-tier architecture is mostly used method for deploying your web application in the cloud. In this the architecture is divided into three layers.

🔸Presentation layer ➡️ handles user interaction

🔸Application layer(backend logic) ➡️ processes business logic and data processing

🔸Database layer (database) ➡️ manages data storage and retrieval

Each layer has its responsibilities,allowing for scalability,maitainability,modularity.By dividing the architecture into separate layers we can gain security easy updates without impacting others.

## 🏠 Architecture
![Untitled Diagram drawio](https://github.com/user-attachments/assets/f85558a3-c5d9-4478-96d9-8b0e9fb28f42)


## Steps

we are following a warm standby disaster recovery strategy. so we are using two regions for our deployment i.e. US-EAST-1 and US-EAST-2

### 🔹 VPC

We are going to setup VPC in both regiohns to isolate our resources from the internet.
for that i create two VPC's in North-virginia and Oregon.For the high availability of our application we launched in two aZ's.

Firstly i created a VPC in virginia region with 8 subnets in two availability zones.Among them two are public subnets and six are private subnets.

The same VPC setup should be done in the Disaster Recovery region(oregon) to isolate our resources.

![Screenshot 2025-05-19 014915](https://github.com/user-attachments/assets/558a0daf-0c77-48cf-b041-6690ba4b36c6)

In the oregon region also we need to setup 8 subnets in two availability zones.

![Screenshot 2025-05-19 015708](https://github.com/user-attachments/assets/20127ff3-9538-44f2-bf22-1c39fd52f9d4)

Attach Internet Gateways and NAT Gateways in the both the VPC's and palce nat gateways in the public subnets.After that click on vpc setttings and enable "DNS hostnames" check box

## 🔹Security Groups(sg)

Security groups are very essential part of the infrastructure. Because it can secure the resources in the cloud. SGs are a kind of firewall that allow or block incoming and outgoing traffic. SGs are applied to the resources like ALB, ec2, rds, etc. One resource can have more than one SG.

![Screenshot 2025-05-19 014151](https://github.com/user-attachments/assets/98a7adf4-9179-43cc-bb8a-34d38977db2a)

Here we created security groups for bastion server ,ALB frontend,ALB backend,frontend-sg,backend-sg,primary db-sg.Allow SSH for bastion server and allow HTTP,Https for forntend ALB and backend ALB.

Our fronend server and Backend server will be in a private subnet so add the HTTP rule and select the source as ALB-frontend-sg. So only ALB-frontend can access the frontend server on port 80. You have to add one more rule SSH allows from bastion-jump-server-sg. So that the bastion host can log in to web servers. similar to backend server also select sources as ALB Backend.
For primary database allow MySql/Aurora port 3306 from backend-sg so that only backend server can access it.

same setup should be done in recovery region also.

## 🔹 RDS 
Now we need to create Subnet Group to store our database .create subnet group in your primary region by selecting the vpc that you have created and select availability zones us-east-1a and us-east-1b.same to reccovery region also.
I created a database with "book-rds.rd" and place that database in the subnet group that you have created earlier.

![Screenshot 2025-05-19 014659](https://github.com/user-attachments/assets/02d13e58-5bc4-4002-b946-2d46f0ce0ef5)

once the database is created we need to create a read replica for the database that is used in the recovery region. It is similar to the creating the database in the primary region.

## 🔹 Route53

Route 53 is used to provide DNS service that we can access our web app through the domain name. Firstly I registred my domain as "asapclub.click" which costs me around 5 dollars.

![Screenshot 2025-05-19 015230](https://github.com/user-attachments/assets/85fd17bd-7d42-4f45-9428-e6d0ce92bab2)

Now im going to create two hosted zones for my domain to route traffic and failover strategy
Head over to route53 and create hosted zones in the host zone section.select the vpc that you have created. Give domain name and create host zone as private one.After that create a create record that points to our Rds instance.select sample routing and paste the endpoint of your rds server in the CNAME field and create. The same setup should be done in the recovery region also.

![Screenshot 2025-05-21 104907](https://github.com/user-attachments/assets/67fb6525-a49a-4b5e-a3f8-bb650b6461a0)
