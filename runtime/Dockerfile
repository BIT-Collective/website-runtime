# website-runtime: gemeinsames Basis-Image fuer alle statischen BIT-Websites.
# Eine Website braucht nur: FROM ghcr.io/bit-collective/website-runtime:<version>
#                            COPY --from=build /src/dist /srv
# Eigene Routen (optional):  COPY <datei>.caddy /etc/caddy/site.d/
# SITE_MODE=staging (Default) -> Basic Auth + noindex, startet ohne BASIC_AUTH_HASH gar nicht.
# SITE_MODE=production        -> oeffentlich.
FROM caddy:2-alpine
COPY Caddyfile /etc/caddy/Caddyfile
COPY mode-staging.caddy mode-production.caddy /etc/caddy/
RUN mkdir -p /etc/caddy/site.d
ENV SITE_MODE=staging
EXPOSE 80
HEALTHCHECK --interval=15s --timeout=3s --retries=3 \
    CMD wget -q --spider http://127.0.0.1/healthz || exit 1
