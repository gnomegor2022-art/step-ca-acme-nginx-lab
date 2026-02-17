# step-ca setup (CA host)

## Init
Example:
- Name: Lab-CA
- DNS/IP: ca.local,192.168.64.10
- Bind: :443
- Provisioner: admin

## Enable ACME provisioner
Commands:
- `sudo step ca provisioner add acme --type ACME --ca-config /root/.step/config/ca.json`
- `sudo systemctl restart step-ca`

## Verify
- `curl -k https://ca.local/roots.pem | head`
- `curl -k https://ca.local/acme/acme/directory`

