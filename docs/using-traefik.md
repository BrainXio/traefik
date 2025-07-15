# Using Traefik

This guide walks you through setting up and using the Traefik reverse proxy in the `brainxio/traefik` repository. It’s written for beginners, so don’t worry if you’re new to Traefik or Docker!

## What You’ll Learn
- How to configure Traefik for secure HTTPS routing.
- How to route traffic to services (like an Nginx web server) and debugging.
- How to use Docker volumes for certificates and logs.

## Step-by-Step Setup

1. **Clone the Repository**
   ```bash
   git clone https://github.com/brainxio/traefik.git
   cd traefik
   ```

2. **Configure Environment Variables**
   Traefik uses a `.env` file for non-sensitive settings. Copy the example file:
   ```bash
   cp .env.example .env
   ```
   Open `.env` in a text editor and set:
   - `TRAEFIK_DOMAIN`: The domain for the Traefik dashboard (e.g., `example.com` for testing or `yourdomain.com` for production).
   - `DOCKER_SOCK`: The Docker socket path (default: `/var/run/docker.sock`). For rootless Docker, set to `/var/run/user/<UID>/docker.sock` (e.g., `/var/run/user/1000/docker.sock`).
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
   **Note**: Ensure the token has "Zone: DNS: Edit" permissions in Cloudflare.

5. **Migrate Existing Certificates (if needed)**
   If switching to Docker volumes for certificates and you have existing files in `./letsencrypt`, copy them to the volume:
   ```bash
   docker run --rm -v traefik_certs:/letsencrypt -v $(pwd)/letsencrypt:/source busybox cp /source/*.json /letsencrypt
   ```
   If starting fresh or encountering issues, remove the local files:
   ```bash
   rm -f letsencrypt/acme-staging.json letsencrypt/acme-prod.json
   ```

6. **Start Traefik**
   Launch Traefik with the DNS configuration:
   ```bash
   docker compose -f docker-compose.extend.cloudflare-dns.yml up -d
   ```
   To use Docker volumes for certificates and logs (instead of local disk):
   ```bash
   docker compose -f docker-compose.extend.cloudflare-dns.yml -f docker-compose.extend.volumes.yml up -d
   ```
   For HTTP challenge (base setup):
   ```bash
   docker compose -f docker-compose.yml -f docker-compose.extend.volumes.yml up -d
   ```

7. **Access the Dashboard**
   For staging (`TRAEFIK_CONFIG=cloudflare-dns-staging` or default), open your browser and go to `https://<TRAEFIK_DOMAIN>` (e.g., `https://example.com`). Enter the username (`admin`) and password from `secrets/traefik_dashboard_users.txt`. For local testing, add to `/etc/hosts`:
   ```bash
   127.0.0.1 example.com
   ```
   **Note**: The staging endpoint may issue untrusted certificates, so you may see a browser warning. In production (`TRAEFIK_CONFIG=cloudflare-dns-prod`), the dashboard is disabled for security.

## Routing Services (e.g., Nginx)
To route traffic to a service like an Nginx web server, add it to the `traefik-net` network and configure Traefik labels. Here’s an example:

1. **Add Nginx to an extend file** (e.g., `docker-compose.extend.nginx.yml`):
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

2. **Update `/etc/hosts` for local testing** (if using `nginx.example.com`):
   ```bash
   127.0.0.1 nginx.example.com
   ```

3. **Restart Traefik and Nginx**:
   ```bash
   docker compose -f docker-compose.extend.cloudflare-dns.yml -f docker-compose.extend.nginx.yml up -d
   ```

4. **Access Nginx**:
   Open your browser and visit `https://nginx.example.com`. You should see the default Nginx welcome page. Expect a certificate warning if using the staging endpoint.

## Debugging Tips
- **Dashboard not loading in staging?**
  - Ensure `TRAEFIK_DOMAIN` is set correctly in `.env` or defaults to `example.com` and resolves to your server (e.g., add `127.0.0.1 example.com` to `/etc/hosts`).
  - Verify basic auth credentials in `secrets/traefik_dashboard_users.txt`. Regenerate if needed:
    ```bash
    echo "admin:$(openssl passwd -apr1 yourpassword)" > secrets/traefik_dashboard_users.txt
    ```
  - Ensure ports 80 and 443 are open (`sudo netstat -tuln | grep ':80\|:443'`).
- **Dashboard inaccessible in production?** This is expected, as the dashboard is disabled (`TRAEFIK_CONFIG=cloudflare-dns-prod`).
- **Certificate issues?**
  - If you see validation errors, ensure the domain is publicly resolvable and DNS records are correct in Cloudflare.
  - If Traefik fails DNS validation, remove certificate files and restart:
    ```bash
    rm -f letsencrypt/acme-staging.json letsencrypt/acme-prod.json
    docker compose -f docker-compose.extend.cloudflare-dns.yml up -d
    ```
  - With volumes, inspect or remove the volume and restart:
    ```bash
    docker volume rm traefik_certs
    docker compose -f docker-compose.extend.cloudflare-dns.yml -f docker-compose.extend.volumes.yml up -d
    ```
- **Log access with volumes?**
  - View logs from the volume:
    ```bash
    docker run --rm -v traefik_logs:/logs busybox cat /logs/traefik.log
    ```
- **View Traefik logs**:
   ```bash
   docker logs traefik
   ```
- **Need help?** Open an issue on GitHub or check the [Traefik Documentation](https://doc.traefik.io/traefik/).

## License
This project is licensed under the MIT License. See `LICENSE` for details.