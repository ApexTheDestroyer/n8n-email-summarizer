# n8n Physical Server Setup Guide

This guide documents the end-to-end deployment of **n8n** on an internal physical server using Docker, Nginx, HTTPS, SSE push, and Google OAuth for Gmail credentials.

> **Important security note:** Never commit Google OAuth client secrets, passwords, API keys, private keys, or other credentials to GitHub. This guide uses a placeholder for the Google Client Secret. Store the real secret securely.

---

## Architecture Overview

```text
User Browser
    |
    | HTTPS :8443
    v
Nginx Reverse Proxy
    |
    | HTTP :5678 (localhost only)
    v
n8n Docker Container
    |
    +--> /home/node/.n8n  (persistent data)
    |
    +--> Google OAuth / Gmail
```

### Server Details

| Component | Configuration |
|---|---|
| Server IP | `192.1.5.34` |
| SSH User | `mis` |
| n8n Domain | `192.1.5.34.sslip.io` |
| HTTPS Port | `8443` |
| n8n Internal Port | `5678` |
| Reverse Proxy | Nginx |
| Push Backend | SSE |
| Timezone | `Asia/Manila` |
| Container | `n8n_email_summarizer` |
| Project Directory | `~/n8n~project` |

---

# Step 1: Remote Server Access

```bash
ssh mis@192.1.5.34
```

---

# Step 2: System Update & Docker Setup

```bash
# Update system packages
sudo apt update && sudo apt upgrade -y

# Install Docker and essential utilities
sudo apt install --no-install-recommends -y docker.io docker-compose-plugin

# Grant Docker permissions to the 'mis' user
sudo usermod -aG docker mis

# Refresh group membership in the current shell
newgrp docker

# Verify Docker Compose installation
docker compose version
```

Optional verification:

```bash
docker --version
docker compose version
```

---

# Step 3: Project Directory & Permissions

```bash
# Create directory structure
mkdir -p ~/n8n~project/n8n_data

# Move into the project directory
cd ~/n8n~project

# Assign ownership to n8n's internal container user (UID 1000)
sudo chown -R 1000:1000 ./n8n_data
```

Verify:

```bash
ls -ld ~/n8n~project
ls -ld ~/n8n~project/n8n_data
```

---

# Step 4: Self-Signed SSL Certificate Generation

Create the SSL directory:

```bash
sudo mkdir -p /etc/nginx/ssl
```

Generate a 2048-bit RSA self-signed certificate valid for one year:

```bash
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/nginx/ssl/n8n.key \
  -out /etc/nginx/ssl/n8n.crt \
  -subj "/C=PH/ST=CentralLuzon/L=Angeles/O=Company/OU=IT/CN=192.1.5.34.sslip.io"
```

Verify the files:

```bash
sudo ls -l /etc/nginx/ssl/
```

> Because this is self-signed, browsers will normally display a certificate warning. Verify the hostname and server before proceeding.

---

# Step 5: Configure `docker-compose.yml`

Create the Compose file:

```bash
cat <<'EOF' > ~/n8n~project/docker-compose.yml
services:
  n8n-email-app:
    image: docker.n8n.io/n8nio/n8n:latest
    container_name: n8n_email_summarizer
    restart: always

    ports:
      - "127.0.0.1:5678:5678"

    environment:
      - N8N_HOST=192.1.5.34.sslip.io
      - N8N_PORT=5678
      - N8N_PROTOCOL=https
      - NODE_ENV=production
      - WEBHOOK_URL=https://192.1.5.34.sslip.io:8443/
      - N8N_EDITOR_BASE_URL=https://192.1.5.34.sslip.io:8443/
      - N8N_PUSH_BACKEND=sse
      - GENERIC_TIMEZONE=Asia/Manila

    volumes:
      - ./n8n_data:/home/node/.n8n
EOF
```

Start n8n:

```bash
cd ~/n8n~project
docker compose up -d
```

Check status:

```bash
docker compose ps
```

View logs:

```bash
docker compose logs -f n8n-email-app
```

