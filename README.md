# 🛒 PrestaShop 9.1 Deployment on AWS (EC2 + RDS)

> A fully functional e-commerce store deployed on AWS Free Tier using Amazon EC2 as the web/application server and Amazon RDS MySQL as a dedicated, separate database instance.

---

## 🌐 Live URLs

| Resource | URL |
|---|---|
| Store Front-end | [https://ec2-13-49-228-209.eu-north-1.compute.amazonaws.com/](https://ec2-13-49-228-209.eu-north-1.compute.amazonaws.com/) |


> Store Name: Cyber-Testing  
> Deployment Date: 13 April 2026

---

## 🏗️ Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                        AWS (eu-north-1)                     │
│                                                             │
│   ┌─────────────────────────┐    ┌──────────────────────┐  │
│   │      EC2 Instance        │    │    RDS Instance      │  │
│   │  (prestashop-app)        │    │  (prestashop-db)     │  │
│   │                          │    │                      │  │
│   │  Ubuntu 22.04 LTS        │───▶│  MySQL (latest)      │  │
│   │  t3.micro (Free Tier)    │    │  db.t3.micro         │  │
│   │                          │    │  20 GiB gp2          │  │
│   │  Apache 2  + PHP 8.3     │    │                      │  │
│   │  PrestaShop 9.1.0        │    │  Port: 3306          │  │
│   └─────────────────────────┘    └──────────────────────┘  │
│          Public IPv4: 13.49.228.209          Same VPC       │
└─────────────────────────────────────────────────────────────┘
```

Key design decision: The database runs on a dedicated RDS managed instance, completely separate from the EC2 application server. This follows best practices for separation of concerns, scalability, and managed backup/maintenance.

---

## 📋 Tech Stack

| Component | Technology |
|---|---|
| Cloud Provider | Amazon Web Services (AWS Free Tier) |
| Web Server | Apache 2 |
| Application Server | EC2 — Ubuntu 22.04 LTS, t3.micro |
| Language Runtime | PHP 8.3 |
| Database | Amazon RDS — MySQL (latest), db.t3.micro |
| E-commerce Platform | PrestaShop 9.1.0 |
| Region | eu-north-1 (Stockholm) |

---

## 🚀 Implementation Steps

### Step 1 — Launch EC2 Instance

- AMI: Ubuntu Server 22.04 LTS
- Instance type: t3.micro (Free Tier eligible)
- Region: eu-north-1
- Key pair: New `.pem` key pair created and downloaded

Security Group Rules:

| Type | Protocol | Port | Source |
|---|---|---|---|
| SSH | TCP | 22 | My IP only |
| HTTP | TCP | 80 | 0.0.0.0/0 |
| HTTPS | TCP | 443 | 0.0.0.0/0 |

![EC2 Launch Instance summary page](docs/screenshots/screenshot(1).png)




![EC2 instance in Running state](docs/screenshots/screenshot(2).png): `ec2-13-49-228-209.eu-north-1.compute.amazonaws.com`

---

### Step 2 — Create RDS Database (Separate from EC2)

- Engine: MySQL (latest version)
- Template: Free Tier
- DB Instance Identifier: `prestashop-db`
- Master Username: `admin`
- Instance Class: db.t3.micro
- Storage: 20 GiB gp2
- Availability Zone: eu-north-1b
- Publicly Accessible: Yes (for this assignment only)
- Port: 3306

> RDS was intentionally created after the EC2 instance and kept separate to ensure the database runs on a dedicated managed MySQL server — not on the application server.

![RDS Database Creation page](docs/screenshots/screenshot(3).png)

![RDS instance Available status](docs/screenshots/screenshot(4).png)

### Step 3 — Set Up LAMP Stack on EC2

Connect to EC2 via SSH:

```bash
chmod 400 prestashop-key.pem
ssh -i prestashop-key.pem ubuntu@ec2-13-49-228-209.eu-north-1.compute.amazonaws.com
```

Update system and install Apache + PHP 8.3:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install apache2 php8.3 php8.3-mysql php8.3-gd php8.3-curl php8.3-xml \
  php8.3-mbstring php8.3-zip php8.3-intl php8.3-opcache unzip -y
sudo systemctl enable --now apache2
```

Download and extract PrestaShop 9.1.0:

```bash
cd /var/www/html
sudo wget https://api.prestashop-project.org/assets/prestashop-classic/9.1.0-4.0/prestashop.zip \
  -O prestashop.zip
sudo unzip prestashop.zip
```

Set correct file permissions:

```bash
sudo chown -R www-data:www-data /var/www/html
sudo chmod -R 755 /var/www/html
```

![Database connection error](docs/screenshots/screenshot(5).png)

---

### Step 4 — Run the PrestaShop Web Installer & Connect to RDS

Opened browser and navigated to: `http://ec2-13-49-228-209.eu-north-1.compute.amazonaws.com`

Database Configuration (where EC2 and RDS are joined):

| Field | Value |
|---|---|
| Database server address | `prestashop-db.cpcy2sc66oem.eu-north-1.rds.amazonaws.com` |
| Database name | `prestashop` |
| Database login | `admin` |
| Tables prefix | `ps_` |

#### ⚠️ Issue Encountered & Resolution

Error: `Connection to MySQL server succeeded, but database 'prestashop' not found`

Cause: The RDS initial database name was left blank during creation.

Solution: Connected to RDS from EC2 via MySQL client and manually created the database:

```sql
CREATE DATABASE `prestashop` CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
GRANT ALL PRIVILEGES ON `prestashop`. TO 'admin'@'%';
FLUSH PRIVILEGES;
```



![Database connection error](docs/screenshots/screenshot(6).png)


---

### Step 5 — Post-Installation Security

After installation completed, the `/install` directory was removed to prevent unauthorized reinstallation:

```bash
sudo rm -rf install
```




> Screenshot: PrestaShop security warning page requiring deletion of `/install` folder before Back Office access is permitted.

> Screenshot: PrestaShop Back Office (Admin Dashboard) fully accessible after the `/install` folder was deleted — showing store dashboard with sales forecast chart.

---

### Step 6 — Final Verification

- ✅ Live store accessible and functional at the public URL
- ✅ Database running on dedicated RDS instance (separate from EC2)
- ✅ All AWS resources remained within Free Tier limits



---

## ✅ Outcome

PrestaShop 9.1 was successfully deployed by:

1. Launching the EC2 instance first
2. Creating the RDS MySQL database second (separate hosting)
3. Connecting them during the PrestaShop web installer

The store is now live, publicly accessible, and fully operational within AWS Free Tier constraints.

---

## 📁 Repository Structure

```
/
├── README.md               ← This file
├── docs/
│   ├── screenshots/        ← All referenced screenshots (Figures 1–10)
│   └── architecture.png    ← Architecture diagram export
└── config/
    └── security-group.md   ← Security group rules reference
```

---

## 📝 Notes

- SSH access is restricted to the deployer's IP address only for security.
- The `/install` folder has been permanently deleted post-deployment.
- The admin Back Office URL uses an obfuscated path for added security.
- RDS was set to "Publicly accessible: Yes" for this assignment only — this should be disabled in production environments.
- HTTPS is configured but the certificate is self-signed; a production deployment should use AWS Certificate Manager (ACM) or Let's Encrypt.

---

Deployed on AWS Free Tier — eu-north-1 (Stockholm)
