# Using Traefik Metrics

This guide explains how to enable and use Prometheus metrics with the Traefik reverse proxy in the `brainxio/traefik` repository. It’s designed for beginners, so don’t worry if you’re new to Traefik or Prometheus!

## What You’ll Learn
- Enable Traefik’s Prometheus metrics endpoint.
- Access the metrics endpoint.
- Configure an external Prometheus instance to scrape Traefik metrics.
- Debug metrics issues.

## Enabling Metrics

1. **Ensure Traefik is Running**
   Follow the setup steps in `README.md` or `docs/using-traefik.md` to start Traefik with the desired configuration (e.g., DNS or HTTP challenge, with volumes and healthchecks):
   ```bash
   docker compose -f docker-compose.extend.cloudflare-dns.yml -f docker-compose.extend.volumes.yml -f docker-compose.extend.healthcheck.yml up -d
   ```

2. **Enable Metrics**
   Apply the metrics override file:
   ```bash
   docker compose -f docker-compose.extend.cloudflare-dns.yml -f docker-compose.extend.volumes.yml -f docker-compose.extend.healthcheck.yml -f docker-compose.extend.metrics.yml up -d
   ```
   This enables the `/metrics` endpoint on `http://<TRAEFIK_DOMAIN>:8080/metrics`, protected by basic auth.

3. **Verify Metrics Endpoint**
   Access the metrics endpoint using the `admin` username and password from `secrets/traefik_dashboard_users.txt`:
   ```bash
   curl -u admin:yourpassword http://<TRAEFIK_DOMAIN>:8080/metrics
   ```
   For local testing, ensure `<TRAEFIK_DOMAIN>` (e.g., `example.com`) is in `/etc/hosts`:
   ```bash
   echo "127.0.0.1 example.com" >> /etc/hosts
   ```
   Expect output like:
   ```
   # HELP traefik_config_last_reload_success Last config reload success
   # TYPE traefik_config_last_reload_success gauge
   traefik_config_last_reload_success 1
   ```

## Configuring Prometheus
To collect Traefik metrics, configure an external Prometheus instance to scrape the `/metrics` endpoint.

1. **Edit Prometheus Configuration**
   Add a scrape job to `prometheus.yml`:
   ```yaml
   scrape_configs:
     - job_name: 'traefik'
       static_configs:
         - targets: ['<TRAEFIK_DOMAIN>:8080']
       basic_auth:
         username: admin
         password: yourpassword
   ```
   Replace `<TRAEFIK_DOMAIN>` with your domain (e.g., `example.com`) and `yourpassword` with the password from `secrets/traefik_dashboard_users.txt`.

2. **Restart Prometheus**
   Reload or restart your Prometheus instance to apply the configuration. Refer to your Prometheus setup for specific commands.

3. **Verify Metrics in Prometheus**
   Access the Prometheus UI (typically `http://<prometheus-host>:9090`) and check the `traefik_config_last_reload_success` metric or others like `traefik_service_requests_total`.

## Debugging Tips
- **Metrics endpoint not accessible?**
  - Ensure `--metrics.prometheus=true` is set in the Compose file.
  - Test the endpoint:
    ```bash
    curl -u admin:yourpassword http://<TRAEFIK_DOMAIN>:8080/metrics
    ```
  - Check Traefik logs:
    ```bash
    docker run --rm -v traefik_logs:/logs busybox cat /logs/traefik.log
    ```
  - Verify `TRAEFIK_DOMAIN` resolves (e.g., `/etc/hosts` or DNS).
- **Prometheus not scraping?**
  - Check Prometheus logs for errors.
  - Ensure basic auth credentials match `secrets/traefik_dashboard_users.txt`.
  - Verify network access to `<TRAEFIK_DOMAIN>:8080`.
- **Need help?** Open a GitHub issue or check [Traefik Documentation](https://doc.traefik.io/traefik/observability/metrics/prometheus/).

## Available Metrics
Traefik exposes metrics like:
- `traefik_service_requests_total`: Total HTTP requests per service.
- `traefik_service_request_duration_seconds`: Request latency.
- `traefik_config_last_reload_success`: Config reload status.
See [Traefik Prometheus Metrics](https://doc.traefik.io/traefik/observability/metrics/prometheus/) for a full list.

## Next Steps
- Explore `docs/using-traefik.md` for general Traefik setup and routing.
- Consider adding a local Prometheus instance for testing (future feature).