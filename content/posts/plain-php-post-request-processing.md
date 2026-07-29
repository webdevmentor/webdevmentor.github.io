---
title: "POST Requests in Plain PHP sauber verarbeiten"
date: 2026-07-28
draft: false
description: "Erfahre, wie du POST Requests in Plain PHP mithilfe von Request DTO, Validator, Mapper und Domain Model sauber und wartbar verarbeitest."
tags:
  - php
  - plain-php
  - architecture
  - oop
  - request
categories:
  - PHP
---

Wer mit PHP arbeitet, kommt früher oder später mit **POST (Hypertext Transfer Protocol POST)** Requests in Berührung. Immer dann, wenn ein Benutzer Daten an einen Server sendet – beispielsweise über ein Kontaktformular, eine Registrierung oder einen Login – kommt in der Regel ein POST Request zum Einsatz.

Im Gegensatz zu einem **GET (Hypertext Transfer Protocol GET)** Request, der hauptsächlich zum Abrufen von Daten verwendet wird, dient ein POST Request dazu, neue Daten zu übermitteln oder bestehende Daten zu verändern. Die eigentliche Verarbeitung beginnt auf dem Server, wo PHP die übermittelten Werte unter anderem über das globale Array `$_POST` bereitstellt.

Für kleine Projekte genügt es häufig, die Werte direkt aus `$_POST` auszulesen. Mit zunehmender Projektgröße verteilt sich diese Logik jedoch schnell über mehrere Dateien. Das Auslesen der Daten, das Umwandeln in die richtigen Datentypen, die Validierung und schließlich die Erstellung von Objekten vermischen sich häufig in einer einzigen Datei.

Genau an diesem Punkt setzt dieser Artikel an.

Wir entwickeln **kein Framework** und bauen auch keine vollständige Anwendung. Stattdessen betrachten wir ausschließlich den Weg eines POST Requests – vom Eingang der Daten bis zu einem fertigen Domain-Model.

Dabei kommen vier kleine Klassen zum Einsatz, die jeweils genau eine Aufgabe übernehmen:

* **Request DTO (Data Transfer Object)** – nimmt die eingehenden Daten entgegen und übernimmt das Casting in die passenden Datentypen.
* **Validator** – prüft, ob die Daten vollständig und fachlich gültig sind.
* **Mapper** – übersetzt das Request DTO in das Domain-Model.
* **Domain Model** – repräsentiert die eigentlichen Fachobjekte der Anwendung.

Alles, was nach diesem Schritt passiert – etwa das Speichern in einer Datenbank, das Versenden einer E-Mail oder der Aufruf einer **API (Application Programming Interface)** – gehört bewusst **nicht** zum Thema dieses Artikels.

Der vollständige Beispielcode ist im folgenden Repository verfügbar:

**Repository:** https://github.com/webdevmentor/post-request-example

Im weiteren Verlauf verfolgen wir Schritt für Schritt den Weg eines POST Requests durch diese vier Klassen und sehen uns an, warum diese Trennung den Code übersichtlicher und wartbarer macht.

----

## Das Request DTO

Der erste Schritt nach dem Eingang eines POST Requests ist die Überführung der Daten in eine eigene Struktur.

PHP stellt die eingehenden Daten zunächst als Array über `$_POST` bereit. Dieses Array kommt direkt aus der Außenwelt und enthält zunächst nur Werte, die vom Client übertragen wurden. Für die weitere Verarbeitung in unserer Anwendung ist diese Form jedoch nicht besonders komfortabel.

Genau hier kommt das **Request DTO (Data Transfer Object)** zum Einsatz.

Ein Request DTO hat eine einfache Aufgabe: Es nimmt die Daten aus dem eingehenden Request entgegen und überführt sie in eine definierte Struktur. Dabei können bereits erste technische Anpassungen erfolgen, beispielsweise das Entfernen von Leerzeichen oder die Umwandlung in die benötigten Datentypen.

Das Request DTO kennt dabei ausschließlich die Struktur der eingehenden Daten. Es entscheidet nicht, ob die Daten fachlich gültig sind. Diese Aufgabe übernimmt später der Validator.

Der Ablauf sieht vereinfacht so aus:

<img src="static/images/request-flow.svg?qb=1234" style="max-height: 320px;">

Im Beispielprojekt wird ein Gästebuch-Eintrag verarbeitet. Deshalb heißt die konkrete Klasse `GuestbookEntryRequest`. Der Name beschreibt den Anwendungsfall des Beispiels, während die Aufgabe der Klasse weiterhin die eines allgemeinen Request DTOs bleibt.

