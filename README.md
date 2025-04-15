# Employee-Management-App-Full-Production-Deployment-Guide


Here's a comprehensive and cleanly structured **step-by-step deployment project** in Markdown format. This version uses clear transitions and logical groupings for setting up the database, backend, frontend, HTTPS with Certbot, and application reconfiguration.

---

```markdown
# Employee Management App – Full Production Deployment Guide

This guide walks through deploying a full-stack Employee Management Application with:

- Spring Boot (Backend)
- React (Frontend)
- MySQL (RDS)
- Nginx (Web Server)
- Certbot (HTTPS)

---

## Step 1: Database Setup (MySQL RDS)

1. Create an RDS MySQL Database from the AWS Console or use Terraform.
2. Note the following details:
   - **Database Name**
   - **Endpoint**
   - **Username**
   - **Password**
3. Open MySQL Workbench and connect to your RDS instance to test the connection.

---

## Step 2: Local App Test (Before Production Setup)

Test both the backend and frontend locally.

### Backend

```bash
cd backend-directory
mvn clean install
java -jar target/your-backend.jar
```

Edit `application.properties` to update:

```
spring.datasource.url=jdbc:mysql://<your-db-endpoint>:3306/<db-name>
spring.datasource.username=<your-username>
spring.datasource.password=<your-password>
```

### Frontend

```bash
cd frontend-directory
npm install
npm start
```

Edit `src/service/EmployeeService.js` to use:

```js
const BASE_URL = "http://localhost:8080/api/employees";
```

Ensure the application is working locally before proceeding.

---

## Step 3: Backend Production Deployment

### 1. Create a Production Server (Ubuntu)
- SSH into the server and create a non-root user.

### 2. Install Required Software

Install Java 17, Maven, and Node.js:

```bash
sudo apt update -y
sudo apt upgrade -y
sudo apt install openjdk-17-jdk openjdk-17-jre -y
java --version
```

Install Node.js:

```bash
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs
```

### 3. Copy Jar File or Clone and Build

Option A: Copy the jar file directly to the server.  
Option B: Clone the project and build it:

```bash
git clone <your-repo-url>
cd backend-directory
mvn clean install
```

### 4. Create a Systemd Service

```bash
sudo nano /etc/systemd/system/spring.service
```

Paste the following:

```ini
[Unit]
Description=Spring Boot App
After=syslog.target

[Service]
User=spring
Restart=always
RestartSec=30s
ExecStart=/usr/bin/java -jar /home/spring/employee-app/employeemanagmentbackend/target/employeemanagmentbackend-0.0.1-SNAPSHOT.jar
SuccessExitStatus=143

[Install]
WantedBy=multi-user.target
```

Enable and start the service:

```bash
sudo systemctl daemon-reload
sudo systemctl enable spring.service
sudo systemctl start spring.service
```

---

## Step 4: Frontend Production Deployment

### 1. Install NGINX

```bash
sudo apt install nginx -y
sudo ufw allow 'OpenSSH'
sudo ufw allow 'Nginx Full'
sudo ufw allow 8080
```

### 2. Build the React App

```bash
cd employee-app/employeemanagement-frontend
npm run build
```

### 3. Copy Build Files to NGINX Root

```bash
sudo mkdir /var/www/front
sudo cp -r asset-manifest.json index.html static/ /var/www/front/
```

---

## Step 5: NGINX Configuration

### 1. Create NGINX Config File

```bash
sudo mkdir /etc/nginx/sites-available/spring
sudo nano /etc/nginx/sites-available/spring
```

Paste the following configuration:

```nginx
server {
 listen 80;
 server_name lilianedevops.online www.lilianedevops.online;

 location / {
   root /var/www/front;
   index index.html index.htm;
   proxy_http_version 1.1;
   proxy_set_header Upgrade $http_upgrade;
   proxy_set_header Connection 'upgrade';
   proxy_set_header Host $host;
   proxy_cache_bypass $http_upgrade;
   try_files $uri $uri/ /index.html;
 }
}

server {
 listen 80;
 server_name spring.lilianedevops.online;

 location / {
   proxy_pass http://161.35.161.173:8080;
   proxy_http_version 1.1;
   proxy_set_header Upgrade $http_upgrade;
   proxy_set_header Connection 'upgrade';
   proxy_set_header Host $host;
   proxy_cache_bypass $http_upgrade;
 }
}
```

### 2. Enable NGINX Config

```bash
sudo ln -s /etc/nginx/sites-available/spring /etc/nginx/sites-enabled/spring
sudo systemctl restart nginx
```

---

## Step 6: Install and Configure Certbot SSL

### 1. Install Certbot

```bash
sudo snap install core; sudo snap refresh core
sudo apt remove certbot
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

### 2. Generate SSL Certificates

```bash
sudo certbot --nginx -d react.lilianedevops.online -d spring.lilianedevops.online -d front.lilianedevops.online
```

Follow prompts to:
- Enter your email
- Agree to terms
- Configure automatic HTTPS

---

## Step 7: Update Frontend API URLs

### 1. Open Service File

```bash
cd /home/spring/employee-app/employeemanagement-frontend/src/service
nano EmployeeService.js
```

### 2. Update BASE_URL

```js
const BASE_URL = "https://spring.lilianedevops.online/api/employees";
```

Save and close the file.

---

## Step 8: Final Application Test

1. Visit: `https://react.lilianedevops.online`
2. Test creating, viewing, and deleting employees.
3. Ensure HTTPS works for both backend and frontend routes.

---

## Optional

- Set up CI/CD for auto-deploy
- Use environment variables for backend URLs
- Monitor using Prometheus + Grafana or CloudWatch
```

---

Let me know if you'd like this exported as a downloadable `README.md` or pushed into a GitHub repo structure.
