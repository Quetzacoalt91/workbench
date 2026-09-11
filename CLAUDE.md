# Workbench — Parallel PrestaShop Instances

This folder contains multiple independent PrestaShop instances, one per subfolder (e.g. `PrestaShop-1/`, `PrestaShop-2/`, ...). The number of instances is not fixed — more may be added over time. Treat any subfolder containing a `docker-compose.yml` and its own `.env` as a shop instance.

Each instance is a full docker stack (PrestaShop app + MySQL + maildev) made independent from the others via its own `.env`: unique `COMPOSE_PROJECT_NAME`, unique host ports (HTTP/HTTPS/DB/maildev), and its own Docker network. Instances can be started, stopped, and used concurrently without interfering with each other — see the instance's own `Makefile` (`make docker-up`, `make docker-down`, etc.) and its `.env` for its assigned ports.

## Always locking a shop before use

Multiple Claude Code conversations may work in this workbench at the same time. Before using a shop instance (starting it, running tests against it, changing its data, etc.), a conversation must claim it with a lock file to avoid two conversations colliding on the same instance.

- Lock file location: this folder (the workbench root), not inside the instance subfolder.
- Lock file name: `<name>.lock`, where `<name>` matches the instance's subfolder name (e.g. `PrestaShop-1.lock`).
- Before using an instance, check whether its lock file already exists. If it does, that shop is in use — pick a different, unlocked instance instead.
- If it doesn't exist, create it before starting work on that shop, and keep it present for as long as the shop is in use.
- Remove the lock file when done with the shop (before ending the conversation), so other conversations can use it again.
- A lock file only needs to exist; its content is not load-bearing, but it's helpful to note what's using it (e.g. a short description and timestamp) in case a stale lock needs to be investigated.
- If all instances are taken, stop and warn the user.

## How to add a new instance

To bring up a new shop instance (named `PrestaShop-<N>` below, adjust to the next free number):

1. Run `git clone git@github.com:PrestaShop/PrestaShop.git PrestaShop-<N>`.
2. In `PrestaShop-<N>/docker-compose.yml`, parameterize the hardcoded ports and network name so they can be overridden per instance:
   - `mysql` port mapping → `"${DB_PORT:-3306}:3306"`
   - `prestashop-git` port mappings → `"${HTTP_PORT:-8001}:80"` and `"${HTTPS_PORT:-8002}:443"`
   - `maildev` port mappings → `"${MAILDEV_UI_PORT:-1080}:1080"` and `"${MAILDEV_SMTP_PORT:-1025}:1025"`
   - the `prestashop-network` entry under `networks:` → keep the key as `prestashop-network` but set `name: ${NETWORK_NAME:-prestashop-network}` (only the `name:` value is interpolated; the top-level network key itself is not)
3. Create `PrestaShop-<N>/.env` with values unique to this instance: `COMPOSE_PROJECT_NAME`, `NETWORK_NAME`, `HTTP_PORT`, `HTTPS_PORT`, `DB_PORT`, `MAILDEV_UI_PORT`, `MAILDEV_SMTP_PORT`, and `PS_DOMAIN=localhost:<HTTP_PORT>`. Pick ports that don't collide with any other instance (existing or unrelated docker stacks on the host).
4. Run `make docker-up` from inside `PrestaShop-<N>/`.
5. Wait for the automatic install to finish (composer install, asset build, DB install — this can take a while, especially if run alongside other instances), then verify the shop is alive: `curl http://localhost:<HTTP_PORT>/` should return 200.
