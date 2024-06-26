####Example of Docker Compose Services


```yaml
services:
  my-service:
    build:
      context: .
      dockerfile: ./docker/prod/Dockerfile
    image: my-company/myproj-my-service:${APP_ENV:-local}
    environment:
      - PHP_OPCACHE_ENABLE=1
    restart: always
    expose:
      - 8080
    labels:
      - traefik.enable=true
      - traefik.http.routers.${APP_ENV:-local}-myproj-myservice.rule=Host(`${APP_DOMAIN}`)
      - traefik.http.services.${APP_ENV:-local}-myproj-myservice-service.loadbalancer.server.port=8080
      - traefik.docker.network=${APP_ENV:-local}-reverse-proxy-net
      # - traefik.http.routers.${APP_ENV:-local}-myproj-myservice.middlewares=${TRAEFIK_MIDDLEWARES:-}
      # - traefik.http.middlewares.cloudflarewarp.plugin.cloudflarewarp.disableDefault=${TRAEFIK_CLOUDFLAREWARP_DISABLE:-true}
    # ports:
    #   - '${APP_PORT:-80}:8080'
    volumes:
      - './.env:/var/www/html/.env'
      - './storage/logs:/var/www/html/storage/logs'
      - './storage/clockwork:/var/www/html/storage/clockwork'
    networks:
      - reverse-proxy-net
      - main-net
    depends_on:
      # - mysql
      - redis
  redis:
    image: 'redis:alpine'
    restart: always
    expose:
      - 6379
    volumes:
      - '/var/docker-data/redis/${APP_ENV:-local}-myproj/data:/data'
    networks:
      - main-net
    healthcheck:
      test:
        - CMD
        - redis-cli
        - ping
      retries: 3
      timeout: 5s
networks:
  reverse-proxy-net:
    name: "${APP_ENV:-local}-reverse-proxy-net"
    external: true
  main-net:
    name: "${APP_ENV:-local}-myproj-main-net"
    driver: bridge
```
