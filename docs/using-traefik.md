# Using Traefik

This guide walks you through setting up and using the Traefik reverse proxy in the `brainxio/traefik` repository. It’s written for beginners, so don’t worry if you’re new to Traefik or Docker!

## What You’ll Learn
- Configure Traefik for secure HTTPS routing.
- Route traffic to services and debug issues.
- Use Docker volumes and healthchecks.

## Step-by-Step Setup

1. **Clone the Repository**
   ```bash
   git clone https://github.com/brainxio/traefik.git
   cd traefik
   ```

2. **Configure Environment Variables**
   Copy the example file:
   ```bash
   cp .env.example .env
   ```
   Edit `.env` to set:
   - `TRAEFIK_DOMAIN`: Dashboard domain (e.g., `example.com` or `yourdomain.com`).
   - `DOCKER_SOCK`: Docker socket path (default: `/var/run/docker.sock`; for rootless, use `/var/run/user/<UID>/docker.sock`).
   - `TRAEFIK_CERTIFICATESRESOLVERS_LETSENCRYPT_ACME_EMAIL`: Email for Let’s Encrypt (default: `user@example.com`).
   - `TRAEFIK_CONFIG`: Config file (default: `cloudflare-dns-staging`; set to `cloudflare-dns-prod` for production).

3. **Set Up Basic Auth for Staging Dashboard**
   Create credentials:
   ```bash
   mkdir secrets
   echo "admin:$(openssl passwd -apr1 yourpassword)" > secrets/traefik_dashboard_users.txt
   ```
   Verify:
   ```bash
   cat secrets/traefik_dashboard_users.txt
   ```
   **Note**: In production (`TRAEFIK_CONFIG=cloudflare-dns-prod`), the dashboard is disabled.

4. **Set Up Cloudflare API Token for DNS**
   Create:
   ```bash
   echo "your-cloudflare-api-token" > secrets/cloudflare_api_token.txt
   ```
   **Note**: Token needs "Zone: DNS: Edit" permissions.

5. **Migrate Existing Certificates (if needed)**
   Copy certificates to the volume:
   ```bash
   docker run --rm -v traefik_certs:/letsencrypt -v $(pwd)/letsencrypt:/source busybox cp /source/*.json /letsencrypt
   ```
   To start fresh:
   ```bash
   rm -f letsencrypt/acme-staging.json letsencrypt/acme-prod.json
   docker volume rm traefik_certs
   ```

6. **Start Traefik**
   For DNS:
   ```bash
   docker compose -f docker-compose.extend.cloudflare-dns.yml up -d
   ```
   With volumes:
   ```bash
   docker compose -f docker-compose.extend.cloudflare-dns.yml -f docker-compose.extend.volumes.yml up -d
   ```
   For HTTP:
   ```bash
   docker compose -f docker-compose.yml -f docker-compose.extend.volumes.yml up -d
   ```
   With healthchecks:
   ```bash
   docker compose -f docker-compose.extend.cloudflare-dns.yml -f docker-compose.extend.volumes.yml -f docker-compose.extend.healthcheck.yml up -d
   ```

7. **Verify the Dashboard and Healthcheck**
   For staging (`TRAEFIK_CONFIG=cloudflare-dns-staging`), visit `https://<TRAEFIK_DOMAIN>` with `admin` and password from `secrets/traefik_dashboard_users.txt`. For local testing:
   ```bash
   echo "127.0.0.1 example.com" >> /etc/hosts
   ```
   Check health:
   ```bash
   docker inspect --format '{{.State.Health.Status}}' traefik
   ```
   Test healthcheck:
   ```bash
   docker exec traefik traefik healthcheck
   ```
   **Note**: Staging may show certificate warnings; production disables the dashboard.

## Routing Services (e.g., Nginx)
1. **Add to an extend file** (e.g., `docker-compose.extend.nginx.yml`):
   ```yaml
   networks:
     traefik-net:
       name: traefik-net
   services:
     nginx:
       image: nginx:latest
       labels:
         - "traefik.enable=true"
         - "traefik.http.routers.nginx.rule=Host(`nginx.example.com`)"
         - "traefik.http.routers.nginx.entrypoints=websecure"
         - "traefik.http.routers.nginx.tls.certresolver=letsencrypt"
       networks:
         - traefik-net
   ```

2. **Update `/etc/hosts`**:
   ```bash
   echo "127.0.0.1 nginx.example.com" >> /etc/hosts
   ```

3. **Restart**:
   ```bash
   docker compose -f docker-compose.extend.cloudflare-dns.yml -f docker-compose.extend.nginx.yml up -d
   ```

4. **Access**: Visit `https://nginx.example.com`.

## Debugging Tips
- **Dashboard issues?**
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
- **Logs**:
  - Volume logs:
    ```bash
    docker run --rm -v traefik_logs:/logs busybox cat /logs/traefik.log
    ```
  - Container logs:
    ```bash
    docker logs traefik
    ```

## Adding Features
- Use `docker-compose.extend.volumes.yml` for volumes.
- Use `docker-compose.extend.healthcheck.yml` for healthchecks.
- Next: Metrics with `docker-compose.extend.metrics.yml`.

## Need Help?
- See [Traefik Documentation](https://doc.traefik.io/traefik/).
- Open a GitHub issue or join the Traefik community.