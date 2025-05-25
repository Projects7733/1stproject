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


![Screenshot 2025-05-19 014915](https://github.com/user-attachments/assets/558a0daf-0c77-48cf-b041-6690ba4b36c6)

Firstly i created a VPC in virginia region with 8 subnets in two availability zones.Among them two are public subnets and six are private subnets.
![Screenshot 2025-05-19 015644](https://github.com/user-attachments/assets/04f38fd8-2f3e-46c9-8c9e-c9a615bd5dca)

Here we are creating the resilient architecture for ensuring high security we are not placing the web server in the public subnet for security breaches.simply we place a bastion server in the pub-subnet from there we can communicate with  our web server,App server, Database server which are placed in the private subnet.

The same VPC setup should be done in the Disaster Recovery region(oregon) to isolate our resources.
![Screenshot 2025-05-19 015708](https://github.com/user-attachments/assets/20127ff3-9538-44f2-bf22-1c39fd52f9d4)
