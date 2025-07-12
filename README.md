# Traefik Reverse Proxy

Welcome to the `brainxio/traefik` repository! This project sets up a Traefik reverse proxy using Docker Compose to securely route traffic to your services with automatic HTTPS certificates from Let’s Encrypt. It’s designed to be easy to use, even if you’re new to Traefik or Docker.

## What is Traefik?

Traefik is a modern reverse proxy that makes it simple to route web traffic to your services. It automatically handles HTTPS certificates, so your connections are secure, and it works seamlessly with Docker to discover and route to your containers.

## Getting Started

### Prerequisites
- **Docker** and **Docker Compose** installed on your system.
- A domain name (e.g., `yourdomain.com`) for production certificates, or use `traefik.local` for local testing with the staging endpoint.
- A valid email address for Let’s Encrypt certificate notifications.
- Basic familiarity with the command line.

### Setup Steps

1. **Clone the Repository**
   ```bash
   git clone https://github.com/brainxio/traefik.git
   cd traefik
   ```

2. **Create the `.env` File**
   Copy the example file and customize it:
   ```bash
   cp .env.example .env
   ```
   Edit `.env` to set:
   - `TRAEFIK_DOMAIN`: The domain for the Traefik dashboard (e.g., `traefik.local` for testing or `traefik.yourdomain.com` for production).
   - `DOCKER_SOCK`: The Docker socket path (default: `/var/run/docker.sock`). For rootless Docker, set to `/var/run/user/<UID>/docker.sock` (e.g., `/var/run/user/1000/docker.sock`).
   - `TRAEFIK_CERTIFICATESRESOLVERS_letsencrypt_ACME_EMAIL`: A valid, deliverable email (e.g., `user@yourdomain.com`).
   - `TRAEFIK_CONFIG`: The Traefik configuration file (default: `http-staging` for staging). Set to `http-prod` for production.

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
   **Note**: The `secrets/` directory is ignored by `.gitignore` and not committed.

4. **Clear Existing Certificates (if needed)**
   If switching between staging and production or encountering certificate issues, remove the existing certificate files:
   ```bash
   rm -f letsencrypt/acme-staging.json letsencrypt/acme-prod.json
   ```

5. **Start Traefik**
   Run the following command to start Traefik:
   ```bash
   docker-compose up -d
   ```

6. **Access the Dashboard**
   For staging (`TRAEFIK_CONFIG=http-staging`), open your browser and go to `https://<TRAEFIK_DOMAIN>` (e.g., `https://traefik.local`). Enter the username (`admin`) and password from `secrets/traefik_dashboard_users.txt`. For local testing, add to `/etc/hosts`:
   ```bash
   127.0.0.1 traefik.local
   ```
   **Note**: The staging endpoint issues untrusted certificates, so you may see a browser warning. In production (`TRAEFIK_CONFIG=http-prod`), the dashboard is disabled for security.

### Next Steps
- Check out `docs/using-traefik.md` for tips on routing services (like an Nginx web server) and debugging.
- Add more services by creating `docker-compose.extend.<feature>.yml` files (e.g., for metrics or custom routing).
- For production, update `TRAEFIK_DOMAIN` to a public domain and set `TRAEFIK_CONFIG=http-prod` in `.env`.

## Troubleshooting
- **Dashboard not loading in staging?**
  - Ensure `TRAEFIK_DOMAIN` is set correctly in `.env` and resolves to your server (e.g., add `127.0.0.1 traefik.local` to `/etc/hosts`).
  - Verify basic auth credentials in `secrets/traefik_dashboard_users.txt`. Regenerate if needed:
    ```bash
    echo "admin:$(openssl passwd -apr1 yourpassword)" > secrets/traefik_dashboard_users.txt
    ```
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
- **Rootless Docker issues?** Confirm `DOCKER_SOCK` is set to the correct path (e.g., `/var/run/user/1000/docker.sock`).
- **Need help?** Open an issue on GitHub or check the [Traefik Documentation](https://doc.traefik.io/traefik/).

## License
This project is licensed under the MIT License. See `LICENSE` for details.