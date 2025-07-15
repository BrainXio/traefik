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

5. **Migrate Existing Certificates (if needed)**
   If switching to Docker volumes for certificates and you have existing files in `./letsencrypt`, copy them to the volume:
   ```bash
   docker run --rm -v traefik_certs:/letsencrypt -v $(pwd)/letsencrypt:/source busybox cp /source/*.json /letsencrypt
   ```
   If starting fresh or encountering issues, remove the local files or volume:
   ```bash
   rm -f letsencrypt/acme-staging.json letsencrypt/acme-prod.json
   docker volume rm traefik_certs
   ```

6. **Start Traefik**
   Run Traefik with the DNS configuration:
   ```bash
   docker compose -f docker-compose.extend.cloudflare-dns.yml up -d
   ```
   To use Docker volumes for certificates and logs:
   ```bash
   docker compose -f docker-compose.extend.cloudflare-dns.yml -f docker-compose.extend.volumes.yml up -d
   ```
   For HTTP challenge:
   ```bash
   docker compose -f docker-compose.yml -f docker-compose.extend.volumes.yml up -d
   ```
   To enable healthchecks:
   ```bash
   docker compose -f docker-compose.extend.cloudflare-dns.yml -f docker-compose.extend.volumes.yml -f docker-compose.extend.healthcheck.yml up -d
   ```

7. **Verify the Dashboard and Healthcheck**
   For staging (`TRAEFIK_CONFIG=cloudflare-dns-staging`), visit `https://<TRAEFIK_DOMAIN>` (e.g., `https://example.com`) with `admin` and the password from `secrets/traefik_dashboard_users.txt`. For local testing:
   ```bash
   echo "127.0.0.1 example.com" >> /etc/hosts
   ```
   Check health:
   ```bash
   docker inspect --format '{{.State.Health.Status}}' traefik
   ```
   Expect `healthy` if Traefik is running correctly. In production (`TRAEFIK_CONFIG=cloudflare-dns-prod`), the dashboard is disabled.

### Next Steps
- Check `docs/using-traefik.md` for routing and debugging tips.
- Add features like metrics with `docker-compose.extend.metrics.yml`.
- For production with DNS, set `TRAEFIK_DOMAIN` to a public domain and `TRAEFIK_CONFIG=cloudflare-dns-prod`.

## Troubleshooting
- **Dashboard not loading in staging?**
  - Check `TRAEFIK_DOMAIN` in `.env` or `/etc/hosts`.
  - Regenerate credentials:
    ```bash
    echo "admin:$(openssl passwd -apr1 yourpassword)" > secrets/traefik_dashboard_users.txt
    ```
  - Verify ports: `sudo netstat -tuln | grep ':80\|:443'`.
- **Healthcheck failing?**
  - Check logs:
    ```bash
    docker run --rm -v traefik_logs:/logs busybox cat /logs/traefik.log
    ```
  - Test healthcheck:
    ```bash
    docker exec traefik traefik healthcheck
    ```
  - Ensure ping is enabled in the config.
- **Certificate issues?**
  - Ensure domain resolves (DNS-01).
  - Clear certificates:
    ```bash
    rm -f letsencrypt/acme-staging.json letsencrypt/acme-prod.json
    docker compose -f docker-compose.extend.cloudflare-dns.yml up -d
    ```
  - With volumes:
    ```bash
    docker volume rm traefik_certs
    docker compose -f docker-compose.extend.cloudflare-dns.yml -f docker-compose.extend.volumes.yml up -d
    ```
- **Need help?** Open a GitHub issue or check [Traefik Documentation](https://doc.traefik.io/traefik/).

## License
This project is licensed under the MIT License. See `LICENSE` for details.