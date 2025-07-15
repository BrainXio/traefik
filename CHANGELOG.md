# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]
### Added
- Added `docker-compose.extend.volumes.yml` to use Docker volumes (`traefik_certs`, `traefik_logs`) for certificates and logs instead of local disk, with file-based logging.
- Wildcard SAN support for DNS-01 challenge certificates in `docker-compose.extend.cloudflare-dns.yml`.
- Renamed configuration files to `traefik-cloudflare-dns-staging.yml` and `traefik-cloudflare-dns-prod.yml` for clarity.
- Updated environment variable defaults to generic `example.com` and `user@example.com` in `.env.example`.

## [0.1.0] - 2025-07-15
### Added
- Initial Traefik setup with HTTPS via Let’s Encrypt (HTTP challenge).
- Docker Compose configuration with `traefik-net` network for compatibility with other services.
- Staging (`traefik-http-staging.yml`) and production (`traefik-http-prod.yml`) configurations, selectable via `TRAEFIK_CONFIG` with defaults.
- Basic auth for staging dashboard using Docker secret (`traefik_dashboard_users.txt`).
- Dashboard disabled in production for security.
- Static (`traefik-http-*.yml`) and dynamic (`config.yml`) configurations.
- Environment variable defaults for robustness without `.env`.
- Beginner-friendly `README.md` and `docs/using-traefik.md` for setup, Nginx routing example, and debugging.
- `.env.example` for non-sensitive variables and `.gitignore` to exclude secrets and artifacts.