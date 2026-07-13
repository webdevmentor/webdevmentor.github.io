---
title: "Docker für PHP, Apache und MariaDB – aus einem Container wird eine Entwicklungsumgebung"
date: '2026-07-13T20:31:00+02:00'
draft: false
tags: ["PHP", "Apache", "Docker", "MySQL", "PDO", "MariaDB", "Entwicklungsumgebung"]
---

Im [letzten Beitrag](https://www.webdevmentor.info/posts/php-apache-docker/) unserer Docker-Serie haben wir eine einfache PHP-Entwicklungsumgebung mit Docker erstellt. Für kleine Projekte oder erste Experimente ist das bereits völlig ausreichend und hat den großen Vorteil, dass keine lokale PHP-Installation mehr notwendig ist.

Sobald eine Anwendung Daten speichern soll, stößt dieses einfache Setup allerdings an seine Grenzen. Neben dem Webserver benötigen wir nun auch eine Datenbank. Außerdem muss PHP die passenden Erweiterungen mitbringen, damit später überhaupt eine Verbindung zur Datenbank aufgebaut werden kann.

In diesem Beitrag erweitern wir deshalb unsere bestehende Docker-Umgebung um einen MariaDB-Container und passen unseren Webcontainer entsprechend an.

## Unsere compose.yml erweitern

Im letzten Beitrag bestand unsere `compose.yml` lediglich aus einem Webcontainer.

```yaml
services:
  web:
    image: php:8.5-apache
    ports:
      - "8080:80"
    volumes:
      - .:/var/www/html
```

Für unsere Datenbank ergänzen wir nun einen zweiten Service.

```yaml
services:
  web:
    build: .
    ports:
    - "8080:80"
    volumes:
      - .:/var/www/html

  mariadb:
    image: mariadb:10.11
    environment:
      MARIADB_ROOT_PASSWORD: root
      MARIADB_DATABASE: app
      MARIADB_USER: app
      MARIADB_PASSWORD: secret
    volumes:
      - mariadb_data:/var/lib/mysql

volumes:
  mariadb_data:
```

Aus einem einzelnen Container ist damit eine kleine Entwicklungsumgebung geworden. Der Webcontainer kümmert sich um PHP und Apache, während MariaDB unsere Daten speichert.

## Warum braucht die Datenbank ein eigenes Volume?

Vielleicht ist dir der neue Abschnitt am Ende der Datei bereits aufgefallen.

```yaml
volumes:
  mariadb_data:
```

Ein Docker-Container ist grundsätzlich austauschbar. Wird ein Container gelöscht, gehen auch alle darin gespeicherten Daten verloren.

Damit uns das mit unserer Datenbank nicht passiert, verwenden wir ein **Docker Volume**. Man kann es sich zunächst wie einen Speicherort vorstellen, den Docker unabhängig vom eigentlichen Container verwaltet.

Der MariaDB-Container speichert seine Daten nicht im Container selbst, sondern im Volume `mariadb_data`. Selbst wenn der Container später gelöscht oder neu erstellt wird, bleiben unsere Tabellen und Daten erhalten.

## Den Webcontainer anpassen

Vielleicht ist dir noch eine weitere Änderung aufgefallen.

Im letzten Beitrag haben wir den Webcontainer direkt über das `image`-Attribut gestartet.

```yaml
image: php:8.5-apache
```

Jetzt verwenden wir stattdessen:

```yaml
build: .
```

Der Grund ist einfach: Unser PHP-Container benötigt jetzt zusätzlich die Erweiterungen für die Kommunikation mit MySQL-kompatiblen Datenbanken. Dafür erstellen wir erstmals ein eigenes Dockerfile.

```dockerfile
FROM php:8.5-apache

RUN docker-php-ext-install mysqli pdo pdo_mysql
```

Mehr ist an dieser Stelle zunächst gar nicht notwendig.

Während der Entwicklung möchten wir unseren Quellcode möglichst komfortabel bearbeiten. Deshalb kopieren wir ihn nicht in das Image, sondern binden ihn direkt in den Container ein. Änderungen an den PHP-Dateien stehen dadurch sofort zur Verfügung und wir müssen das Image nicht nach jeder kleinen Anpassung neu erstellen.

Dafür verwenden wir dieses Volume:

```yaml
volumes:
  - .:/var/www/html
```

Diese Art der Einbindung bezeichnet Docker als **Bind Mount**. Dabei wird unser Projektverzeichnis direkt mit dem Verzeichnis `/var/www/html` im Container verbunden.

## Die Container starten

Sind beide Dateien vorbereitet, genügt wie gewohnt ein einziger Befehl.

```bash
docker compose up -d
```

Docker erstellt zunächst unser PHP-Image aus dem Dockerfile, lädt anschließend das MariaDB-Image herunter, legt das Docker Volume an und startet beide Container.

Mit

```bash
docker compose ps
```

lässt sich anschließend überprüfen, ob beide Dienste erfolgreich laufen.

## Wie findet PHP die Datenbank?

Obwohl Webserver und Datenbank in unterschiedlichen Containern laufen, können sie problemlos miteinander kommunizieren.

Docker Compose erstellt dafür automatisch ein gemeinsames Netzwerk. Jeder Service ist innerhalb dieses Netzwerks über seinen Servicenamen erreichbar. Unsere Datenbank ist deshalb nicht unter `localhost`, sondern unter `mariadb` erreichbar.

Eine Verbindung mit PDO könnte beispielsweise so aussehen:

```php
$dsn = 'mysql:host=mariadb;dbname=app;charset=utf8mb4';

$username = 'app';
$password = 'secret';

$pdo = new PDO($dsn, $username, $password);
```

Den Hostnamen müssen wir dabei nicht selbst konfigurieren. Docker übernimmt die interne Namensauflösung automatisch.

## Warum MariaDB?

In diesem Beitrag verwenden wir bewusst MariaDB. Die Datenbank ist vollständig Open Source, weit verbreitet und für die meisten PHP-Anwendungen ein direkter Ersatz für MySQL.

Soll stattdessen MySQL verwendet werden, genügt in den meisten Fällen bereits eine kleine Änderung.

```yaml
image: mysql:8.4
```

Außerdem müssen die Umgebungsvariablen von `MARIADB_` auf `MYSQL_` angepasst werden.

## Fazit

Mit wenigen zusätzlichen Zeilen ist aus unserem einzelnen PHP-Container eine vollständige Entwicklungsumgebung geworden. Docker Compose startet Webserver und Datenbank gemeinsam, Docker Volumes sorgen dafür, dass unsere Daten dauerhaft erhalten bleiben, und über das Dockerfile können wir unser PHP-Image an die Anforderungen unseres Projekts anpassen.

Damit steht eine solide Grundlage für die weitere Entwicklung bereit – und das alles mit einem einzigen Befehl.
