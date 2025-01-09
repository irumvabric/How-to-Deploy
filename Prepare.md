

Hosting a Django app on an Ubuntu VPS involves several steps, including setting up your VPS, configuring your environment, and deploying your application. Here's a step-by-step guide:

---

### **1. Prepare Your VPS**
1. **Update the server:**
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```
2. **Install essential tools:**
   ```bash
   sudo apt install -y git curl wget build-essential
   ```

---

### **2. Install Python and Virtual Environment**
1. **Install Python 3 and pip:**
   ```bash
   sudo apt install -y python3 python3-pip python3-venv
   ```
2. **Check Python and pip versions:**
   ```bash
   python3 --version
   pip3 --version
   ```

---

### **3. Install and Configure a Database**
1. **Install PostgreSQL (recommended for Django apps):**
   ```bash
   sudo apt install -y postgresql postgresql-contrib libpq-dev
   ```
2. **Create a database and user:**
   ```bash
   sudo -u postgres psql
   CREATE DATABASE your_database_name;
   CREATE USER your_username WITH PASSWORD 'your_password';
   ALTER ROLE your_username SET client_encoding TO 'utf8';
   ALTER ROLE your_username SET default_transaction_isolation TO 'read committed';
   ALTER ROLE your_username SET timezone TO 'UTC';
   GRANT ALL PRIVILEGES ON DATABASE your_database_name TO your_username;
   \q
   ```

---

### **4. Clone and Set Up Your Django Project**
1. **Clone your project repository:**
   ```bash
   git clone https://github.com/your-repo.git
   cd your-repo
   ```
2. **Set up a virtual environment:**
   ```bash
   python3 -m venv venv
   source venv/bin/activate
   ```
3. **Install project dependencies:**
   ```bash
   pip install -r requirements.txt
   ```
4. **Apply migrations:**
   ```bash
   python manage.py migrate
   ```
5. **Collect static files:**
   ```bash
   python manage.py collectstatic
   ```

---

### **5. Configure Gunicorn**
1. **Install Gunicorn:**
   ```bash
   pip install gunicorn
   ```
2. **Run Gunicorn to test:**
   ```bash
   gunicorn --bind 0.0.0.0:8000 your_project_name.wsgi:application
   ```

---

### **6. Install and Configure Nginx**
1. **Install Nginx:**
   ```bash
   sudo apt install -y nginx
   ```
2. **Create an Nginx configuration file:**
   ```bash
   sudo nano /etc/nginx/sites-available/your_project_name
   ```
   Add the following content:
   ```nginx
   server {
       listen 80;
       server_name your_domain_or_ip;

       location = /favicon.ico { access_log off; log_not_found off; }
       location /static/ {
           root /path/to/your/project;
       }

       location / {
           include proxy_params;
           proxy_pass http://127.0.0.1:8000;
       }
   }
   ```
3. **Enable the configuration:**
   ```bash
   sudo ln -s /etc/nginx/sites-available/your_project_name /etc/nginx/sites-enabled
   ```
4. **Test the Nginx configuration:**
   ```bash
   sudo nginx -t
   ```
5. **Restart Nginx:**
   ```bash
   sudo systemctl restart nginx
   ```

---

### **7. Secure Your VPS with SSL (Optional but Recommended)**
1. **Install Certbot:**
   ```bash
   sudo apt install -y certbot python3-certbot-nginx
   ```
2. **Obtain an SSL certificate:**
   ```bash
   sudo certbot --nginx -d your_domain_or_ip
   ```
3. **Verify the auto-renewal configuration:**
   ```bash
   sudo certbot renew --dry-run
   ```

---

### **8. Configure Gunicorn as a Systemd Service**
1. **Create a service file:**
   ```bash
   sudo nano /etc/systemd/system/your_project_name.service
   ```
   Add the following content:
   ```ini
   [Unit]
   Description=gunicorn daemon for your_project_name
   After=network.target

   [Service]
   User=your_username
   Group=www-data
   WorkingDirectory=/path/to/your/project
   ExecStart=/path/to/your/project/venv/bin/gunicorn --workers 3 --bind unix:/path/to/your/project/your_project_name.sock your_project_name.wsgi:application

   [Install]
   WantedBy=multi-user.target
   ```
2. **Start and enable the service:**
   ```bash
   sudo systemctl start your_project_name
   sudo systemctl enable your_project_name
   ```

---

### **9. Finalize Deployment**
- Visit your domain or server IP to verify the deployment.
- Debug any issues by checking the logs for Gunicorn (`journalctl -u your_project_name`) and Nginx (`/var/log/nginx/error.log`).

This setup should result in a robust, production-ready deployment of your Django app on your Ubuntu VPS. Let me know if you encounter any specific issues!