Der erste Einsatz des Request DTOs befindet sich im Repository:

https://github.com/webdevmentor/post-request-example/tree/step/02-introduction-request-dto

Die Klasse sieht in vereinfachter Form so aus:

```php
final class GuestbookEntryRequest
{
    public function __construct(
        public string $firstname,
        public string $lastname,
        public string $homepage,
        public string $twitter,
        public string $message,
    ) {}
}
```

Bereits der Constructor zeigt eine wichtige Eigenschaft des Request DTOs: Die erwarteten Daten sind explizit definiert.

Statt später an verschiedenen Stellen mit beliebigen Array-Schlüsseln zu arbeiten:

```php
$post['firstname']
$post['message']
```

existiert nun ein Objekt mit klar beschriebenen Eigenschaften:

```php
$request->firstname;
$request->message;
```

Die Anwendung arbeitet dadurch nicht mehr direkt mit dem ursprünglichen Request-Array, sondern mit einer festen Struktur.

Durch die Typdefinitionen ist außerdem sichtbar, welche Werte erwartet werden. Ein Vorname ist beispielsweise ein `string`, genau wie eine Nachricht oder eine Homepage URL (Uniform Resource Locator).

Damit ist die erste Verantwortung des Request DTOs erfüllt: Aus unstrukturierten Request-Daten wird ein definiertes Objekt.

Der nächste Schritt ist die Erstellung dieses Objekts aus den tatsächlichen POST-Daten.

Dafür verwendet das Beispiel eine eigene Factory-Methode:

```php
public static function fromPost(array $post): self
{
    return new self(
        firstname: trim($post['firstname'] ?? ''),
        lastname: trim($post['lastname'] ?? ''),
        homepage: trim($post['homepage'] ?? ''),
        twitter: trim($post['twitter'] ?? ''),
        message: trim($post['message'] ?? ''),
    );
}
```

Die Methode `fromPost()` kapselt den Übergang zwischen dem PHP-Array und unserem Objekt.

Hier passieren bereits erste technische Verarbeitungsschritte:

- Fehlende Werte erhalten einen Standardwert (`''`).
- Leerzeichen am Anfang und Ende werden entfernt.
- Die Werte werden direkt den passenden Eigenschaften des DTOs zugeordnet.

Diese Schritte sind bewusst technisch gehalten. Das Request DTO bereitet die Daten für die Anwendung auf, trifft aber noch keine fachlichen Entscheidungen.

Ein leerer Name wird hier nicht abgelehnt. Eine ungültige Homepage wird hier nicht geprüft.

Das DTO sagt lediglich:

> "Diese Daten sind angekommen und liegen jetzt in einer einheitlichen Struktur vor."

Ob die Daten für unsere Anwendung gültig sind, entscheidet der nächste Schritt: der Validator.

Den vollständigen Stand dieses Schritts findest du im Repository:

https://github.com/webdevmentor/post-request-example/tree/step/02-introduction-request-dto

----

## Der Validator

Nachdem die Daten aus dem POST Request in ein Request DTO überführt wurden, liegt uns eine strukturierte Darstellung der Eingaben vor.

Im ersten Beispiel wurden die Daten bereits validiert. Das ist ein wichtiger Schritt, denn eine Anwendung sollte nicht einfach beliebige Eingaben weiterverarbeiten.

Das Problem ist jedoch nicht die Validierung selbst, sondern deren Platzierung.

Wenn die Prüfung direkt zwischen dem Einlesen des Requests und der weiteren Verarbeitung stattfindet, vermischen sich verschiedene Verantwortlichkeiten:

- Der Request muss entgegengenommen werden.
- Die Daten müssen in eine Struktur gebracht werden.
- Die Eingaben müssen fachlich geprüft werden.

Mit dem nächsten Schritt wird diese Verantwortung aus der bisherigen Verarbeitung herausgelöst und in eine eigene Klasse verschoben: den **Validator**.

Der Validator kennt das Request DTO und überprüft ausschließlich, ob die enthaltenen Daten die erwarteten Regeln erfüllen.

Der Ablauf erweitert sich damit:

<img src="static/images/request-flow-validator.svg?qb=14" style="max-height:320px;">

Im Beispielprojekt heißt die konkrete Klasse `GuestbookEntryValidator`. Auch hier verwenden wir im Artikel weiterhin den allgemeinen Begriff **Validator**, da die Aufgabe unabhängig vom Gästebuch-Beispiel gleich bleibt.

