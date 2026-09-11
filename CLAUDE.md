# Workbench — Parallel PrestaShop Instances

This folder contains multiple independent PrestaShop instances, one per subfolder (e.g. `PrestaShop-1/`, `PrestaShop-2/`, ...). The number of instances is not fixed — more may be added over time. Treat any subfolder containing a `docker-compose.yml` and its own `.env` as a shop instance.

Each instance is a full docker stack (PrestaShop app + MySQL + maildev) made independent from the others via its own `.env`: unique `COMPOSE_PROJECT_NAME`, unique host ports (HTTP/HTTPS/DB/maildev), and its own Docker network. Instances can be started, stopped, and used concurrently without interfering with each other — see the instance's own `Makefile` (`make docker-up`, `make docker-down`, etc.) and its `.env` for its assigned ports.

## Always locking a shop before use

Multiple Claude Code conversations may work in this workbench at the same time. Before using a shop instance (starting it, running tests against it, changing its data, etc.), a conversation must claim it with a lock file to avoid two conversations colliding on the same instance.

Use skill /shop-lock when a conversation starts.
