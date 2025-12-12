# How to deploy a Simple Web Application Next.js on Ubuntu Server From Scratch
This guide will walk you through the steps to deploy a simple Next.js web application on an Ubuntu server from scratch. We will cover setting up the server, installing necessary software, and deploying the application.

## Requirements
- An Ubuntu server (18.04 or later)
- A domain name
- Basic knowledge of terminal commands and SSH
- A Next.js application ready for deployment
- Node.js installed on your local machine
- Git installed on your local machine
- SSH access to your Ubuntu server
- A text editor (like VSCode, nano, etc.)
- A package manager like npm or yarn
- A process manager like PM2
- A web server like Nginx
- A certbot for SSL certificates


## Step 1: Update and Upgrade the Server

> Please use `sudo` if required permission

First, connect to your Ubuntu server via SSH:

```bash
ssh root@72.61.124.193
```
Once connected, update and upgrade the server packages:

```bash
sudo apt update && sudo apt upgrade -y
```

## Step 2: Install Node.js and npm
Install Node.js and npm using the NodeSource repository:

```bash
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs
```
Verify the installation:

```bash
node -v
npm -v
```

## Step 3: Install Git
Install Git to clone your Next.js application repository:

```bash
sudo apt install git -y
```

Verify the installation:

```bash
git --version
```

## Step 4: Clone Your Next.js Application
Clone your Next.js application from your Git repository:

```bash
git clone https://github.com/pphatdev/pphat.stackdev.cloud.git
cd pphat.stackdev.cloud
```

## Step 5: Install Application Dependencies
Install the necessary dependencies for your Next.js application:

```bash
npm install
```

## Step 6: Build the Next.js Application
Build your Next.js application for production:

```bash
npm run build
```

## Step 7: Install PM2
Install PM2 to manage your application process:
```bash
sudo npm install -g pm2
```

## Step 8: Start the Application with PM2
Start your Next.js application using PM2:
```bash
pm2 start npm --name "pphat.stackdev.cloud" -- start -- -p 3000
```

## Step 9: Configure PM2 to Start on Boot
Set up PM2 to start your application on server boot:
```bash
pm2 startup systemd
```


Follow the instructions provided by the command to enable the startup script. Then save the PM2 process list:
```bash
pm2 save
```

## Step 10: Install and Configure Nginx
Install Nginx:
```bash
sudo apt install nginx -y
```

Configure Nginx as a reverse proxy for your Next.js application. Create a new configuration file:
```bash
sudo nano /etc/nginx/sites-available/pphat.stackdev.cloud.conf
```

Add the following configuration, replacing `pphat.stackdev.cloud` with your actual domain name:
```nginx
server {
    listen 443 ssl;
    listen [::]:443 ssl;
    server_name pphat.stackdev.cloud;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

Enable the Nginx configuration:
```bash
sudo ln -s /etc/nginx/sites-available/pphat.stackdev.cloud.conf /etc/nginx/sites-enabled/
sudo nginx -t
sudo service nginx restart
```

## Step 11: Obtain an SSL Certificate

DNS Configuration
Make sure your domain name points to your server's IP address.

### DNS Type A Record
- Host: @ (or your domain name: pphat.stackdev.cloud)
- Value: Your server's IP address (72.61.124.193)
- TTL: Automatic or 1 hour

### Verify DNS Propagation
You can use tools like [WhatsMyDNS](https://www.whatsmydns.net/) to check if your domain is pointing to the correct IP address.

Install Certbot
```bash
sudo apt install certbot python3-certbot-nginx -y
```

Obtain and Install the SSL Certificate
```bash
sudo certbot --nginx -d pphat.stackdev.cloud
```

Follow the prompts to complete the SSL installation.
The certbot will automatically configure Nginx to use the SSL certificate.

### Verify SSL Configuration on Nginx
Check the Nginx configuration file to ensure SSL is set up correctly:
```bash
sudo nano /etc/nginx/sites-available/pphat.stackdev.cloud.conf
```

## Step 12: Test Your Application
Open your web browser and navigate to `https://pphat.stackdev.cloud`. You should see your Next.js application running securely over HTTPS.