Die Klasse sieht vereinfacht so aus:

```php
final class GuestbookEntryValidator
{
    public static function validate(GuestbookEntryRequest $request): array
    {
        $errors = [];

        if ($request->firstname === '' && $request->lastname === '') {
            $errors[] = 'Bitte gib deinen Namen ein.';
        }

        if ($request->message === '') {
            $errors[] = 'Bitte gib eine Nachricht ein.';
        }

        if (mb_strlen($request->message) > 500) {
            $errors[] = 'Die Nachricht darf maximal 500 Zeichen lang sein.';
        }

        return $errors;
    }
}
```

Die Aufgabe der Klasse ist bewusst überschaubar: Sie nimmt ein Request DTO entgegen und liefert eine Liste von Fehlern zurück.

Dabei verändert der Validator die Daten nicht.

Er macht aus einer ungültigen Nachricht keine gültige Nachricht. Er kürzt keine Texte und ergänzt keine fehlenden Werte.

Er beantwortet lediglich die Frage:

> "Erfüllen diese Daten die Regeln unserer Anwendung?"

Im Beispiel werden drei Regeln geprüft:

- Es muss entweder ein Vorname oder ein Nachname angegeben werden.
- Eine Nachricht darf nicht leer sein.
- Eine Nachricht darf maximal 500 Zeichen enthalten.

Das Ergebnis der Prüfung ist ein Array mit Fehlermeldungen.

Ein leeres Array bedeutet:

```php
[]
```

Die Daten sind gültig.

Enthält das Array Einträge, wurden Regeln verletzt:

```php
[
    'Bitte gib eine Nachricht ein.',
]
```

Die weitere Verarbeitung kann anhand dieses Ergebnisses entscheiden, ob mit den Daten fortgefahren wird.

Auch hier bleibt die Verantwortung klar getrennt:

- Das Request DTO beschreibt die eingegangenen Daten.
- Der Validator prüft diese Daten.
- Die nächste Schicht entscheidet, was mit gültigen Daten passiert.

Den vollständigen Stand dieses Schritts findest du im Repository:

https://github.com/webdevmentor/post-request-example/tree/step/03-extract-validator

----

## Der Mapper

Nach dem Request DTO und der Validierung befinden sich die Daten in einem guten Zustand.

Wir wissen jetzt:

- Die Daten wurden aus dem POST Request übernommen.
- Die Werte befinden sich in einer festen Struktur.
- Die Eingaben erfüllen unsere Validierungsregeln.

Trotzdem sind wir noch nicht am Ziel.

Das Request DTO beschreibt weiterhin die Daten aus Sicht des eingehenden Requests. Es ist ein Objekt der Transportebene.

Unsere Anwendung benötigt jedoch ein Objekt, das die eigentliche Fachlichkeit beschreibt: das Domain-Model.

Genau für diese Übersetzung ist der **Mapper** zuständig.

Der Mapper verbindet zwei unterschiedliche Welten:

<img src="static/images/request-dto-to-domain-model.svg?qb=ß090938928" style="max-height:200px;">

Dabei kopiert ein Mapper nicht einfach nur Werte von einem Objekt in ein anderes. Er kann auch Anpassungen durchführen, die für das Domain-Model sinnvoll sind.

Im Beispielprojekt heißt die konkrete Klasse `GuestbookEntryMapper`. Die allgemeine Aufgabe bleibt jedoch immer gleich: Ein Request DTO wird in ein Domain-Model überführt.

Die Klasse sieht vereinfacht so aus:

```php
final class GuestbookEntryMapper
{
    public static function fromRequest(
        GuestbookEntryRequest $request
    ): GuestbookEntry {

        $displayName = trim(
            $request->firstname . ' ' . $request->lastname
        );

        return new GuestbookEntry(
            displayName: $displayName,
            homepage: $request->homepage !== ''
                ? $request->homepage
                : null,
            twitter: $request->twitter !== ''
                ? $request->twitter
                : null,
            message: $request->message,
        );
    }
}
```

Schon an der Signatur der Methode erkennen wir die Aufgabe:

```php
fromRequest(GuestbookEntryRequest $request): GuestbookEntry
```

Die Methode nimmt ein Request DTO entgegen und liefert ein Domain-Model zurück.

Der Mapper ist damit die Stelle, an der entschieden wird, wie aus den eingehenden Daten ein Objekt der Anwendung entsteht.

Ein gutes Beispiel dafür sind optionale Felder.

Ein HTML-Formular liefert bei leeren Eingaben häufig einen leeren String:

