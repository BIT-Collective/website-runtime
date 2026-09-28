# Vorlage: statische Website auf Basis von website-runtime.
# Anpassen: Build-Befehl und Ausgabeordner (dist/), ggf. Dateien, die nicht ausgeliefert werden duerfen.

FROM node:22-alpine AS build
WORKDIR /src
COPY package.json package-lock.json ./
RUN npm ci
COPY . .
RUN npm run build

# Version bewusst festnageln — Runtime-Updates laufen als eigener Ring-Rollout.
FROM ghcr.io/bit-collective/website-runtime:v1.0.0
COPY --from=build /src/dist /srv
