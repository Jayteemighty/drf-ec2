# How to Deploy Django REST Framework on AWS EC2 with Nginx, Gunicorn, and Supervisor**
## **README.md: Deploying Django REST Framework on AWS EC2**

This guide walks you through deploying a Django REST Framework (DRF) application on AWS EC2 using **Nginx**, **Gunicorn**, and **Supervisor**.

---

### **Prerequisites**
- AWS account with EC2 access.
- GitHub repository with your Django project.
- Basic knowledge of AWS, Linux, and Django.

---

### **Steps**
1. **Set Up an EC2 Instance**:
   - Create a security group with SSH and HTTP access.
   - Launch an Ubuntu 22.04 LTS instance.

2. **Connect to the EC2 Instance**:
   - Use EC2 Instance Connect.
   - Update system packages.

3. **Set Up the Django Environment**:
   - Install Python, create a virtual environment, and install Django and Gunicorn.

4. **Clone and Configure Your Django Project**:
   - Clone your GitHub repository and install dependencies.
   - Update `settings.py` for production.

5. **Install and Configure Nginx**:
   - Install Nginx and create a configuration file for your app.

6. **Set Up Gunicorn**:
   - Create a Gunicorn configuration file for Supervisor.
   - Reload Supervisor to manage Gunicorn.

7. **Test Your Deployment**:
   - Visit your EC2 public IP to verify the app is running.

---

### **Sample Configuration Files**
#### **gunicorn.conf**
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

#### **django.conf**
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

### **Conclusion**
This setup ensures your Django app is secure, scalable, and runs smoothly in the background. For advanced configurations, consider adding HTTPS or scaling with AWS Elastic Load Balancer (ELB).