```php
''
```

Für die weitere Verarbeitung ist es jedoch oft hilfreicher, zwischen zwei Zuständen unterscheiden zu können:

```php
null
```

bedeutet:

> Es wurde kein Wert angegeben.

während

```php
''
```

lediglich bedeutet:

> Es existiert ein String, der keine Zeichen enthält.

Der Mapper übernimmt diese kleine, aber wichtige Umwandlung:

```php
homepage: $request->homepage !== ''
    ? $request->homepage
    : null,
```

Dadurch erhält das Domain-Model eine klarere Darstellung der Daten.

Auch der Anzeigename wird erst an dieser Stelle zusammengesetzt:

```php
$displayName = trim(
    $request->firstname . ' ' . $request->lastname
);
```

Das Request DTO kennt lediglich die einzelnen Eingabefelder aus dem Formular.

Die Bedeutung dieser Felder entsteht erst beim Mapping in das Domain-Model.

Genau deshalb gehört diese Logik nicht in das Request DTO.

Das DTO sagt:

> "Diese Daten wurden übermittelt."

Der Mapper sagt:

> "So werden diese Daten innerhalb unserer Anwendung verwendet."

Das Domain-Model selbst muss anschließend nicht mehr wissen, woher die Daten ursprünglich kamen.

Den vollständigen Stand dieses Schritts findest du im Repository:

https://github.com/webdevmentor/post-request-example/tree/step/04-introduce-domain-object

----

## Das Domain-Model

Nach Request DTO, Validator und Mapper ist der eigentliche Übergang abgeschlossen.

Die Daten wurden:

- aus dem POST Request übernommen,
- in eine definierte Struktur gebracht,
- validiert,
- und in ein Objekt der Anwendung übersetzt.

An diesem Punkt spielt die ursprüngliche Quelle der Daten keine Rolle mehr.

Das Domain-Model weiß nicht, ob die Daten ursprünglich aus einem HTML-Formular, einer API oder einer anderen Quelle stammen. Es beschreibt ausschließlich die Informationen, die unsere Anwendung benötigt.

Im Beispielprojekt wird ein Gästebuch-Eintrag durch die Klasse `GuestbookEntry` dargestellt.

Die Klasse sieht bewusst einfach aus:

```php
final class GuestbookEntry
{
    public function __construct(
        public string $displayName,
        public ?string $homepage,
        public ?string $twitter,
        public string $message
    ) {}
}
```

Auf den ersten Blick wirkt diese Klasse vielleicht unspektakulär. Genau das ist aber ihre Stärke.

Ein Domain-Model muss nicht automatisch eine große Klasse mit vielen Methoden sein. Seine wichtigste Aufgabe ist es, die Daten und die Bedeutung eines Fachobjekts innerhalb der Anwendung abzubilden.

In unserem Beispiel beschreibt ein `GuestbookEntry` genau die Informationen, die ein Eintrag im Gästebuch benötigt:

- einen Anzeigenamen,
- optional eine Homepage,
- optional einen Twitter-Account,
- und die eigentliche Nachricht.

Auffällig sind die optionalen Werte:

```php
public ?string $homepage,
public ?string $twitter,
```

Das `?string` bedeutet, dass diese Eigenschaften entweder einen String oder `null` enthalten können.

Das passt zu der Entscheidung aus dem Mapper: Ein nicht ausgefülltes optionales Feld wird nicht als leerer String gespeichert, sondern als fehlender Wert dargestellt.

Dadurch bleibt das Domain-Model eindeutig:

```php
null
```

bedeutet:

> Es gibt keinen Wert.

Ein leerer String dagegen würde bedeuten:

> Es gibt einen Wert, aber dieser besteht aus keinen Zeichen.

Diese Unterscheidung mag bei einem kleinen Gästebuch unwichtig wirken. In größeren Anwendungen wird sie jedoch schnell relevant.

Der wichtigste Punkt ist aber die Trennung der Verantwortlichkeiten:

- Das Request DTO kennt die eingehenden Daten.
- Der Validator kennt die Regeln für gültige Eingaben.
- Der Mapper kennt die Übersetzung zwischen beiden Welten.
- Das Domain-Model repräsentiert die eigentliche Fachlichkeit.

Damit ist der POST Request verarbeitet, ohne dass eine einzelne Klasse alle Aufgaben übernehmen musste.

Den vollständigen Stand dieses Schritts findest du im Repository:

https://github.com/webdevmentor/post-request-example/tree/step/05-add-domain-model
