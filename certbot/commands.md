# Certbot (Web host)

## Trust step-ca root
Download roots:
- `curl -k https://ca.local:443/roots.pem -o /tmp/roots.pem`

Split and add to OS trust store (Ubuntu):
- `sudo csplit -f /tmp/stepca- /tmp/roots.pem '/END CERTIFICATE/+1' '{*}'`
- `sudo cp /tmp/stepca-00 /usr/local/share/ca-certificates/stepca-root.crt`
- `sudo cp /tmp/stepca-01 /usr/local/share/ca-certificates/stepca-intermediate.crt || true`
- `sudo update-ca-certificates`

## Issue cert
If nginx uses :80, stop it first:
- `sudo systemctl stop nginx`

Request:
- `sudo certbot certonly --server https://ca.local/acme/acme/directory --standalone -d myhost.local --register-unsafely-without-email --agree-tos`

Cert paths:
- `/etc/letsencrypt/live/myhost.local/fullchain.pem`
- `/etc/letsencrypt/live/myhost.local/privkey.pem`

