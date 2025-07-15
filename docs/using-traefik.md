# Using Traefik

This guide walks you through setting up and using the Traefik reverse proxy in the `brainxio/traefik` repository. It’s written for beginners, so don’t worry if you’re new to Traefik or Docker!

## What You’ll Learn
- How to configure Traefik for secure HTTPS routing.
- How to route traffic to services like an Nginx web server.
- How to debug common issues.

## Step-by-Step Setup

### 1. Clone the Repository
Download the project to your server or local machine:
```bash
git clone https://github.com/brainxio/traefik.git
cd traefik
```

### 2. Configure Environment Variables
Traefik uses a `.env` file for non-sensitive settings. Copy the example file:
```bash
cp .env.example .env
```
Open `.env` in a text editor and set:
- `TRAEFIK_DOMAIN`: The domain for the Traefik dashboard (e.g., `traefik.local` for testing or `traefik.yourdomain.com` for production).
- `DOCKER_SOCK`: The Docker socket path (default: `/var/run/docker.sock`). For rootless Docker, set to `/var/run/user/<UID>/docker.sock` (e.g., `/var/run/user/1000/docker.sock`).
- `TRAEFIK_CERTIFICATESRESOLVERS_letsencrypt_ACME_EMAIL`: A valid, deliverable email for Let’s Encrypt (e.g., `user@yourdomain.com`).
- `TRAEFIK_CONFIG`: The Traefik configuration file (default: `http-staging` for staging). Set to `http-prod` for production.

Example `.env`:
```
TRAEFIK_DOMAIN=traefik.local
DOCKER_SOCK=/var/run/docker.sock
TRAEFIK_CERTIFICATESRESOLVERS_letsencrypt_ACME_EMAIL=user@yourdomain.com
TRAEFIK_CONFIG=http-staging
```

### 3. Set Up Basic Auth for Staging Dashboard
Create a `secrets/` directory and generate a username:password pair in htpasswd format:
```bash
mkdir secrets
echo "admin:$(openssl passwd -apr1 yourpassword)" > secrets/traefik_dashboard_users.txt
```
Replace `yourpassword` with a strong password. Verify the file:
```bash
cat secrets/traefik_dashboard_users.txt
```
**Note**: The `secrets/` directory is ignored by `.gitignore` and not committed. In production (`TRAEFIK_CONFIG=http-prod`), the dashboard is disabled, so basic auth is not needed.

### 4. Clear Existing Certificates (if needed)
If switching between staging and production or encountering certificate issues, remove the existing certificate files:
```bash
rm -f letsencrypt/acme-staging.json letsencrypt/acme-prod.json
```

### 5. Start Traefik
Launch Traefik with Docker Compose:
```bash
docker-compose up -d
```
This starts Traefik in the background, listening on ports 80 (HTTP) and 443 (HTTPS).

### 6. Verify the Dashboard
For staging (`TRAEFIK_CONFIG=http-staging`), open your browser and go to `https://<TRAEFIK_DOMAIN>` (e.g., `https://traefik.local`). Enter the username (`admin`) and password from `secrets/traefik_dashboard_users.txt`. For local testing, add to `/etc/hosts`:
```
127.0.0.1 traefik.local
```
**Note**: The staging endpoint issues untrusted certificates, so you may see a browser warning. In production (`TRAEFIK_CONFIG=http-prod`), the dashboard is disabled for security.

## Routing Services (e.g., Nginx)
To route traffic to a service like an Nginx web server, add it to the `traefik-net` network and configure Traefik labels. Here’s an example for Nginx:

1. **Add Nginx to `docker-compose.yml` or an extend file** (e.g., `docker-compose.extend.nginx.yml`):
   ```yaml
   services:
     nginx:
       image: nginx:latest
       networks:
         - traefik-net
       labels:
         - "traefik.enable=true"
         - "traefik.http.routers.nginx.rule=Host(`nginx.local`)"
         - "traefik.http.routers.nginx.entrypoints=websecure"
         - "traefik.http.routers.nginx.tls.certresolver=letsencrypt"
   networks:
     traefik-net:
       name: traefik-net
   ```

2. **Update `/etc/hosts` for local testing** (if using `nginx.local`):
   ```
   127.0.0.1 nginx.local
   ```

3. **Restart Traefik and Nginx**:
   ```bash
   docker-compose up -d
   ```

4. **Access Nginx**:
   Open your browser and visit `https://nginx.local`. You should see the default Nginx welcome page. Expect a certificate warning if using the staging endpoint.

## Debugging Tips
- **Dashboard not loading in staging?**
  - Check `TRAEFIK_DOMAIN` in `.env` matches your domain or `/etc/hosts` entry.
  - Verify basic auth credentials in `secrets/traefik_dashboard_users.txt`. Regenerate if needed:
    ```bash
    echo "admin:$(openssl passwd -apr1 yourpassword)" > secrets/traefik_dashboard_users.txt
    ```
  - Ensure ports 80 and 443 are open (`sudo netstat -tuln | grep ':80\|:443'`).
- **Dashboard inaccessible in production?** This is expected, as the dashboard is disabled (`TRAEFIK_CONFIG=http-prod`).
- **Certificate issues?**
  - If you see `Cannot issue for "traefik.local": Domain name does not end with a valid public suffix`, use a public domain (e.g., `yourdomain.com`) or keep `TRAEFIK_CONFIG=http-staging` for testing.
  - If Traefik uses the wrong endpoint, remove certificate files and restart:
    ```bash
    rm -f letsencrypt/acme-staging.json letsencrypt/acme-prod.json
    docker-compose up -d
    ```
  - Verify environment variables:
    ```bash
    echo $TRAEFIK_CERTIFICATESRESOLVERS_letsencrypt_ACME_EMAIL
    echo $TRAEFIK_CONFIG
    ```
    Update `.env` if needed and restart Traefik: `docker-compose up -d`.
- **Rootless Docker issues?**
  - Ensure `DOCKER_SOCK` is set to the correct path (e.g., `/var/run/user/1000/docker.sock`).
  - Verify the Docker socket is accessible: `docker info --format '{{.HostInfo.ID}}'`.
- **Service not routing?**
  - Confirm the service is on the `traefik-net` network.
  - Check Traefik labels in the service’s Docker Compose file.
- **View logs**:
  ```bash
  docker logs traefik
  ```

## Adding Features
You can extend Traefik with additional Docker Compose files:
- **Enable metrics**: Create `docker-compose.extend.metrics.yml` (see future guides in `docs/`).
- **Change log levels**: Create `docker-compose.change.logging.yml`.

Check `docs/` for more guides as features are added.

## Need Help?
- Read the [Traefik Documentation](https://doc.traefik.io/traefik/).
- Open an issue on GitHub for bugs or questions.
- Join the Traefik community for support.

Happy routing!