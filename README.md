# Step-CA + ACME + Nginx Lab

Personal DevOps training project.

## 🔧 Stack
- Smallstep step-ca (ACME server)
- Certbot (ACME client)
- Nginx
- macOS + Linux VM

## 📌 Goal
Issue a certificate from internal step-ca via ACME and configure nginx to use it.

## 🏗 Architecture

CA Server: 192.168.64.10 (ca.local)  
Web Server: 192.168.64.20 (myhost.local)

## 🔐 Certificate Flow

1. step-ca runs ACME directory
2. certbot requests certificate
3. certificate saved to /etc/letsencrypt/
4. nginx uses fullchain.pem + privkey.pem

## 🚀 Result

HTTPS working on https://myhost.local

