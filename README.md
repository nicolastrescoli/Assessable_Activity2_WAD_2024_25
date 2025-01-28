# Assessable_Activity2_WAD_2024_25
A repository where the Assessable_Activity2_WAD_2024_25 is documented.

## 1. Initial Configuration
For this deployment, we need to select the EC2 option from the dashboard.
![image](https://github.com/user-attachments/assets/b3eed0db-9248-46cc-ac7a-c17ac8d49743)

### 1.1 Security Groups

To configure access to the EC2 instance:
1. Create a new Security Group.
2. Add the following rules:
   - **HTTP**: Allow traffic from anyone (`0.0.0.0/0`).
   - **HTTPS**: Allow traffic from anyone (`0.0.0.0/0`).
   - **SSH**: Restrict traffic to your own IP address for secure CLI access.

---

### 1.2 Instances

1. Launch a new EC2 instance with the following configuration:
   - Name: `react-app`
   - OS: **Ubuntu 24.04 LTS (64-bit ARM architecture)**
   - Instance type: `t4g.nano`
2. Use the previously created Security Group and a key pair for SSH access.

---

### 1.3 Elastic IP

1. Allocate a new Elastic IP.
2. Associate the Elastic IP with your instance.

![image](https://github.com/user-attachments/assets/0dd0e454-c07f-4afa-bcaf-3097b946e121)

> **Note:** During this project you will notice that the IP address changes. This is since the exercise was done throughout several days and having an instance or an IP address in use consumes credit on AWS, so that several instances and IPs have been used among different days.

---

## 2. Project Deployment

### 2.1 SSH Connection

1. Download the `.pem` key file for the instance from AWS.
![image](https://github.com/user-attachments/assets/6422c186-16e2-4d6c-9c42-bd3faebf9d47)

2. Use the key file to connect via SSH:
   ```bash
   ssh -i ./Downloads/labsuser.pem ubuntu@34.195.44.214

### 2.2 Installing Required Services
#### 2.2.1 Update and Upgrade

Run the following commands to ensure your system is up to date:

```bash
   sudo apt update
   sudo apt upgrade -y
```

#### 2.2.2 Install Apache and Enable Proxy

Install Apache and enable the necessary modules for proxying:
```bash
sudo apt install apache2 -y
sudo a2enmod proxy proxy_http
sudo systemctl restart apache2
```
#### 2.2.3 Install Node.js, npm, and pm2

Installing the package manager npm, node.js and pm2 for managing node processes:
```bash
sudo apt install nodejs -y
sudo apt install npm -y
sudo npm install -g pm2
```
### 2.3 Clone and Build the React Project
#### 2.3.1 Clone the Repository

We are going to put the react project into the root directory of the instance, into a subdirectory called “/adivina-el-numero”, so that:
```bash
git clone https://github.com/gisgarme/adivina-el-numero.git /adivina-el-numero
cd /adivina-el-numero
```
#### 2.3.2 Install Dependencies and Build

Install project dependencies and compile the project:
```bash
npm install
npm run build
```
This will generate a "/dist" directory with the compiled React application. This directory will the one that users will have access to.

#### 2.3.3 Start the Application

Use pm2 to start the React project:
```bash
pm2 start npm --name "vite-server" -- run dev -- --host
pm2 startup
pm2 save
```
### 2.4 Configure the Apache Proxy

Redirect all connections from port 80 to the Vite-React port (5173). To do this, modify the Apache configuration:

Create a new configuration file:
```bash
sudo cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/adivina-el-numero.conf
sudo nano /etc/apache2/sites-available/adivina-el-numero.conf
```
Add the following content to the configuration file:
```
<VirtualHost *:80>
    ServerName <elastic-ip>
    DocumentRoot /adivina-el-numero/dist

    <Directory /adivina-el-numero/dist>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ProxyPreserveHost On
    ProxyPass / http://127.0.0.1:5173/
    ProxyPassReverse / http://127.0.0.1:5173/
</VirtualHost>
```
Save and enable the configuration:
```bash
sudo a2ensite adivina-el-numero.conf
sudo systemctl restart apache2
```

## 3 Final Test
After all these configurations, if we access the IP through the brower, we find the application has been deployed successfully.
![image](https://github.com/user-attachments/assets/d00282e9-a41c-4ec5-afc0-7dcca9e054f3)
