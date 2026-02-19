# How to Configure Nginx for Multiple Subdomains  
## Same Service & Different Services on One Server

This document explains how to configure **Nginx** to handle multiple **subdomains** on a single server.  
We’ll cover two scenarios:

1. Multiple subdomains pointing to the **same service**
2. Subdomains pointing to **different services**

We’ll use your example for the domain `stackdev.cloud`:

- Main domain: `stackdev.cloud`
- API subdomain: `api.stackdev.cloud`
- Assets subdomain: `asset.stackdev.cloud`

---

## 1. Overview of the Setup

### Services

- Main Web App:
  - URL: `https://stackdev.cloud/`
  - Backend: `http://127.0.0.1:3100`

- API (same service as main, but with `/api` path):
  - URL: `https://api.stackdev.cloud/`
  - Backend: `http://127.0.0.1:3100/api`

- Assets (different service / different port):
  - URL: `https://asset.stackdev.cloud/`
  - Backend: `http://127.0.0.1:3101/asset`

### Certificates

Each domain/subdomain has its own SSL certificate:

- `stackdev.cloud`
  - `/etc/letsencrypt/live/stackdev.cloud/fullchain.pem`
  - `/etc/letsencrypt/live/stackdev.cloud/privkey.pem`

- `api.stackdev.cloud`
  - `/etc/letsencrypt/live/api.stackdev.cloud/fullchain.pem`
  - `/etc/letsencrypt/live/api.stackdev.cloud/privkey.pem`

- `asset.stackdev.cloud`
  - `/etc/letsencrypt/live/asset.stackdev.cloud/fullchain.pem`
  - `/etc/letsencrypt/live/asset.stackdev.cloud/privkey.pem`

---

## 2. Full Nginx Configuration Example

Create a file like:

```bash
sudo nano /etc/nginx/sites-available/stackdev-subdomains.conf
```


### If you don't generated certificate yet
Paste the following to protected error when generating certificates:

