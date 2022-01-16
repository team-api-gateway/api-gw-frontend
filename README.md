# api-gw-frontend

Dies ist das Frontend für das API-Gateway

## Setup für lokale Entwicklung

1. Installieren der dependencies
   ```bash
   $ npm install
   ```
2. Umgebungsvariablen in .env setzen (siehe .env.example)
3. ggf. Datenbank und Backend starten
4. Webseite mit hot reload starten (Seite erreichbar unter localhost:3000)
   ```bash
   $ npm run dev
   ```
## Setup für Production

Das Projekt kann entweder direkt auf dem Produktionsserver mit npm gestartet werden, oder als statisches Projekt exportiert werden:

Mittels npm:
```bash
$ npm run build
$ npm run start
```

Statisches Projekt erstellen:
```bash
$ npm run generate
```

Für Details über nuxtjs: [documentation](https://nuxtjs.org).
