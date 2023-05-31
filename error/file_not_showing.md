# User Story

When re-setup our erpnext project while sending the Mail. we in that mail the image is not rendering after doing,
some research we i dentified the error cam from the `treafik`. then we changed some changes in the `treafik`.
here the changes in `overrides/compose.traefik-ssl.yaml`


`compose.traefik-ssl.yaml`
```
version: "3.3"

services:
  traefik:
    labels:
      # https-redirect middleware to redirect HTTP to HTTPS
      - traefik.http.middlewares.https-redirect.redirectscheme.scheme=https
      - traefik.http.middlewares.https-redirect.redirectscheme.permanent=true
      # traefik-http to use the middleware to redirect to https
      - traefik.http.routers.traefik-public-http.middlewares=https-redirect
      # traefik-https the actual router using HTTPS
      # Uses the environment variable TRAEFIK_DOMAIN
      - traefik.http.routers.traefik-public-https.rule=Host(`${TRAEFIK_DOMAIN}`)
      - traefik.http.routers.traefik-public-https.entrypoints=https
      - traefik.http.routers.traefik-public-https.tls=true
      # Use the special Traefik service api@internal with the web UI/Dashboard
      - traefik.http.routers.traefik-public-https.service=api@internal
      # Use the "le" (Let's Encrypt) resolver created below
      - traefik.http.routers.traefik-public-https.tls.certresolver=le
      # Enable HTTP Basic auth, using the middleware created in the traefik.yaml file
      - traefik.http.routers.traefik-public-https.middlewares=admin-auth
    command:
      # Enable Docker in Traefik, so that it reads labels from Docker services
      - --providers.docker=true
      # Do not expose all Docker services, only the ones explicitly exposed
      - --providers.docker.exposedbydefault=false
      # Create an entrypoint http listening on port 80
      - --entrypoints.http.address=:80
      # Create an entrypoint https listening on port 443
      - --entrypoints.https.address=:443
      # Create the certificate resolver le for Let's Encrypt, uses the environment variable EMAIL
      - --certificatesresolvers.le.acme.email=${EMAIL:?No EMAIL set}
      # Store the Let's Encrypt certificates in the mounted volume
      - --certificatesresolvers.le.acme.storage=/certificates/acme.json
      # Use the TLS Challenge for Let's Encrypt
      - --certificatesresolvers.le.acme.tlschallenge=true
      # Enable the access log, with HTTP requests
      - --accesslog
      # Enable the Traefik log, for configurations and errors
      - --log
      # Enable the Dashboard and API
      - --api
    ports:
      - 443:443
    volumes:
      - cert-data:/certificates

volumes:
  cert-data:
```