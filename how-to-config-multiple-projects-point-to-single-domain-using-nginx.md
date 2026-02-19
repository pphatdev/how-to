# Configuring Multiple Projects to Point to a Single Domain Using Nginx

This document explains how to configure **Nginx** so multiple projects (web app, API, assets, etc.) can share a **single domain**, using your example configuration for `stackdev.cloud`.

---

## 1. Use Case Overview

You have one domain:

- **Domain**: `https://stackdev.cloud`

And multiple services on the same server:

- **Main Web App** → `http://127.0.0.1:3000`
- **API Service** → `http://127.0.0.1:3001`
- **Assets Service** → `http://127.0.0.1:3002`

You want:

- `https://stackdev.cloud/` → Web app on port `3000`
- `https://stackdev.cloud/api` → API on port `3001`
- `https://stackdev.cloud/assets` → Assets on port `3002`
- HTTP (`:80`) redirected to HTTPS (`:443`)

---

## 2. Prerequisites

Before configuring Nginx:

1. **Nginx installed**
   ```bash
   sudo apt update
   sudo apt install nginx
   ```

2. **Domain DNS** pointing to your server’s IP  
   Example: `stackdev.cloud` → `191.10.1.0` (your server IP)

3. **Services running** locally:
   - Web app on `127.0.0.1:3000`
   - API on `127.0.0.1:3001`
   - Assets service on `127.0.0.1:3002`

4. **SSL certificate** (e.g. from Let’s Encrypt) for `stackdev.cloud`:
   - Full chain: `/etc/letsencrypt/live/stackdev.cloud/fullchain.pem`
   - Private key: `/etc/letsencrypt/live/stackdev.cloud/privkey.pem`

---

## 3. Full Nginx Configuration

Create a file, for example:

```bash
sudo nano /etc/nginx/sites-available/stackdev.cloud.conf
```

Paste the configuration below:

```nginx
# Redirect
# Case: user enters via IP or http (e.g., http://191.10.1.0?ref=123)
# Redirect to domain using HTTPS
server {
    listen 80;
    listen [::]:80;
    server_name stackdev.cloud;

    # Always redirect HTTP to HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;

    server_name stackdev.cloud;

    # Certificate
    ssl_certificate /etc/letsencrypt/live/stackdev.cloud/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/stackdev.cloud/privkey.pem;

    # SSL Protocols (optional but recommended)
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;

    # Compression
    gzip on;
    gzip_types text/plain text/css application/json application/javascript;

    # ===============================
    # 1) Main WebApp '/'
    # https://stackdev.cloud  →  http://127.0.0.1:3000
    # ===============================
    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_buffering off;
    }

    # ===============================
    # 2) API Service
    # ករណី Service Project ផ្សេងគ្នា ប៉ុន្តែចង់ប្រើ Domain តែមួយ
    # Current domain server: https://stackdev.cloud
    # Proxy Pass to API Service: http://127.0.0.1:3001
    # Example:
    #   https://stackdev.cloud/api
    #   → Proxy Pass: http://127.0.0.1:3001/api
    # ===============================
    location /api {
        proxy_pass http://127.0.0.1:3001/api;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # ===============================
    # 3) Assets Service
    # ករណី Service Project ផ្សេងគ្នា ប៉ុន្តែចង់ប្រើ Domain តែមួយ
    # Current domain server: https://stackdev.cloud
    # Proxy Pass to Assets Service: http://127.0.0.1:3002
    # Example:
    #   https://stackdev.cloud/assets
    #   → Proxy Pass: http://127.0.0.1:3002/assets
    # ===============================
    location /assets {
        proxy_pass http://127.0.0.1:3002/assets;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

---

## 4. Explanation

### 4.1 HTTP → HTTPS Redirect

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name stackdev.cloud;
    return 301 https://$server_name$request_uri;
}
```

- Listens on **port 80 (HTTP)**.
- Redirects all traffic to `https://stackdev.cloud` with the original path and query string.
- `301` indicates a **permanent redirect** (good for SEO and browsers).

### 4.2 SSL & Security

```nginx
ssl_certificate /etc/letsencrypt/live/stackdev.cloud/fullchain.pem;
ssl_certificate_key /etc/letsencrypt/live/stackdev.cloud/privkey.pem;

ssl_protocols TLSv1.2 TLSv1.3;
ssl_ciphers HIGH:!aNULL:!MD5;
ssl_prefer_server_ciphers on;
```

