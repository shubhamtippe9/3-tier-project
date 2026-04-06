# 3-Tier Project (Tomcat & MySQL on AWS)

A 3-tier web application deployed on AWS using **Nginx** (Jump/Web Server), **Apache Tomcat** (Application Server), and **MariaDB/RDS** (Database Server).

---

## Architecture Overview

```
Internet
   │
   ▼
[Jump Server - Nginx]       ← Public Subnet
   │  (Reverse Proxy)
   ▼
[Application Server - Tomcat]  ← Private Subnet 1
   │
   ▼
[Database Server - MariaDB/RDS] ← Private Subnet 2
```

---

## Prerequisites

- AWS Account
- Key Pair (.pem file)
- Basic knowledge of AWS Console (EC2, VPC, RDS)

---

## Step-by-Step Setup

### 1. Create VPC
- Create a new VPC from the AWS VPC Console.

### 2. Create 3 Subnets
Inside the VPC, create the following subnets:
- `public-subnet` — for the Jump/Web Server
- `private-subnet-1` — for the Application Server
- `private-subnet-2` — for the Database Server

### 3. Enable Auto-Assign Public IP
- Edit the **public-subnet** settings and enable **Auto-assign public IPv4 address**.

### 4. Create Security Group
Create a security group (`my-SG`) with the following inbound rules:
| Port | Protocol | Purpose  |
|------|----------|----------|
| 22   | TCP      | SSH      |
| 80   | TCP      | HTTP     |
| 8080 | TCP      | Tomcat   |
| 3306 | TCP      | MySQL/MariaDB |

### 5. Launch EC2 Instances
| Instance         | Subnet           | Type     |
|------------------|------------------|----------|
| jump-server      | public-subnet    | t3.micro |
| application-server | private-subnet-1 | t3.micro |
| database-server  | private-subnet-2 | t3.micro |

### 6. Create & Attach Internet Gateway
- Create an Internet Gateway (`my-IG`) and attach it to the VPC.

### 7. Create Route Table for Public Subnet
- Create a route table (`my-RT`).
- Add route: `0.0.0.0/0` → Internet Gateway.
- Associate it with **public-subnet** only.

### 8. Transfer Key Pair to Jump Server
```bash
scp -i new.key.pem new.key.pem ec2-user@<jump-server-public-ip>:/home/ec2-user
```

### 9. Create NAT Gateway
- Create a NAT Gateway (`my-NT`) in the **public-subnet** (requires an Elastic IP).
- This allows the private application server to access the internet.

### 10. Create Route Table for Private Subnet (NAT)
- Create a route table (`nat-RT`).
- Add route: `0.0.0.0/0` → NAT Gateway.
- Associate it with **private-subnet-1**.

### 11. Access Application Server via Jump Server
```bash
# On your local machine
ssh -i new.key.pem ec2-user@<jump-server-public-ip>

# On the jump server
chmod 400 new.key.pem
ssh -i new.key.pem ec2-user@<app-server-private-ip>
```

---

## Application Server Setup (Apache Tomcat)

### 12. Download Apache Tomcat
```bash
curl -o apache-tomcat-9.0.113.tar.gz \
  https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.113/bin/apache-tomcat-9.0.113.tar.gz
```

### 13. Extract to /opt
```bash
sudo tar -xzvf apache-tomcat-9.0.113.tar.gz -C /opt
```

### 14. Install Java (Required by Tomcat)
```bash
yum install java -y
```

### 15. Start Tomcat
```bash
cd /opt/apache-tomcat-9.0.113/bin/
./catalina.sh start
```

### 16. Deploy Web Application
```bash
cd /opt/apache-tomcat-9.0.113/webapps/
curl -O https://s3-us-west-2.amazonaws.com/studentapi-c/it/student.war
```

---

## Jump Server Setup (Nginx Reverse Proxy)

### 17. Install Nginx
```bash
yum install nginx -y
```

### 18. Start & Enable Nginx
```bash
systemctl start nginx.service
systemctl enable nginx.service
```

### 19. Configure Reverse Proxy
Edit `/etc/nginx/nginx.conf` and add the following location block:
```nginx
location / {
    proxy_pass http://<app-server-private-ip>:8080/student/;
}
```

### 20. Restart Nginx
```bash
systemctl restart nginx.service
```

---

## Database Setup (MariaDB / AWS RDS)

### 21. Verify Application is Accessible
- Hit the **public IP of the Jump Server** in a browser — you should see the Student Registration Form.
- At this stage, data will **not** be stored (no DB connected yet).

### 22. Create RDS Instance
- Engine: **MariaDB**
- Set username & password
- Select the **same VPC** and **Security Group**
- Select **Availability Zone** matching `private-subnet-2`

### 23. Access Database Server
```bash
ssh -i new.key.pem ec2-user@<database-server-private-ip>
```

### 24. Associate private-subnet-2 with NAT Route Table
- Go to `nat-RT` → Edit subnet associations → add **private-subnet-2**.

### 25. Install MariaDB
```bash
sudo -i
yum install mariadb105* -y
```

### 26. Start & Enable MariaDB
```bash
systemctl start mariadb.service
systemctl enable mariadb.service
```

### 27. Connect to RDS & Create Database
```bash
mysql -h <rds-endpoint> -u <username> -p<password>
```

```sql
CREATE DATABASE studentapp;
USE studentapp;
CREATE TABLE IF NOT EXISTS students (
    student_id INT NOT NULL AUTO_INCREMENT,
    student_name VARCHAR(100) NOT NULL,
    student_addr VARCHAR(100) NOT NULL,
    student_age VARCHAR(3) NOT NULL,
    student_qual VARCHAR(20) NOT NULL,
    student_percent VARCHAR(10) NOT NULL,
    student_year_passed VARCHAR(10) NOT NULL,
    PRIMARY KEY (student_id)
);
```

---

## Connect Application Server to RDS

### 28. Download MySQL Connector
```bash
cd /opt/apache-tomcat-9.0.113/lib/
curl -O https://s3-us-west-2.amazonaws.com/studentapi-cit/mysql-connector.jar
```

### 29. Configure context.xml
```bash
vim /opt/apache-tomcat-9.0.113/conf/context.xml
```

Add the following inside `<Context>`:
```xml
<Resource name="jdbc/TestDB"
    auth="Container"
    type="javax.sql.DataSource"
    maxTotal="500"
    maxIdle="30"
    maxWaitMillis="1000"
    username="<db-username>"
    password="<db-password>"
    driverClassName="com.mysql.jdbc.Driver"
    url="jdbc:mysql://<rds-endpoint>:3306/studentapp?useUnicode=yes&amp;characterEncoding=utf8"/>
```

### 30. Restart Tomcat
```bash
./catalina.sh stop
./catalina.sh start
```

### 31. Verify Full Stack
- Open browser and navigate to `http://<jump-server-public-ip>`
- You should see the **Student Registration Form**
- Register a student — the data will now be stored in the RDS database
- Visit `/viewStudents` to see all stored records

---

## Project Summary

| Component       | Technology         | Location        |
|-----------------|--------------------|-----------------|
| Web/Proxy Layer | Nginx              | Public Subnet   |
| App Layer       | Apache Tomcat 9    | Private Subnet 1|
| DB Layer        | MariaDB (AWS RDS)  | Private Subnet 2|

> **3-Tier Project successfully deployed on AWS!**
