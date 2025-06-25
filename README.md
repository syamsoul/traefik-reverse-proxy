#### Creating Required Networks

Before running the services, you need to create the necessary Docker networks. For the reverse proxy setup, create an overlay network using the following command:

```bash
docker network create --driver overlay reverse-proxy-net
```

#### Deploying the Reverse Proxy Service

To start or update the reverse proxy service, run this command:

```bash
docker stack deploy -c ./docker-stack.yml reverse-proxy
```

#### Shutting Down the Reverse Proxy Service

To shutdown the reverse proxy service, run this command:

```bash
docker stack rm reverse-proxy
```

#### Example of Docker Compose Services

```yaml
services:
  redis:
    image: redis:alpine
    volumes:
      - redis-data:/data
    healthcheck:
      test:
        - CMD
        - redis-cli
        - ping
      retries: 3
      timeout: 5s
    networks:
      - main-net

  my-service:
    image: ${IMAGE_NAME}:${LATEST_IMAGE_TAG}
    environment:
      - PHP_OPCACHE_ENABLE=1
    networks:
      - main-net
      - reverse-proxy-net
    volumes:
      - 'shared-lock:/var/www/html/.shared-lock'
      - './storage/logs:/var/www/html/storage/logs'
      - './storage/clockwork:/var/www/html/storage/clockwork'
    deploy:
      replicas: 1
      labels:
        - traefik.enable=true
        - traefik.http.routers.main.rule=Host(`${APP_DOMAIN}`)
        - traefik.http.routers.main.entrypoints=web
        - traefik.http.services.main-service.loadbalancer.server.port=8080
        - traefik.docker.network=reverse-proxy-net
        # - traefik.http.routers.main-service.middlewares=${TRAEFIK_MIDDLEWARES:-}
        # - traefik.http.middlewares.cloudflarewarp.plugin.cloudflarewarp.disableDefault=${TRAEFIK_CLOUDFLAREWARP_DISABLE:-true}

networks:
  main-net:
  reverse-proxy-net:
    name: "reverse-proxy-net"
    external: true
