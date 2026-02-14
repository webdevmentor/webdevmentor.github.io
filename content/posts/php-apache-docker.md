---
title: "PHP & Apache mit Docker: Die Umgebung der Shared Hoster"
date: 2026-02-14
draft: false
tags: ["PHP", "Apache", "Docker", "Shared Hosting", "Entwicklungsumgebung"]
---

Im [letzten Post]({{< ref "local-environments.md" >}}) haben wir uns angeschaut, wie man mit Docker ganz einfach PHP-Skripte über das CLI ausführen kann. Das ist super für kleine Tools oder zum Lernen der Sprache. Aber wenn wir echte Webanwendungen entwickeln, brauchen wir meistens einen Webserver.

Heute schauen wir uns an, wie wir **PHP zusammen mit Apache ([apache.org](https://httpd.apache.org/)) in Docker** nutzen. Warum Apache? Weil es nach wie vor der Standard in der Welt des Shared Hostings ist.

### Warum Apache und nicht Nginx?

Wenn du dein Projekt bei einem klassischen Shared Hoster (wie Strato, IONOS oder All-In-One) hostest, begegnest du fast immer einem Apache-Webserver (oder dem Apache-kompatiblen LiteSpeed). Nginx ist zwar extrem performant und beliebt bei VPS- oder Cloud-Setups, aber im Massenmarkt des Webhostings führt an Apache kein Weg vorbei.

Ein Blick auf die aktuellen Statistiken (Quelle: [W3Techs](https://w3techs.com/technologies/cross/programming_language/web_server), Stand Februar 2026) unterstreicht das:

*   **95,5&nbsp;%** aller PHP-Websites nutzen **Apache**.
*   **99,1&nbsp;%** nutzen **LiteSpeed** (welches `.htaccess` und Apache-Konfigurationen versteht).
*   **88,2&nbsp;%** nutzen Nginx (oft jedoch nur als Reverse Proxy *vor* einem Apache).

Für dich als Entwickler bedeutet das: Wenn du lokal mit Apache entwickelst, ist deine Umgebung viel näher an dem, was dich später beim Deployment erwartet. Besonders die mächtigen `.htaccess`-Dateien funktionieren hier "out-of-the-box".

### PHP & Apache mit einem Docker-Befehl

Docker macht uns den Einstieg extrem leicht. Es gibt offizielle Images, die PHP und Apache bereits fertig kombiniert haben.

```bash
docker run --rm -p 8080:80 -v $(pwd):/var/www/html php:8.5-apache
```

**Was passiert hier?**
*   `php:8.5-apache`: Wir nutzen das offizielle Image, das Apache schon an Bord hat.
*   `-p 8080:80`: Wir mappen den Port 80 des Containers (Standard für HTTP) auf Port 8080 deines Rechners.
*   `-v $(pwd):/var/www/html`: Wir mounten dein aktuelles Verzeichnis in das Standard-Web-Verzeichnis von Apache.

Wenn du jetzt deinen Browser öffnest und `http://localhost:8080` aufrufst, wird deine `index.php` direkt vom Apache-Server verarbeitet.

### Professioneller mit Docker Compose

Sobald dein Projekt wächst, wird der `docker run` Befehl unhandlich. Hier kommt **Docker Compose** ins Spiel. Erstelle eine Datei namens `docker-compose.yml` in deinem Projekt:

```yaml
services:
  web:
    image: php:8.5-apache
    ports:
      - "8080:80"
    volumes:
      - .:/var/www/html
```

Jetzt reicht ein einfaches:

```bash
docker compose up -d
```

&hellip;&nbsp;und deine Umgebung läuft im Hintergrund. Mit `docker compose stop` fährst du sie wieder herunter.

### Die richtige PHP-Version wählen

Ein großer Vorteil von Docker ist, dass du die PHP-Version deines Containers exakt auf deine Zielumgebung beim Hoster abstimmen kannst. In unseren Beispielen nutzen wir `php:8.5-apache`, was der neuesten stabilen Version entspricht. Aktuell verbreitete PHP 8.x Image Tags sind:

*   **php:8.5-apache**: Die neueste stabile Version.
*   **php:8.4-apache**: Die aktuelle stabile Version mit vielen neuen Features.
*   **php:8.3-apache**: Weit verbreitet und stabil.
*   **php:8.2-apache**: Wird von den meisten Hostern standardmäßig angeboten.
*   **php:8.1-apache**: Die Basis der 8er-Reihe, die noch bei vielen älteren Projekten im Einsatz ist.

### Fazit

Apache ist vielleicht nicht der "hipste" Webserver da draußen, aber er ist das Rückgrat des PHP-Webs. Mit Docker kannst du diese klassische Umgebung innerhalb von Sekunden nachbauen, ohne dein lokales System mit Apache-Installationen zu belasten.

In den nächsten Schritten werden wir uns anschauen, wie wir Datenbanken (wie MySQL) hinzufügen, um eine komplette lokale Entwicklungsumgebung zu schaffen. Bleib dran!
