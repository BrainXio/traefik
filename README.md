# Traefik Reverse Proxy

Welcome to the `brainxio/traefik` repository! This project sets up a Traefik reverse proxy using Docker Compose to securely route traffic to your services with automatic HTTPS certificates from Let’s Encrypt. It’s designed to be easy to use, even if you’re new to Traefik or Docker.

## What is Traefik?

Traefik is a modern reverse proxy that makes it simple to route web traffic to your services. It automatically handles HTTPS certificates, so your connections are secure, and it works seamlessly with Docker to discover and route to your containers.

## Getting Started

### Prerequisites
- **Docker** and **Docker Compose** installed on your system.
- A domain name (e.g., `yourdomain.com`) for production certificates, or use a custom domain for testing with the staging endpoint.
- A valid email address for Let’s Encrypt certificate notifications (optional if using the default).
- A Cloudflare API token for DNS-01 challenge (required for staging and production DNS validation).
- Basic familiarity with the command line.

### Setup Steps

1. **Clone the Repository**
   ```bash
   git clone https://github.com/brainxio/traefik.git
   cd traefik
   ```

2. **Create the `.env` File (Optional)**
   Copy the example file to customize settings:
   ```bash
   cp .env.example .env
   ```
   Edit `.env` to set (defaults are used if not specified):
   - `TRAEFIK_DOMAIN`: The domain for the Traefik dashboard (default: `example.com` for testing or set to `yourdomain.com` for production). Supports wildcard subdomains (e.g., `*.yourdomain.com`) for DNS-01 certificates.
   - `DOCKER_SOCK`: The Docker socket path (default: `/var/run/docker.sock`; for rootless Docker, use `/var/run/user/<UID>/docker.sock`, e.g., `/var/run/user/1000/docker.sock`).
   - `TRAEFIK_CERTIFICATESRESOLVERS_LETSENCRYPT_ACME_EMAIL`: A valid, deliverable email for Let’s Encrypt (default: `user@example.com`).
   - `TRAEFIK_CONFIG`: The Traefik configuration file (default: `cloudflare-dns-staging` for staging; set to `cloudflare-dns-prod` for production).

3. **Set Up Basic Auth for Staging Dashboard**
   Create a `secrets/` directory and generate a username:password pair in htpasswd format:
   ```bash
   mkdir secrets
   echo "admin:$(openssl passwd -apr1 yourpassword)" > secrets/traefik_dashboard_users.txt
   ```
   Replace `yourpassword` with a strong password. Verify the file:
   ```bash
   cat secrets/traefik_dashboard_users.txt
   ```
   **Note**: The `secrets/` directory is ignored by `.gitignore` and not committed. In production (`TRAEFIK_CONFIG=cloudflare-dns-prod`), the dashboard is disabled, so basic auth is not needed.

4. **Set Up Cloudflare API Token for DNS**
   Create a `secrets/cloudflare_api_token.txt` file with your Cloudflare API token:
   ```bash
   echo "your-cloudflare-api-token" > secrets/cloudflare_api_token.txt
   ```
   **Note**: Ensure the token has "Zone: DNS: Edit" permissions in Cloudflare and keep it secure (ignored by `.gitignore`).

5. **Clear Existing Certificates (if needed)**
   If switching between configurations or encountering certificate issues, remove the existing certificate files:
   ```bash
   rm -f letsencrypt/acme-staging.json letsencrypt/acme-prod.json
   ```

6. **Start Traefik**
   Run the following command to start Traefik with the DNS configuration:
   ```bash
   docker compose -f docker-compose.extend.cloudflare-dns.yml up -d
   ```

7. **Access the Dashboard**
   For staging (`TRAEFIK_CONFIG=cloudflare-dns-staging` or default), open your browser and go to `https://<TRAEFIK_DOMAIN>` (e.g., `https://example.com`). Enter the username (`admin`) and password from `secrets/traefik_dashboard_users.txt`. For local testing, add to `/etc/hosts`:
   ```bash
   127.0.0.1 example.com
   ```
   **Note**: The staging endpoint may issue untrusted certificates, so you may see a browser warning. In production (`TRAEFIK_CONFIG=cloudflare-dns-prod`), the dashboard is disabled for security.

### Next Steps
- Check out `docs/using-traefik.md` for tips on routing services (like an Nginx web server) and debugging.
- Add more services by creating additional Docker Compose files (e.g., next steps include metrics and logging).
- For production with DNS, update `TRAEFIK_DOMAIN` to a public domain and set `TRAEFIK_CONFIG=cloudflare-dns-prod`.

## Troubleshooting
- **Dashboard not loading in staging?**
  - Ensure `TRAEFIK_DOMAIN` is set correctly in `.env` or defaults to `example.com` and resolves to your server (e.g., add `127.0.0.1 example.com` to `/etc/hosts`).
  - Verify basic auth credentials in `secrets/traefik_dashboard_users.txt`. Regenerate if needed:
    ```bash
    echo "admin:$(openssl passwd -apr1 yourpassword)" > secrets/traefik_dashboard_users.txt
    ```
- **Dashboard inaccessible in production?** This is expected, as the dashboard is disabled (`TRAEFIK_CONFIG=cloudflare-dns-prod`).
- **Certificate issues?**
  - If you see validation errors, ensure the domain is publicly resolvable and DNS records are correct in Cloudflare.
  - If Traefik fails DNS validation, remove certificate files and restart:
    ```bash
    rm -f letsencrypt/acme-staging.json letsencrypt/acme-prod.json
    docker compose -f docker-compose.extend.cloudflare-dns.yml up -d
    ```
- **Need help?** Open an issue on GitHub or check the [Traefik Documentation](https://doc.traefik.io/traefik/).

## License
This project is licensed under the MIT License. See `LICENSE` for details.