Press `Ctrl+C` to stop following logs.

---

# Step 6: Configure Nginx Reverse Proxy

Install Nginx:

```bash
sudo apt install -y nginx
```

Create the site configuration:

```bash
cat <<'EOF' | sudo tee /etc/nginx/sites-available/n8n > /dev/null
server {
    listen 8443 ssl;
    server_name 192.1.5.34.sslip.io 192.1.5.34;

    ssl_certificate /etc/nginx/ssl/n8n.crt;
    ssl_certificate_key /etc/nginx/ssl/n8n.key;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;

    client_max_body_size 50M;

    location / {
        proxy_pass http://127.0.0.1:5678;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto https;

        # Event streaming / SSE settings
        proxy_set_header Connection '';
        proxy_http_version 1.1;
        proxy_buffering off;
        proxy_cache off;
        proxy_connect_timeout 24h;
        proxy_send_timeout 24h;
        proxy_read_timeout 24h;
    }
}
EOF
```

Enable the configuration:

```bash
sudo ln -sf /etc/nginx/sites-available/n8n /etc/nginx/sites-enabled/n8n
```

Remove the default site:

```bash
sudo rm -f /etc/nginx/sites-enabled/default
```

Allow port `8443` through UFW:

```bash
sudo ufw allow 8443/tcp
```

Test and restart Nginx:

```bash
sudo nginx -t
sudo systemctl restart nginx
```

Check status:

```bash
sudo systemctl status nginx --no-pager
```

---

# Step 7: Access n8n Web UI & Account Setup

Navigate to:

```text
https://192.1.5.34.sslip.io:8443
```

Because the deployment uses a self-signed certificate, your browser may show a security warning.

1. Open the n8n URL.
2. Verify that the hostname is correct.
3. Proceed through the browser certificate warning.
4. Complete the **Set up owner account** form.
5. Sign in to n8n.

---

# Step 8: Google Cloud Console OAuth Configuration

Open the Google Cloud Console Credentials page:

https://console.cloud.google.com/apis/credentials

Select the existing **Web Application** OAuth client.

Under **Authorized redirect URIs**, add this exact callback URL:

```text
https://192.1.5.34.sslip.io:8443/rest/oauth2-credential/callback
```

Click **Save**.

### Client ID

Use the existing Google OAuth Client ID:

```text
48194906208-rhfpctmb00lpm3fu9sum6br5mvlde7qa.apps.googleusercontent.com
```

> **Do not commit the Client Secret to GitHub.**

---

# Step 9: Connect Gmail Credential in n8n

Inside n8n:

1. Open a **Gmail** node.
2. Create or select the Gmail OAuth credential.
3. Set the OAuth Redirect URL to:

```text
https://192.1.5.34.sslip.io:8443/rest/oauth2-credential/callback
```

4. Enter the Google OAuth Client ID.
5. Enter the Google OAuth Client Secret securely.
6. Click **Sign in with Google** / **Connect**.
7. Complete the Google authentication flow.
8. Confirm the credential reports a successful connection.

### OAuth Values

```text
OAuth Redirect URL:
https://192.1.5.34.sslip.io:8443/rest/oauth2-credential/callback

Client ID:
48194906208-rhfpctmb00lpm3fu9sum6br5mvlde7qa.apps.googleusercontent.com

Client Secret:
<STORE SECURELY - DO NOT COMMIT TO GITHUB>
```

---

# Step 10: Verify the Deployment

### Check Docker

```bash
docker --version
docker compose version
```

### Check n8n Container

```bash
cd ~/n8n~project
docker compose ps
docker compose logs --tail=100 n8n-email-app
```

### Check Nginx

```bash
sudo nginx -t
sudo systemctl status nginx --no-pager
```

### Check Listening Ports

```bash
sudo ss -tulpn | grep -E '8443|5678'
```

Expected behavior:

- Nginx listens on `8443`.
- n8n listens on `127.0.0.1:5678` and is therefore not directly exposed externally.

### Test the Web URL

```text
https://192.1.5.34.sslip.io:8443
```

