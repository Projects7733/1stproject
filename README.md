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

![3-tier drawio (1)](https://github.com/user-attachments/assets/b3abf0b6-d3bd-4217-9656-3fc2233980a8)

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

We need to setup a health check for our regions through route53 in order if one endpoint fails route53 detects and route traffic to another region. In route53 go to health check section and crete a health check and select an endpoint to monitor.Give the DNS of ALB-backend in domain name and then create.
![Screenshot 2025-05-19 015321](https://github.com/user-attachments/assets/a8557c71-be48-472c-b974-d43f8686f822)
After that we need to create failover record for our ALB-backend and give DNS of ALB-backend in our primary region and in health check seection select the helath id that we have created just above.
The same faiover record should be setup in the secon region.

## 🔹 Certificate Manager
AWs certificate manager is used to requst SSl/TLs certificate for securing our websites.
Head over to the ACM and requst certificate manager and enter domain name and request and status is pending.After that you need to add CNAME record then click on create record in route53 and click on create button.


we need set ACM in both regions.
![Screenshot 2025-05-19 015051](https://github.com/user-attachments/assets/ddcf5c25-324c-417e-882d-1ffa2f2d26bc)

## 🔹 Application Load balancer(ALB) 

Load balancers are used to route tarffic between the servers when requests are huge sometimes.we need to setup two load balancers for our architecture one for frontend and another one for backend.First we have to create target groups for instances.so create target groups in the ec2 dashboard select the vpc that you have created and protocol aims to http 80. 

![Screenshot 2025-05-19 014033](https://github.com/user-attachments/assets/2f4665cb-a7dc-4f36-9fcb-5db33d244c26)

we have to setup in the recovery region also.

Now let's associate these TG with the load balancer.so go to load balancer section create load balancer with vpc of our and select the availability zones that we have to setup instances select internet-facing.In the security group section select the sg(ALB-frontend-sg) that we have created in the earlier and in listeners part select the target group we created(ALB-frontend-TG) and create.

![Screenshot 2025-05-19 013918](https://github.com/user-attachments/assets/4ae2a6fe-9524-4520-a265-b8b01bdabd5f)

In the backend load balancer add another listener part by default it is http we need to select https in listeners section and select target group we created for backend ALB and in the secure listeners settings add our certificate for security of our backend.
we have to setup in the recovery region also.

## 🔹 EC2

we are going to create a temporary frontend and backend server to do all the required setup, take snapshots and create Machine images from it. So that we can utilize it in the launch template.
we are going to set up in one region only. By using the backup service and copying it in the recovery region.
Now go to ec2 service and  create an instance in the for frontend server and create a new keypair for our servers .In the firewalls settings allow all the security groups and in the advanced details section paste the script for launching instance.

![Screenshot 2025-05-19 014225](https://github.com/user-attachments/assets/9fdddb1c-fbf4-451b-9822-e2d579378997)

Now open git bash and login into our frontend server using the .pem file we hae created in earlier and clone the repo for our application files.In your case your application repository. You to configure api file by replacing with our domain name.In my case "https://api.asapclub.click".so that our api  calls on the domain name that will point to our backend server.
After that type npm install to install required packages.

![Screenshot 2025-05-18 222151](https://github.com/user-attachments/assets/84ea15ab-aaec-495e-b828-a3095b434376)

Now login in to the backend server with same pem file create one file. in that file
we have to add DB_HOST="your db name" DB_USEERNAME="username" DB-PASSWORD="password" PORT=3306.close the file and save it
now type npm install
and start backend server with command "sudo pm2 start index.js --name "backendApi".

🔹AMI
we need to create AmI's to create launch templates.so select the frontend server actions select images and template option and then click on create.

![Screenshot 2025-05-19 014326](https://github.com/user-attachments/assets/7b9ba78d-8899-4315-bffb-c4f1a6d2c65b)

## 🔹 Launch Template
Now we have to create launch template for our regions .In the ec2 section select launch template section and select ami that you have created above and select security groups we created for frontend server" primary frontend sg" and  we have to add user-data in advanced details andclick on create.
we need to setup for backened server and another region also.

![Screenshot 2025-05-19 014404](https://github.com/user-attachments/assets/06c8ebf0-1c7b-4b7a-92b3-ce53fec57b47)

## 🔹 Backup service

Backup service is used to copy the images we have created in the primary region into recovery region.so first create backup vault.Go to backup service and create vault with name of our wish and click on create.After that create backup plan with name ,backup time and select 2nd region that you want to copy and create.
Its time to assign resources to our backup so go to assign resources give it name and select specific resource type and select the instance types that we created in ec2 and click on assign resources.

![Screenshot 2025-05-19 020307](https://github.com/user-attachments/assets/ae56a0fc-c2ec-4530-a73c-d3c885efe3d3)
After sometime you will see the instance AMI in the recovery region under the ec2 dashboard in both regions.
## 🔹 Target groups
In the ec2 dashboard configure TG.Give it name for target group select instances as target type and vpc of our and create it.
![Screenshot 2025-05-19 014033](https://github.com/user-attachments/assets/e5af79a4-5386-4f90-9aec-df8eb3b1900a)

## 🔹 Auto scaling group (ASG)
Auto scaling groups enables our instances network traffic ,cpu utilization and automatic scaling up and down.It launches instances from the launch template.Go to Auto scaling Groups create ASG's for frontend and backend servers.Give name to your ASG and select template for frontend server and in vpc select private subnets we have created fpr frontend server. In Next section load balancing attach to a load balancer we have created for frontend servers and select Target group for frontend ALB and next set the group size based on traffic and then create.The same process should be for backend server.

![Screenshot 2025-05-19 014014](https://github.com/user-attachments/assets/2d3cebdd-7d61-4e7c-abae-ff00701cc335)
I have created the same process for the recovery region.

We need to initialize our database and need to create some tables. But we can’t access the RDS instance or backend server directly coz they are in a private subnet and the security group won’t let us login into it. So we need to launch an instance in the same VPC but in the public subnet that instance is called bastion host(jump-server). And through that instance, we will log in to the backend server, and from the backend server we will initialize our database.

![Screenshot 2025-05-19 020307](https://github.com/user-attachments/assets/11dcfc69-3285-46b3-a92c-9d8cb7499412)
 login into the jump server with 3-tier.pem file .from there we can login into backend server to initialize the database.for this we have to install mysql **sudo apt install mysql-server -y**.After that initialize the database with command **mysql -h book.rds.com -u <admin> -p<Yuvaraj123> test < test.db**
![Screenshot 2025-05-18 222151](https://github.com/user-attachments/assets/c132d3a4-c910-4261-b628-5103c8f4d913)
our website can be accessed through the ALB-frontend but it can't be in total functional mode because of ApI's try to call from our browser on the domain name "https://api.asapclub.click" this record didn't add in our domain.so create that record for accessing our website in functional mode.

 In the route53 choose the public hosted zone and create record with failover routing policy and in record name type **api.domain name**click on define failover record.In that select route traffic to **ALB-backend** and in the primary region failover type(primary) and health check Id we created for primary region and create it.
 The same failover record should be setup for recovery region also but without a health check.
 Now we can access the website using our ALB-forntend  DNS.
 
![cloudfront](https://github.com/user-attachments/assets/a5666a53-a4de-41be-bdb0-0067a077a043)


## 🔹 CloudFront
Cloud front is a content delivery network used for caching of every edge location.due to this users face less latency and high performance.For this create distribution in the origin domain select ALB-forntend allow http,https methods and add item with alternative domain name(ou domain name) and select certificate and create distribution.
After it is created i have selected create origin and select second region ALb-frontend.
Again we need to create origin groups in that we select primary region and select all failover criteria options and then create origin group. In the behavior tab select failover group we created above and save it.
Again i have created another record for **3-tier** so our website looks like **3-tier.asapclub.click**.For this i select route traffic to cloudfront distribution and defined the simple record. After sometime our website will look like **https://3-tier.asapclub.click**.

## 🔹 AWS WAF (Web application firewall)
AWS WAF is a firewall that helps to protect apps and api against bots and attackers.Go to WAF service create web ACL's with name and select resources type with cloudfront distribution and hit rule select managed rule there we can select some rules related to security and save it.
Its time for testing our application. In the primary region select the ALB-frontend-sg and ALB-backend-sg and remove the inbound rule like http,https then we created some situation like a disaster.

After sometime route53 will monitor the healthcheck of primary region and find it unhealthy and route traffic to recovery region.we cant add books in secondary region because in the recovery region DB is in read replica only weh have to edit db in secondary region and change to "promote" in Actions to se in functional mode.