- Uses **Let’s Encrypt** certificate files (adjust paths if necessary).
- Restricts SSL/TLS versions to secure ones only.
- Uses secure cipher suites.

### 4.3 Gzip Compression

```nginx
gzip on;
gzip_types text/plain text/css application/json application/javascript;
```

- Enables gzip compression for responses, improving performance.
- You can add more MIME types if needed (e.g., `application/xml`).

### 4.4 Reverse Proxy to Multiple Projects

Each `location` block forwards traffic to a different project via `proxy_pass`.

#### Root `/` → Web App

```nginx
location / {
    proxy_pass http://127.0.0.1:3000;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_buffering off;
}
```

- All requests like:
  - `https://stackdev.cloud/`
  - `https://stackdev.cloud/dashboard`
  - `https://stackdev.cloud/login`
- Go to **Web App** running at `http://127.0.0.1:3000`.

#### `/api` → API Service

```nginx
location /api {
    proxy_pass http://127.0.0.1:3001/api;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
}
```

- Requests like:
  - `https://stackdev.cloud/api`
  - `https://stackdev.cloud/api/users`
  - `https://stackdev.cloud/api/posts/123`
- Are forwarded to `http://127.0.0.1:3001/api...`.

#### `/assets` → Assets Service

```nginx
location /assets {
    proxy_pass http://127.0.0.1:3002/assets;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
}
```

- Requests like:
  - `https://stackdev.cloud/assets/logo.png`
  - `https://stackdev.cloud/assets/js/app.js`
- Are forwarded to `http://127.0.0.1:3002/assets...`.

### 4.5 Important Proxy Headers

Each `location` sets common reverse proxy headers:

```nginx
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto $scheme;
```

- `Host`: Original host requested by the client.
- `X-Real-IP`: Real client IP address.
- `X-Forwarded-For`: Chain of IPs through proxies.
- `X-Forwarded-Proto`: Original protocol (`http` or `https`), useful for apps generating absolute URLs.

---

## 5. Enable and Test the Configuration

### 5.1 Enable Site

Create a symlink from `sites-available` to `sites-enabled`:

```bash
sudo ln -s /etc/nginx/sites-available/stackdev.cloud.conf /etc/nginx/sites-enabled/stackdev.cloud.conf
```

(Optional) Remove the default site if not needed:

```bash
sudo rm /etc/nginx/sites-enabled/default
```

### 5.2 Test Nginx Configuration

```bash
sudo nginx -t
```

Expected output:

```text
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

If there are errors, fix them before continuing.

### 5.3 Reload Nginx

```bash
sudo systemctl reload nginx
# or
sudo systemctl restart nginx
```

---

## 6. Verify Each Project

From your local machine or the server:

```bash
# Root Web App
curl -I https://stackdev.cloud/

# API
curl -I https://stackdev.cloud/api

# Assets
curl -I https://stackdev.cloud/assets
```

You should receive valid responses (200, 301, 302, etc.) depending on each service.

---

## 7. Adding More Projects on the Same Domain

To add another project, e.g., an **admin** service at `http://127.0.0.1:3003`:

1. Add a new `location` block:

```nginx
location /admin {
    proxy_pass http://127.0.0.1:3003/admin;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
}
```

2. Test and reload Nginx:

```bash
sudo nginx -t
sudo systemctl reload nginx
```

Now:

- `https://stackdev.cloud/admin` → `http://127.0.0.1:3003/admin`

---

## 8. Troubleshooting

### 8.1 502 Bad Gateway

**Possible causes:**

- Backend service not running.
- Wrong port or URL in `proxy_pass`.

**Check:**

```bash
# Example: check service on port 3001
curl http://127.0.0.1:3001/api
```

If this fails, fix the service before checking Nginx.

### 8.2 SSL Errors

Make sure the certificate paths exist:

```bash
ls -l /etc/letsencrypt/live/stackdev.cloud/
```

If using Let’s Encrypt, renew with:

```bash
sudo certbot renew --dry-run
```

### 8.3 Check Logs

```bash
# Error logs
sudo tail -f /var/log/nginx/error.log

# Access logs
sudo tail -f /var/log/nginx/access.log
```

---

## 9. Summary

With this configuration, you can:

- Use **one domain** (`stackdev.cloud`) for **multiple services**.
- Route different URL paths (`/`, `/api`, `/assets`, etc.) to different backend projects.
- Terminate **SSL** once at Nginx, simplifying your internal services.
- Keep your infrastructure clean, secure, and easy to extend when you add new services.

For more details, see the official [Nginx documentation](https://nginx.org/en/docs/).
