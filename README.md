# Gotenberg on Ubuntu 24.04 (Ansible) — HTTPS + Let's Encrypt + IP allow-list

Architecture: `client --443/TLS--> nginx --> 127.0.0.1:3000 (Gotenberg, Docker)`

1. Create a DNS record for your domain pointing at the server.
2. On your local workstation: `ansible-galaxy collection install community.docker community.general`
3. In `inventory.ini` set the host plus `gotenberg_domain` and `letsencrypt_email` (under `[gotenberg:vars]`); edit `allowed_ips` and other options in `group_vars/gotenberg.yml`.
4. `ansible-playbook -i inventory.ini deploy-gotenberg.yml`

Test from an allowed IP:

    curl https://YOUR_DOMAIN/health

Notes
- HTTPS (443) is open only to `allowed_ips` (UFW + nginx `allow/deny`).
- HTTP (80) must stay open to everyone: Let's Encrypt validates from many
  IPs. Port 80 only serves the ACME challenge and redirects to HTTPS.
  If you can't allow that, switch to the DNS-01 challenge instead.
- Renewal is automatic (`certbot.timer`); nginx reloads via a deploy hook.
- Test first with `-e letsencrypt_staging=true`. To move to a real cert after
  that, delete `/etc/letsencrypt/live/<domain>` (and `archive/`, `renewal/`) on the
  server, then re-run with staging off.
- Gotenberg listens on localhost only, so Docker's firewall bypass is not an issue.
