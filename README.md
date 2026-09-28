# website-runtime

Gemeinsamer Baukasten für alle statischen Websites, die BIT-Collective über Coolify hostet: die eigenen Seiten und die Kunden-Websites.

Prinzip **„build once, promote“**:
1. GitHub Actions baut **ein** Image und prüft es.
2. Dasselbe Image geht zuerst nach Ring 0 (Staging), dann nach Ring 1 und Ring 2.
3. Ein Rollback bedeutet: den alten Image-Tag setzen und neu deployen.

## Inhalt

| Pfad | Zweck |
|---|---|
| `runtime/` | Basis-Image `ghcr.io/bit-collective/website-runtime`: Caddy, SPA-Fallback, Security-Header, `/healthz` mit Versionsangabe, `SITE_MODE=staging` (Basic Auth + noindex, startet ohne Passwort-Hash nicht) bzw. `production` |
| `.github/workflows/build-website.yml` | wiederverwendbarer Workflow: `push main` → `sha-<kurz>` (mit Prüfungen), Release → derselbe Build bekommt zusätzlich `vX.Y.Z` (Retag ohne Neubau) |
| `.github/workflows/runtime.yml` | baut das Basis-Image selbst |
| `templates/new-website-repo/` | Vorlage für jedes neue Website-Repo (`Dockerfile`, Workflow-Aufruf) |
| `templates/website.compose.yml` | Vorlage für die Coolify-Ressource („Docker Compose“) |
| `bin/bit-deploy` | vom Mac über Tailscale: `IMAGE_TAG` setzen, deployen, auf die neue Version warten; einzeln oder ringweise |

## Neue Website anlegen

1. Neues privates Repo `BIT-Collective/web-k000n` anlegen (Kundennummer, **kein Klarname**).
2. Den Inhalt von `templates/new-website-repo/` hineinkopieren und in `Dockerfile` Build-Befehl und Ausgabeordner anpassen.
3. Nach dem Push auf `main` baut Actions das Image `ghcr.io/bit-collective/web-k000n:sha-…`.
4. In Coolify: Projekt `K000n <Kurzname>` → Ressource „Docker Compose (empty)“ mit dem Inhalt von `templates/website.compose.yml`.
   - Env setzen: `IMAGE`, `IMAGE_TAG`, `SITE_MODE`, und nur für Staging `BASIC_AUTH_USER`/`BASIC_AUTH_HASH` (Hash mit „Keep $ as is“).
   - Domain am Service `web` eintragen und den Ring-Tag setzen.
5. Eine Zeile in `~/.config/bit-deploy/resources` ergänzen, dann `bit-deploy <name> sha-…` ausführen.

Den Passwort-Hash für Staging erzeugst du so: `docker run --rm -it caddy:2-alpine caddy hash-password`

## Release und Ringe

```
git push                       # → sha-1a2b3c4 gebaut
bit-deploy <staging> sha-1a2b3c4
gh release create v1.3.0       # → v1.3.0 = derselbe Digest wie sha-1a2b3c4
bit-deploy <kunde> v1.3.0      # bzw. --ring ring1 --product <p> v1.3.0
```

Ein Update von `website-runtime` selbst wird auch per Release ausgeliefert: Die Website-Repos heben die `FROM`-Zeile an, bauen neu und durchlaufen dann ebenfalls Ring 0 → 1 → 2.

## Grenzen

- Nur statische Seiten. Es wird kein Server-Code ausgeliefert: Liegen PHP-Dateien im Image, bricht der Build ab.
- Kontaktformulare brauchen einen eigenen Mail-Endpunkt. Den gibt es noch nicht.
