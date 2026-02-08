---
title: "Lokale Entwicklungsumgebungen für PHP: Einfach starten"
date: 2026-02-08
draft: false
tags: ["PHP", "Docker", "XAMPP", "lokale Entwicklungsumgebung"]
---

Wenn man als Einsteiger oder Junior-Entwickler PHP lernen möchte, ist eine **lokale Entwicklungsumgebung** die beste Möglichkeit, ohne Live-Server zu experimentieren. Es gibt verschiedene Wege, PHP lokal auszuführen – wir schauen uns drei Varianten an: reines PHP, XAMPP und Docker.

### 1. Einfaches PHP lokal installieren

Die wohl direkteste Variante ist, **PHP einfach auf dem eigenen Rechner zu installieren**. So kann man Skripte sofort in der Konsole oder im Browser testen, ohne zusätzliche Software zu starten. Der Vorteil liegt darin, dass man ein gutes Gefühl für die PHP-Installation selbst bekommt und genau versteht, wie der Interpreter funktioniert. Nachteilig kann es werden, wenn man unterschiedliche Projekte mit verschiedenen PHP-Versionen testen möchte, denn die Einrichtung mehrerer Versionen auf einem Rechner kann schnell unübersichtlich werden. Außerdem fehlen einem direkt Features wie ein vorinstallierter Webserver oder praktische Tools, die die Arbeit erleichtern.

### 2. XAMPP nutzen

XAMPP ist eine fertige **Software-Suite, die Apache, PHP und weitere Tools zusammenbringt**. Für Einsteiger ist es ein echter Gewinn, denn man kann direkt loslegen: Der Apache-Webserver ist eingerichtet, PHP funktioniert sofort, und alles läuft größtenteils ohne weitere Konfiguration. Nachteilig ist jedoch, dass man sich ein Stück weit an die XAMPP-Struktur gewöhnt, die von der Realität auf einem echten Server abweichen kann. Auch Updates oder das parallele Nutzen mehrerer PHP-Versionen sind komplizierter als bei einer sauberen Installation.

### 3. Docker verwenden

Wer etwas experimentierfreudiger ist, kann PHP auch **in einem Docker-Container laufen lassen**. Dabei bekommt man eine **isolierte und reproduzierbare Umgebung**, die sich leicht für verschiedene PHP-Versionen konfigurieren lässt. Docker hält das eigene System sauber, weil keine dauerhafte Installation nötig ist, und ermöglicht einen Blick auf die modernen Deployment-Workflows, wie sie in der Praxis üblich sind. Zwar muss man sich am Anfang mit Containern und Images vertraut machen, aber schon nach kurzer Einarbeitung merkt man, wie flexibel und mächtig diese Methode ist – perfekt, um die eigenen Projekte zukunftssicher aufzusetzen.

### Docker auf MacOS und Linux installieren

Die Installation von Docker auf MacOS oder Linux ist inzwischen recht unkompliziert. Am besten schaut man direkt in die [offizielle Dokumentation](https://docs.docker.com/get-docker/). Kurz zusammengefasst geht man in drei Schritten vor: Docker herunterladen, installieren und den Dienst starten. Danach kann man sofort Container starten, Images ziehen und loslegen. Windows lassen wir bewusst außen vor – Long live Unix! 🐧

Hier ist ein Beispiel, wie man einen **PHP-Container starten** kann, um direkt im aktuellen Projekt zu arbeiten:

```bash
docker run --rm -it -v $(pwd):/var/www -w /var/www -p 8000:8000 php:8.5-cli /bin/bash
```

__Erklärung der einzelnen Teile:__
* `docker run` → startet einen neuen Container
* `--rm` → löscht den Container automatisch, sobald er stoppt, damit dein System sauber bleibt
* `-it` hält die Eingabe offen, sodass der Container auf deine Befehle wartet
* `-v \$(pwd):/var/www` → mountet dein aktuelles Arbeitsverzeichnis  ins Container-Verzeichnis /var/www, damit der Container Zugriff auf deine Dateien hat
* `-w /var/www` → setzt das Arbeitsverzeichnis innerhalb des Containers, sodass du direkt im Projektordner landest
* `-p 8000:8000` → bindet Port 8000 des Containers an Port 8000 deines Hosts, praktisch für den PHP Built-in Server
* `php:8.5-cli` → das PHP-Image, das im Container ausgeführt wird
* `/bin/bash` startest du eine Bash-Shell, in der du direkt PHP-Befehle eingeben oder Skripte ausführen kannst.

Mit diesem Command kannst du direkt PHP-Skripte ausführen oder den PHP Built-in Server starten, ohne dass du PHP dauerhaft auf deinem Rechner installieren musst.

### Praxis: Interaktiver Modus & Built-in Server

Nachdem du jetzt weißt, wie man einen PHP-Container startet und die einzelnen Teile des Docker-Commands versteht, kann es direkt praktisch werden.

**Interaktiver PHP-Modus**

Wir starten den **interaktiven PHP-Modus**, um PHP-Code direkt im Container auszuprobieren:

```bash
php -a
```

```php
echo "Hello World\n";
```

* `php -a` startet den interaktiven PHP-Modus (REPL) direkt in der Shell.

Danach kannst du direkt PHP-Code eintippen und Ergebnisse live sehen. Mit `exit` oder `Ctrl+D` verlässt du den interaktiven Modus wieder. Praktisch für Einsteiger: kein Skript nötig, direkt experimentieren und Feedback sehen.

**PHP Built-in Server**

Als Nächstes schauen wir uns an, wie man den eingebauten PHP-Server im Container startet, um kleine Webprojekte direkt lokal im Browser zu testen:

```bash
php -S 0.0.0.0:8000
```

* `php -S 0.0.0.0:8000` startet den PHP Built-in Server, der auf allen Netzwerkadressen (0.0.0.0) auf Port 8000 lauscht.
* Mit `Ctrl+C` stoppst du den Server wieder.

Jetzt kannst du deinen Browser öffnen und http://localhost:8000 aufrufen, um deine PHP-Dateien live zu sehen.

Alle Requests und Server-Aktivitäten werden direkt in der CLI angezeigt – super praktisch, um zu sehen, welche Dateien geladen werden, und einfache Fehler zu debuggen.

Dieses Setup ist ideal für Einsteiger, um lokale Webprojekte auszuprobieren, ohne Apache oder Nginx konfigurieren zu müssen. Dein Container bleibt sauber und du hast alles isoliert von deinem System.

Die klassischen Wege wie **PHP direkt auf dem Rechner** oder **XAMPP** haben uns gute Dienste geleistet: Sie machen den Einstieg einfach, erklären die Grundlagen und lassen uns schnell erste Skripte laufen. Für viele kleine Projekte sind sie weiterhin solide Optionen.

Mit **Docker** geht es einen Schritt weiter: isolierte, saubere Umgebungen, schnelle Wechsel zwischen PHP-Versionen und ein Blick auf moderne Entwicklungs-Workflows. Besonders für Einsteiger lohnt es sich, jetzt schon einmal hineinzuschnuppern, denn die Kenntnisse zahlen sich später bei größeren Projekten aus.

In zukünftigen Posts werden wir noch tiefer in Docker einsteigen, zusätzliche praktische Tipps geben und zeigen, wie man auch komplexere PHP-Umgebungen elegant aufsetzt. Bleib gespannt!