---

# Step 11: Useful Maintenance Commands

### Restart n8n

```bash
cd ~/n8n~project
docker compose restart
```

### Stop n8n

```bash
cd ~/n8n~project
docker compose down
```

### Start n8n

```bash
cd ~/n8n~project
docker compose up -d
```

### View n8n Logs

```bash
cd ~/n8n~project
docker compose logs -f n8n-email-app
```

### Check Nginx Access Logs

```bash
sudo tail -f /var/log/nginx/access.log
```

### Check Nginx Error Logs

```bash
sudo tail -f /var/log/nginx/error.log
```

---

# Troubleshooting

## n8n Shows `Connection lost`

Confirm the SSE/reverse-proxy settings exist in the Nginx site configuration:

```nginx
proxy_set_header Connection '';
proxy_http_version 1.1;
proxy_buffering off;
proxy_cache off;
proxy_connect_timeout 24h;
proxy_send_timeout 24h;
proxy_read_timeout 24h;
```

Then test and restart Nginx:

```bash
sudo nginx -t
sudo systemctl restart nginx
```

Also verify n8n:

```bash
cd ~/n8n~project
docker compose ps
docker compose logs --tail=100 n8n-email-app
```

## n8n Container Is Not Starting

```bash
cd ~/n8n~project
docker compose logs --tail=200 n8n-email-app
```

Check the ownership of the persistent data directory:

```bash
sudo chown -R 1000:1000 ~/n8n~project/n8n_data
```

Restart the stack:

```bash
cd ~/n8n~project
docker compose up -d
```

## Google OAuth Redirect URI Error

Verify that Google Cloud Console contains exactly:

```text
https://192.1.5.34.sslip.io:8443/rest/oauth2-credential/callback
```

The protocol, hostname, port, and callback path must match the value used by n8n.

## Port 8443 Is Not Reachable

Check Nginx and the listening port:

```bash
sudo systemctl status nginx --no-pager
sudo ss -tulpn | grep 8443
```

Check UFW:

```bash
sudo ufw status
```

Allow the port when necessary:

```bash
sudo ufw allow 8443/tcp
```

---

# Security Notes

## Never Commit Secrets

Do not commit any of the following to GitHub:

- Google OAuth Client Secret
- n8n encryption key
- Passwords
- API keys
- SSH private keys
- SSL private keys
- Database passwords
- Session secrets

Keep these private server-side files out of Git:

```text
/etc/nginx/ssl/n8n.key
~/n8n~project/n8n_data/
```

The `n8n_data` directory contains persistent n8n data and should not be uploaded to GitHub.

## Recommended `.gitignore`

```gitignore
# n8n persistent data
n8n_data/

# Environment files
.env
.env.*
!.env.example

# Local/private key material
*.key
*.pem

# Logs
*.log
```

---

# Final Deployment Checklist

- [ ] SSH access to `192.1.5.34` works.
- [ ] Docker is installed.
- [ ] Docker Compose works.
- [ ] `~/n8n~project` exists.
- [ ] `n8n_data` has correct ownership.
- [ ] SSL certificate and key exist under `/etc/nginx/ssl/`.
- [ ] n8n Docker container is running.
- [ ] n8n is bound to `127.0.0.1:5678`.
- [ ] Nginx is listening on `8443`.
- [ ] UFW allows TCP port `8443`.
- [ ] `https://192.1.5.34.sslip.io:8443` opens successfully.
- [ ] Owner account has been created in n8n.
- [ ] Google OAuth redirect URI matches exactly.
- [ ] Gmail OAuth credential connects successfully.
- [ ] SSE/persistent UI connection works without repeated `Connection lost` errors.
- [ ] No secrets are committed to GitHub.

---

# Suggested Repository Structure

```text
n8n~project/
├── N8N_SETUP.md
├── docker-compose.yml
├── .gitignore
└── n8n_data/              # Do NOT commit
```

Commit only safe documentation and configuration. Keep credentials, private SSL keys, and persistent n8n data on the physical server.
