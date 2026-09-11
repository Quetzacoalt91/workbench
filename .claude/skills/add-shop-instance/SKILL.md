---
name: add-shop-instance
description: Bring up a new PrestaShop-<N> instance in this workbench (clone, parameterize docker-compose ports/network, write .env, docker-up, verify). Use when the user asks to add, create, or spin up a new shop instance.
---

# Adding a new shop instance

Pick the next free `<N>` (one more than the highest existing `PrestaShop-*`), then:

1. Clone: `git clone git@github.com:PrestaShop/PrestaShop.git PrestaShop-<N>`.
2. In `PrestaShop-<N>/docker-compose.yml`, parameterize the hardcoded ports and network name:
   - `mysql` port mapping → `"${DB_PORT:-3306}:3306"`
   - `prestashop-git` port mappings → `"${HTTP_PORT:-8001}:80"` and `"${HTTPS_PORT:-8002}:443"`
   - `maildev` port mappings → `"${MAILDEV_UI_PORT:-1080}:1080"` and `"${MAILDEV_SMTP_PORT:-1025}:1025"`
   - under `networks:`, keep the key `prestashop-network` but set `name: ${NETWORK_NAME:-prestashop-network}` (only the `name:` value is interpolated, not the top-level key)
3. Create `PrestaShop-<N>/.env` with values unique to this instance — check every other instance's `.env` first and pick ports/names that don't collide with any of them or with unrelated stacks on the host:
   - `COMPOSE_PROJECT_NAME`
   - `NETWORK_NAME`
   - `HTTP_PORT`, `HTTPS_PORT`, `DB_PORT`, `MAILDEV_UI_PORT`, `MAILDEV_SMTP_PORT`
   - `PS_DOMAIN=localhost:<HTTP_PORT>`
4. From inside `PrestaShop-<N>/`, run `make docker-up`.
5. Wait for the automatic install (composer install, asset build, DB install — can take a while, more so alongside other running instances), then verify: `curl http://localhost:<HTTP_PORT>/` should return 200.