```nginx
########################################
# Main domain: stackdev.cloud
# Uses service on port 3100
########################################
server {
    server_name stackdev.cloud;

    location / {
        proxy_pass http://127.0.0.1:3100;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

########################################
# API subdomain: api.stackdev.cloud
# Same service as main app (port 3100),
# but using the /api path at the backend
########################################
server {
    server_name api.stackdev.cloud;

    location / {
        # Note: /api path on the same service
        proxy_pass http://127.0.0.1:3100/api;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

########################################
# Assets subdomain: asset.stackdev.cloud
# Different service on port 3101,
# using /asset path at the backend
########################################
server {
    server_name asset.stackdev.cloud;

    location / {
        # Different service + different port
        proxy_pass http://127.0.0.1:3101/asset;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

> Note: After generating the certificates, the certbot will create new ssl configuration on `/etc/nginx/sites-available/stackdev-subdomains.conf` automatically.

###  Case Generated Certificates
Paste the following:

```nginx
########################################
# Main domain: stackdev.cloud
# Uses service on port 3100
########################################
server {
    listen 443 ssl;
    listen [::]:443 ssl;

    server_name stackdev.cloud;

    ssl_certificate /etc/letsencrypt/live/stackdev.cloud/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/stackdev.cloud/privkey.pem;

    location / {
        proxy_pass http://127.0.0.1:3100;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

########################################
# API subdomain: api.stackdev.cloud
# Same service as main app (port 3100),
# but using the /api path at the backend
########################################
server {
    listen 443 ssl;
    listen [::]:443 ssl;

    server_name api.stackdev.cloud;

    ssl_certificate /etc/letsencrypt/live/api.stackdev.cloud/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.stackdev.cloud/privkey.pem;

    location / {
        # Note: /api path on the same service
        proxy_pass http://127.0.0.1:3100/api;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

########################################
# Assets subdomain: asset.stackdev.cloud
# Different service on port 3101,
# using /asset path at the backend
########################################
server {
    listen 443 ssl;
    listen [::]:443 ssl;

    server_name asset.stackdev.cloud;

    ssl_certificate /etc/letsencrypt/live/asset.stackdev.cloud/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/asset.stackdev.cloud/privkey.pem;

    location / {
        # Different service + different port
        proxy_pass http://127.0.0.1:3101/asset;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

> Optional: You can also add `listen 80` servers for each domain to redirect HTTP → HTTPS.

---

## 3. How It Works

### 3.1 Main Domain (`stackdev.cloud`)

```nginx
server_name stackdev.cloud;

location / {
    proxy_pass http://127.0.0.1:3100;
}
```

- Any request like:
  - `https://stackdev.cloud/`
  - `https://stackdev.cloud/login`
  - `https://stackdev.cloud/dashboard`
- Is forwarded to `http://127.0.0.1:3100/...`.

### 3.2 API Subdomain (`api.stackdev.cloud` – Same Service)

```nginx
server_name api.stackdev.cloud;

location / {
    proxy_pass http://127.0.0.1:3100/api;
}
```

- Any request like:
  - `https://api.stackdev.cloud/`
  - `https://api.stackdev.cloud/users`
  - `https://api.stackdev.cloud/posts/1`
- Is forwarded to:
  - `http://127.0.0.1:3100/api/`
  - `http://127.0.0.1:3100/api/users`
  - `http://127.0.0.1:3100/api/posts/1`

So **one codebase on port 3100** serves both:

- Web pages (via `stackdev.cloud`)
- API endpoints (via `api.stackdev.cloud`, mapped to `/api` path)

### 3.3 Assets Subdomain (`asset.stackdev.cloud` – Different Service)

```nginx
server_name asset.stackdev.cloud;

location / {
    proxy_pass http://127.0.0.1:3101/asset;
}
```

- Any request like:
  - `https://asset.stackdev.cloud/`
  - `https://asset.stackdev.cloud/logo.png`
  - `https://asset.stackdev.cloud/js/app.js`
- Is forwarded to:
  - `http://127.0.0.1:3101/asset/`
  - `http://127.0.0.1:3101/asset/logo.png`
  - `http://127.0.0.1:3101/asset/js/app.js`

So **a different service** on port `3101` handles assets, completely independent from the app on port `3100`.

---

## 4. Required DNS & Certificates

### 4.1 DNS Records

You should have at least:

- `stackdev.cloud` → A / AAAA record to your server IP
- `api.stackdev.cloud` → A / AAAA record to the same server IP
- `asset.stackdev.cloud` → A / AAAA record to the same server IP

Example (IPv4 A records):

- `stackdev.cloud    A   191.10.1.0`
- `api.stackdev.cloud   A   191.10.1.0`
- `asset.stackdev.cloud A   191.10.1.0`

### 4.2 Certificates (Let’s Encrypt Example)

You can obtain certificates like:

```bash
# For main domain
sudo certbot certonly --nginx -d stackdev.cloud

# For API subdomain
sudo certbot certonly --nginx -d api.stackdev.cloud

# For Assets subdomain
sudo certbot certonly --nginx -d asset.stackdev.cloud
```

This will generate:

- `/etc/letsencrypt/live/stackdev.cloud/`
- `/etc/letsencrypt/live/api.stackdev.cloud/`
- `/etc/letsencrypt/live/asset.stackdev.cloud/`

---

## 5. Enabling and Testing the Configuration

### 5.1 Enable the Site

Create a symlink from `sites-available` to `sites-enabled`:

```bash
sudo ln -s /etc/nginx/sites-available/stackdev-subdomains.conf /etc/nginx/sites-enabled/stackdev-subdomains.conf
```

(Optional) Remove the default Nginx site:

```bash
sudo rm /etc/nginx/sites-enabled/default
```

### 5.2 Test Nginx Configuration

```bash
sudo nginx -t
```

You should see:

```text
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

### 5.3 Reload Nginx

```bash
sudo systemctl reload nginx
# or
sudo systemctl restart nginx
```

---

## 6. Verifying Each Subdomain

Use `curl` or a browser:

```bash
# Main app
curl -I https://stackdev.cloud/

# API
curl -I https://api.stackdev.cloud/

# Assets
curl -I https://asset.stackdev.cloud/
```

If your services are running correctly, you should receive valid HTTP statuses (200, 301, 302, etc.).

---

## 7. Adding More Subdomains

To add another subdomain, e.g. `admin.stackdev.cloud` using a different service:

1. Add a new `server` block:

```nginx
server {
    listen 443 ssl;
    listen [::]:443 ssl;
    server_name admin.stackdev.cloud;

    ssl_certificate /etc/letsencrypt/live/admin.stackdev.cloud/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/admin.stackdev.cloud/privkey.pem;

    location / {
        proxy_pass http://127.0.0.1:3200;  # admin service
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

2. Add DNS `admin.stackdev.cloud` → server IP.
3. Obtain SSL certificate for `admin.stackdev.cloud`.
4. Test (`nginx -t`) and reload Nginx.

---

## 8. Common Headers in Reverse Proxy

Every `location` uses these headers:

```nginx
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto $scheme;
```

They ensure:

- The backend knows the original **host**.
- You keep track of the **real client IP**.
- You know the original **protocol (HTTP/HTTPS)**.

Many frameworks (Express, Django, Laravel, etc.) rely on these headers when running behind a reverse proxy.

---

## 9. Summary

With this configuration:

- **Multiple subdomains** (`stackdev.cloud`, `api.stackdev.cloud`, `asset.stackdev.cloud`) are handled by **one Nginx instance**.
- You can:
  - Map different subdomains to **the same service** (with different paths).
  - Map other subdomains to **different services** on different ports.
- Each subdomain can have its own **SSL certificate** and configuration.
- This pattern is ideal for:
  - Splitting frontend / API / assets.
  - Organizing microservices under clean subdomain names.

For more details, see the official [Nginx HTTP server configuration docs](https://nginx.org/en/docs/http/ngx_http_core_module.html#server).
