# Ubuntu Home Server — Compose Conventions

This repo is a Docker Compose based home server. The root [docker-compose.yaml](docker-compose.yaml)
only declares shared resources (networks, the project name) and pulls in every
service via `include:`. Each service lives in its own file, grouped by folder:

- `compose/applications/` — user-facing apps
- `compose/infrastructure/` — platform services
- `compose/iot/` — home automation / IoT services

If asked to create a new compose file and the target folder isn't specified or
obvious from context, ask which of the three it belongs, or if a new folder needs to be created before creating it.

## Service file template

Use [compose/applications/homebox.yaml](compose/applications/homebox.yaml) as the
reference example. Structure every new service file as:

```yaml
# Short description of what the service does.
# Link to docs, e.g. https://example.com/docs

services:
  service-name:
    container_name: service-name
    image: registry/image:tag
    restart: unless-stopped
    depends_on:
      - service-name
    security_opt:
      - no-new-privileges:true
    env_file: $DOCKER_SECRETS_DIR/service-name_secrets
    environment: # non-secret config only
      - TZ=$TZ
      - SOME_OPTION=value
    volumes:
      - $DOCKER_DATA_DIR/service-name:/data:rw
    networks:
      - traefik-proxy
    healthcheck:
      test: ['...']
    labels:
      - traefik.enable=true
      - traefik.http.routers.service-name-router.rule=Host(`service-name.homeserver.lukan.rocks`)
      - traefik.http.routers.service-name-router.entrypoints=web,websecure
      - traefik.http.routers.service-name-router.tls=true
      - traefik.http.routers.service-name-router.tls.certresolver=letsencrypt
      - traefik.http.routers.service-name-router.service=service-name-services
      - traefik.http.services.nf-price-tracker-services.loadbalancer.server.port=....
    # anything else (command, extra labels, etc.) goes after labels
```

Key order within a service is fixed: `container_name`, `image`, `restart`, `depends_on`, `privileged`, `network_mode`, `extra_hosts`, `security_opt`, `env_file`, `environment`, `volumes`, `networks`, `healthcheck`, `labels`, then anything else the service needs. `privileged` and `network_mode` are only present when the container genuinely requires them and replace `networks` when used — a service doesn't declare both. `extra_hosts` is only present when needed; it sits right after `network_mode`/`privileged` since it's networking-related config, whether or not the service actually uses `network_mode`.

Defaults — deviate only when the container genuinely requires it or otherwise specified:
- `restart: unless-stopped`
- `security_opt: [no-new-privileges:true]`
- `networks: [traefik-proxy]`
- `labels` at minimum enable Traefik routing for the service's web interface

`environment` entries use list form (`- KEY=value`), not map form (`KEY: value`).

## Secrets

Any env var holding a secret or password does **not** go in `environment:`.
Instead:

1. Create a file in `./secrets/` (referenced as `$DOCKER_SECRETS_DIR` in compose
   files), formatted as one or more `KEY=VALUE` lines — e.g.
   `secrets/service_secrets` contains `API_KEY=...`.
2. Reference it from the service with `env_file:`, not the Docker Compose
   top-level `secrets:` block:
   ```yaml
   env_file:
     - $DOCKER_SECRETS_DIR/service-name_secrets
   ```

Everything else (feature flags, log levels, hostnames, non-sensitive config)
stays inline under `environment:`.