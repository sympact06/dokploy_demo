# Dokploy Demo – Laravel + Inertia

Een kant-en-klare voorbeeldapplicatie die laat zien hoe je een moderne Laravel-backend
combineert met een React/Inertia-frontend mét een kant-en-klare **Docker Compose**-configuratie.  
Met behulp van **Nixpacks** kan dit project nog steeds direct worden gebouwd en uitgerold
op **Dokploy**, maar lokaal heb je met één commando een volledig werkende stack (Laravel + MySQL + Mailpit).

---

## Inhoudsopgave

1. Introductie
2. Vereisten
3. Lokaal ontwikkelen
4. Deployen naar Dokploy
5. Omgevingsvariabelen
6. Veelgestelde vragen

---

## 1. Introductie

Dit project is gebaseerd op Laravel 10, Inertia.js, React 18 en Vite.  
Alle standaard Laravel-features (migraties, queue, storage, tests) zijn
beschikbaar.  
Voor de frontend worden TypeScript en shadcn/ui-componenten gebruikt.

Het doel van deze repository is om **plug-and-play** te zijn op Dokploy:
* géén Dockerfile nodig
* géén speciale build-scripts
* alles wordt afgehandeld door Nixpacks

---

## 2. Vereisten

Lokaal:
* PHP ^8.2 met ext-pdo, ext-openssl, ext-mbstring, ext-intl
* Composer
* Node ^20 + **Bun** (package-manager & runtime)
* Docker + Docker Compose (voor de database en Mailpit)

In Dokploy:
* Een Dokploy-account + een server
* Een MySQL of PostgreSQL-database (kan rechtstreeks in Dokploy worden
  aangemaakt)
* Een ".env" (of env-vars) met minimaal `APP_KEY` en DB-gegevens

---

## 3. Lokaal ontwikkelen

```bash
# 1. Repo klonen
$ git clone <dit-repo>
$ cd dokploy_demo

# 2. PHP-afhankelijkheden
$ composer install

# 3. Node-afhankelijkheden
$ bun install

# 4. .env aanmaken
$ cp env.example .env
$ php artisan key:generate
# Pas indien nodig je DB-gegevens aan

# 5. Docker-services starten (app, database & mail)
$ docker compose up -d

# 6. Database migreren & seeden
$ php artisan migrate --seed

# 7. Start ontwikkelen
$ bun run dev    # start Vite dev-server
# Open daarna http://localhost:8000 in je browser.
```

> Tip: laat `bun run dev` continu draaien; Vite ververst de pagina automatisch.

---

## 4. Deployen naar Dokploy (Nixpacks)

1. **Nieuwe applicatie**
   * Kies *Git Repository* en selecteer deze repo.
   * Leave *Build Type* op **Nixpacks** (standaard).
2. **Poortinstelling**  
   Nixpacks detecteert automatisch Nginx/Php-FPM en expose-t poort **80**.
   Je hoeft niets aan te passen.
3. **Omgevingsvariabelen**
   * Voeg de waarden uit `env.example` toe.
   * Essentieel zijn `APP_KEY`, `APP_ENV`, `APP_DEBUG=false` en alle `DB_*`
     variabelen.
   * Stel bovendien `NIXPACKS_PHP_ROOT_DIR=/app/public` in zodat Nginx je
     Laravel public-map serveert.
4. **Database toevoegen**
   * Ga in Dokploy naar **Databases > Nieuw** en maak een MySQL of PostgreSQL
     instantie.
   * Kopieer de connection string naar je applicatie-env-vars (`DB_HOST`,
     `DB_PORT`, `DB_DATABASE`, `DB_USERNAME`, `DB_PASSWORD`).
5. **Deploy**  
   Klik op *Deploy*.  Dokploy
   * checkt de repo uit
   * bouwt via Nixpacks (installeert PHP, Bun, Composer)
   * voert `vite build` uit
   * start Nginx/Php-FPM
6. **Domein koppelen**  
   Voeg een domein toe, selecteer *Exposed Port* **80** en klaar!

> **Queue-workers** nodig?  
> Voeg in Dokploy een extra *Worker*-proces toe met het commando:
> `php artisan queue:work --sleep=3 --tries=3 --max-time=3600`.

---

## 5. Omgevingsvariabelen

| Variabele            | Verplicht | Beschrijving                           |
|----------------------|-----------|----------------------------------------|
| `APP_KEY`            | ja        | Laravel app-sleutel (`php artisan key:generate`)|
| `APP_ENV`            | ja        | `production` in Dokploy               |
| `APP_DEBUG`          | ja        | `false` in Dokploy                    |
| `APP_URL`            | nee       | Publieke URL (https://example.com)     |
| `DB_CONNECTION`      | ja        | `mysql` of `pgsql`                     |
| `DB_HOST`/`PORT`/... | ja        | Door Docker Compose (lokaal) of Dokploy (prod) |
| `NIXPACKS_PHP_ROOT_DIR` | ja   | Altijd `/app/public`                   |

Alle overige standaard Laravel-variabelen (cache, queue, mail) kun je naar
behoefte toevoegen.

---

## 6. Veelgestelde vragen

### 1. Moet ik zelf een Dockerfile schrijven?

Nee.  Nixpacks genereert automatisch een multi-stage build met Nginx en
Php-FPM.

### 2. Hoe run ik queue-workers of cron-jobs?

Voeg in Dokploy een extra **Worker**- of **Scheduler**-proces toe.  Gebruik als
commando respectievelijk `php artisan queue:work` of `php artisan schedule:run`.

### 3. Waar vind ik de logs?

Alle container-logs zijn zichtbaar in het *Logs*-tabblad van je Dokploy
applicatie.

---

> Met ❤️ gebouwd als demo. 