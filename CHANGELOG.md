# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [0.1.0] - 2025-07-12
### Added
- Initial Traefik setup with HTTPS via Let’s Encrypt (HTTP challenge).
- Docker Compose configuration with `traefik-net` network for compatibility with other services.
- Staging (`traefik-http-staging.yml`) and production (`traefik-http-prod.yml`) configurations, selectable via `TRAEFIK_CONFIG`.
- Basic auth for staging dashboard using Docker secret (`traefik_dashboard_users.txt`).
- Dashboard disabled in production for security.
- Static (`traefik-http-*.yml`) and dynamic (`config.yml`) configurations.
- Beginner-friendly `README.md` and `docs/using-traefik.md` for setup, Nginx routing example, and debugging.
- `.env.example` for non-sensitive variables and `.gitignore` to exclude secrets and artifacts.