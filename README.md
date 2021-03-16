## Simple HTTP Traefik Example

1. Run `docker-compose up -d`
2. Both example services will be available at: [whoami-1.127.0.0.1.xip.io:8080](http://whoami-1.127.0.0.1.xip.io:8080) and [whoami-2.127.0.0.1.xip.io:8080](http://whoami-2.127.0.0.1.xip.io:8080)

---

## SSL + LetsEncrypt

You can enable SSL and LetsEncrypt using some additional configuration.

### docker-compose.yml
```
version: '3.7'

services:
  trafiek:
    container_name: traefik
    image: traefik:latest
    restart: always
    labels:
      traefik.http.routers.http_catchall.rule: HostRegexp(`{any:.+}`)
      traefik.http.routers.http_catchall.entrypoints: web
      traefik.http.routers.http_catchall.middlewares: https_redirect
      traefik.http.middlewares.https_redirect.redirectscheme.scheme: https
      traefik.http.middlewares.https_redirect.redirectscheme.permanent: true
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /root/acme.json:/acme.json
      - ./traefik.yml:/etc/traefik/traefik.yml
    networks:
      - reverse-proxy
    ports:
      - "80:80"
      - "443:443"

networks:
  reverse-proxy:
    name: reverse-proxy
```

### traefik.yml
```
providers:
  docker: {}

entryPoints:
  web:
    address: :80
  websecure:
    address: :443

certificatesResolvers:
  le:
    acme:
      email: YOUR_EMAIL
      storage: acme.json
      httpChallenge:
        entryPoint: web
```

## Your service docker-composer.yml
```
    labels
      traefik.enable: true
      [...]
      traefik.http.routers.SERVICE.tls: true
      traefik.http.routers.SERVICE.tls.certresolver: le
```

---

## SSL + LetsEncrypt + Cloudflare

When using multidomain certificates, CloudFlare is needed.

### docker-compose.yml
```
version: '3.7'

services:
  trafiek:
    container_name: traefik
    image: traefik:latest
    restart: always
    labels:
      traefik.http.routers.http_catchall.rule: HostRegexp(`{any:.+}`)
      traefik.http.routers.http_catchall.entrypoints: web
      traefik.http.routers.http_catchall.middlewares: https_redirect
      traefik.http.middlewares.https_redirect.redirectscheme.scheme: https
      traefik.http.middlewares.https_redirect.redirectscheme.permanent: true
    environment:
      - CF_API_EMAIL=YOUR_CLOUDFLARE_EMAIL
      - CF_API_KEY=YOUR_CLOUDFLARE_API_KEY
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /root/acme.json:/acme.json
      - /root/acme_cf.json:/acme_cf.json
      - ./traefik.yml:/etc/traefik/traefik.yml
    networks:
      - reverse-proxy
    ports:
      - "80:80"
      - "443:443"

networks:
  reverse-proxy:
    name: reverse-proxy
```

### traefik.yml
```
providers:
  docker: {}

entryPoints:
  web:
    address: :80
  websecure:
    address: :443

certificatesResolvers:
  le:
    acme:
      email: YOUR_EMAIL
      storage: acme.json
      httpChallenge:
        entryPoint: web
  cloudflare:
    acme:
      email: YOUR_EMAIL
      storage: acme_cf.json
      dnsChallenge:
        provider: cloudflare
        resolvers:
          - "1.1.1.1:53"
          - "1.0.0.1:53"
```

## Your service docker-composer.yml

```
    labels
      traefik.enable: true
      [...]
      traefik.http.routers.SERVICE.tls: true
      traefik.http.routers.SERVICE.tls.certresolver: cloudflare
```

---

## SSL with custom certificate

When using your own certificate, you can merge all settings but still apply custom certs to routers

### docker-compose.yml
```
version: '3.7'

services:
  trafiek:
    container_name: traefik
    image: traefik:latest
    restart: always
    labels:
      traefik.http.routers.http_catchall.rule: HostRegexp(`{any:.+}`)
      traefik.http.routers.http_catchall.entrypoints: web
      traefik.http.routers.http_catchall.middlewares: https_redirect
      traefik.http.middlewares.https_redirect.redirectscheme.scheme: https
      traefik.http.middlewares.https_redirect.redirectscheme.permanent: true
    environment:
      - CF_API_EMAIL=YOUR_CLOUDFLARE_EMAIL
      - CF_API_KEY=YOUR_CLOUDFLARE_API_KEY
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /root/acme.json:/acme.json
      - /root/acme_cf.json:/acme_cf.json
      - ./traefik.yml:/etc/traefik/traefik.yml
      - ./dynamic.yml:/etc/traefik/dynamic.yml
      - ./SSL:/SSL
    networks:
      - reverse-proxy
    ports:
      - "80:80"
      - "443:443"

networks:
  reverse-proxy:
    name: reverse-proxy
```

### traefik.yml
```
providers:
  docker: {}
  file:
    filename: /etc/traefik/dynamic.yml

entryPoints:
  web:
    address: :80
  websecure:
    address: :443

certificatesResolvers:
  le:
    acme:
      email: YOUR_EMAIL
      storage: acme.json
      httpChallenge:
        entryPoint: web
  cloudflare:
    acme:
      email: YOUR_EMAIL
      storage: acme_cf.json
      dnsChallenge:
        provider: cloudflare
        resolvers:
          - "1.1.1.1:53"
          - "1.0.0.1:53"
```

### dynamic.yml
```
tls:
  certificates:
    - certFile: /SSL/DOMAIN.cert
      keyFile: /SSL/DOMAIN.key
```

## Your service docker-composer.yml

```
    labels
      traefik.enable: true
      [...]
      traefik.http.routers.SERVICE.tls: true
```
