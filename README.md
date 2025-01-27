# How to Deploy Django REST Framework on AWS EC2 with Nginx, Gunicorn, and Supervisor

This guide provides a comprehensive walkthrough for deploying a Django REST Framework (DRF) application on an AWS EC2 instance. We’ll use **Nginx** as the web server, **Gunicorn** as the application server, and **Supervisor** to manage background processes. By the end of this guide, your Django app will be live and accessible via the internet.

---

## **Table of Contents**
1. [Prerequisites](#prerequisites)
2. [Step 1: Set Up an EC2 Instance](#step-1-set-up-an-ec2-instance)
3. [Step 2: Connect to the EC2 Instance](#step-2-connect-to-the-ec2-instance)
4. [Step 3: Set Up the Django Environment](#step-3-set-up-the-django-environment)
5. [Step 4: Clone and Configure Your Django Project](#step-4-clone-and-configure-your-django-project)
6. [Step 5: Install and Configure Nginx](#step-5-install-and-configure-nginx)
7. [Step 6: Set Up Gunicorn](#step-6-set-up-gunicorn)
8. [Step 7: Test Your Deployment](#step-7-test-your-deployment)
9. [Sample Configuration Files](#sample-configuration-files)
10. [Conclusion](#conclusion)

---

## **Prerequisites**
Before starting, ensure you have the following:
1. An **AWS account** with access to EC2.
2. A **GitHub repository** containing your Django project.
3. Basic familiarity with:
   - AWS services (EC2, Security Groups).
   - Linux commands (Ubuntu).
   - Django and Django REST Framework.

---

## **Step 1: Set Up an EC2 Instance**
1. **Log in to AWS**:
   - Go to the [AWS Management Console](https://aws.amazon.com/console/).
   - Navigate to the **EC2 Dashboard**.

2. **Create a Security Group**:
   - Click on **Security Groups** in the left sidebar.
   - Click **Create Security Group**.
   - Name: `my-sg`.
   - Description: `Security group for Django app`.
   - Add the following inbound rules:
     - **SSH** (Port 22) – Allow access from your IP.
     - **HTTP** (Port 80) – Allow access from all IPv4 and IPv6 addresses (`0.0.0.0/0` and `::/0`).

3. **Launch an EC2 Instance**:
   - Click **Instances** in the left sidebar.
   - Click **Launch Instances**.
   - Name: `my-server`.
   - Choose **Ubuntu 22.04 LTS** as the AMI.
   - Select the **t2.micro** instance type (free tier eligible).
   - Under **Key Pair**, select **Proceed without a key pair**.
   - Under **Network Settings**, select the security group `my-sg`.
   - Click **Launch Instance**.

---

## **Step 2: Connect to the EC2 Instance**
1. **Connect via EC2 Instance Connect**:
   - Go to the **Instances** page.
   - Select your instance and click **Connect**.
   - Choose **EC2 Instance Connect** and click **Connect**.

2. **Update System Packages**:
   - Run the following commands to update and upgrade the system:
     ```bash
     sudo apt-get update
     sudo apt-get upgrade -y
     ```

---

## **Step 3: Set Up the Django Environment**
1. **Install Python and Required Packages**:
   ```bash
   sudo apt-get install python3-pip python3-venv -y
   ```

2. **Create a Virtual Environment**:
   ```bash
   python3 -m venv env
   source env/bin/activate
   ```

3. **Install Django and Gunicorn**:
   ```bash
   pip install django gunicorn
   ```

---

## **Step 4: Clone and Configure Your Django Project**
1. **Clone Your GitHub Repository**:
   ```bash
   git clone https://github.com/your-username/your-repo.git
   cd your-repo
   ```

2. **Install Project Dependencies**:
   ```bash
   pip install -r requirements.txt
   ```

3. **Update `settings.py`**:
   - Set `ALLOWED_HOSTS` to include your EC2 public IP:
     ```python
     ALLOWED_HOSTS = ['your-ec2-public-ip']
     ```
   - Configure the database settings (if using PostgreSQL or MySQL).

---

## **Step 5: Install and Configure Nginx**
1. **Install Nginx**:
   ```bash
   sudo apt-get install nginx -y
   ```

2. **Create a Configuration File for Your Django App**:
   ```bash
   sudo nano /etc/nginx/sites-available/django.conf
   ```
   Add the following:
   ```nginx
   server {
       listen 80;
       server_name your-ec2-public-ip;

       location / {
           include proxy_params;
           proxy_pass http://unix:/home/ubuntu/your-repo/app.sock;
       }
   }
   ```

3. **Enable the Configuration**:
   ```bash
   sudo ln -s /etc/nginx/sites-available/django.conf /etc/nginx/sites-enabled/
   sudo nginx -t  # Test the configuration
   sudo systemctl restart nginx
   ```

---

## **Step 6: Set Up Gunicorn**
1. **Install Supervisor**:
   ```bash
   sudo apt-get install supervisor -y
   ```

2. **Create a Gunicorn Configuration File for Supervisor**:
   ```bash
   sudo nano /etc/supervisor/conf.d/gunicorn.conf
   ```
   Add the following:
   ```ini
   [program:gunicorn]
   directory=/home/ubuntu/your-repo
   command=/home/ubuntu/env/bin/gunicorn --workers 3 --bind unix:/home/ubuntu/your-repo/app.sock your_project.wsgi:application
   autostart=true
   autorestart=true
   stderr_logfile=/var/log/gunicorn/gunicorn.err.log
   stdout_logfile=/var/log/gunicorn/gunicorn.out.log

   [group:guni]
   programs:gunicorn
   ```

3. **Create a Log Directory for Gunicorn**:
   ```bash
   sudo mkdir -p /var/log/gunicorn
   ```

4. **Reload Supervisor**:
   ```bash
   sudo supervisorctl reread
   sudo supervisorctl update
   sudo supervisorctl status
   ```

---

## **Step 7: Test Your Deployment**
1. **Visit Your EC2 Public IP**:
   - Open a browser and navigate to `http://your-ec2-public-ip`.
   - If everything is set up correctly, you should see your Django app running.

---

## **Sample Configuration Files**
### **gunicorn.conf**
```ini
[program:gunicorn]
directory=/home/ubuntu/your-repo
command=/home/ubuntu/env/bin/gunicorn --workers 3 --bind unix:/home/ubuntu/your-repo/app.sock your_project.wsgi:application
autostart=true
autorestart=true
stderr_logfile=/var/log/gunicorn/gunicorn.err.log
stdout_logfile=/var/log/gunicorn/gunicorn.out.log

[group:guni]
programs:gunicorn
```

### **django.conf**
```nginx
server {
    listen 80;
    server_name your-ec2-public-ip;

    location / {
        include proxy_params;
        proxy_pass http://unix:/home/ubuntu/your-repo/app.sock;
    }
}
```

---

## **Conclusion**
By following this guide, you’ve successfully deployed a Django REST Framework application on AWS EC2 using Nginx, Gunicorn, and Supervisor. This setup ensures your app is secure, scalable, and runs smoothly in the background.

For advanced configurations:
- Add HTTPS using **Let’s Encrypt**.
- Scale your application using **AWS Elastic Load Balancer (ELB)**.

---
