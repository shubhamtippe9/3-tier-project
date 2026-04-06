🚀 3-Tier Architecture Project (Tomcat & MySQL)
📌 Project Overview

This project demonstrates a 3-Tier Architecture deployed on AWS using:

🌐 Nginx (Web Layer / Jump Server)
⚙️ Apache Tomcat (Application Layer)
🗄️ MySQL / MariaDB (Database Layer - RDS)

It follows best practices with VPC, Subnets, NAT Gateway, and Security Groups.

🏗️ Architecture Diagram (Logical)
User → Nginx (Public Subnet) → Tomcat (Private Subnet) → MySQL RDS (Private Subnet)
⚙️ Technologies Used
☁️ AWS (EC2, VPC, RDS, NAT Gateway)
🌐 Nginx
⚙️ Apache Tomcat
🗄️ MySQL / MariaDB
🐧 Linux (Amazon Linux / Ubuntu)
🔐 SSH & Key Pair
📂 Project Setup Steps
🔹 1. Networking Setup
Create VPC
Create Subnets:
🌍 Public Subnet
🔒 Private Subnet 1 (App Server)
🔒 Private Subnet 2 (DB Server)
Attach Internet Gateway
Configure Route Tables
🔹 2. Security Configuration
Create Security Group:
✅ SSH (22)
✅ HTTP (80)
✅ Tomcat (8080)
✅ MySQL (3306)
🔹 3. EC2 Instances Setup
🖥️ Jump Server → Public Subnet
⚙️ Application Server → Private Subnet
🗄️ Database Server → Private Subnet
🔹 4. NAT Gateway Setup
Create NAT Gateway in Public Subnet
Attach it to Private Subnet Route Table
🔹 5. Application Server Setup (Tomcat)
# Install Java
sudo yum install java -y

# Download Tomcat
cd /opt
wget <tomcat-url>
tar -xvf apache-tomcat.tar.gz

# Start Tomcat
cd bin
./catalina.sh start
🔹 6. Deploy Application
Upload your website into:
/opt/apache-tomcat/webapps/
🔹 7. Nginx Setup (Jump Server)
sudo yum install nginx -y
sudo systemctl start nginx
sudo systemctl enable nginx
Configure reverse proxy in:
/etc/nginx/nginx.conf
🔹 8. Database Setup (RDS - MariaDB)
Create RDS Instance
Configure:
Username & Password
VPC & Security Group
Connect:
mysql -h <endpoint> -u <username> -p
🔹 9. Connect App to Database
Download MySQL Connector → /lib folder
Edit:
/opt/apache-tomcat/conf/context.xml
Add DB credentials:
Endpoint
Username
Password
🔹 10. Restart Services
./catalina.sh stop
./catalina.sh start
✅ Final Output
🌍 Access website using Public IP of Jump Server
💾 Data stored in RDS Database
🎯 Key Features
🔐 Secure architecture (Private Subnets)
🌐 Public access via Nginx
⚙️ Scalable design
☁️ Cloud-native deployment


👨‍💻 Author

Shubham Tippe Cloud & DevOps Learner

GitHub
https://github.com/shubhamtippe9

linkedin
https://www.linkedin.com/in/shubhamtippe9

📜 License

This project is for educational and learning purposes.
