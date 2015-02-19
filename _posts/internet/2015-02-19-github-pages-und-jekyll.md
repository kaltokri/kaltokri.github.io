---
layout: post
category : Internet
title: GitHub pages und jekyll
tagline: "Kostenloser Webspace mit eingebautem Blogsystem"
tags : [jekyll, github pages, blog]
---
{% include JB/setup %}

# Vorwort

Seit einiger Zeit betreibe ich verschiedene Webseiten mit [Joomla][]. Jetzt wollte ich für mich mal eine Alternative testen,
bei der man nicht immer wieder Updates einspielen muss. Nachdem ich mir verschiedene CMS Systeme angeschaut hatte bin
ich per Zufall über [GitHub pages][] und [Jekyll][] gestolpert. Diese Seite ist mittels [Jekyll][] erstellt worden und
wird auf [GitHub pages][] gehosted.

# GitHub pages

Die meisten von uns kennen [GitHub][] bereits. Falls Sie damit nichts anfangen können, hier eine
kleine Erklärung von Wikipedia:

Seite [„GitHub“](http://de.wikipedia.org/w/index.php?title=GitHub&oldid=136725990) Wikipedia, Bearbeitungsstand:
12. Dezember 2014, 21:20 UTC. (Abgerufen: 8. Februar 2015, 10:08 UTC)

> GitHub ist ein webbasierter Hosting-Dienst für Software-Entwicklungsprojekte. Namensgebend ist das
> Versionsverwaltungs-System Git.

[GitHub pages][] ist eine Erweiterung die darauf spezialisiert ist eine Webseite direkt aus einem git repository zu
veröffentlichen. Alles was man tun muss ist ein neues GitHub repository zu erstellen und bei der Namensgebung einem
bestimmten Schema zu folgen:

    <gewünschter Name>.github.io

also z.B. [kaltokri.github.io][]

Wenn man nun eine html-Datei dem Repository hinzufügt und sie hoch lädt, kann man sie über die gewählte URl erreichen.
Somit ist man in der Lage eine statische Webseite kostenlos zu veröffentlichen. Und diese Webseite ist über git
natürlich versionsverwaltet. Das an sich ist schon eine tolle Sache, aber [GitHub pages][] kann noch viel mehr.

Es gibt einen [automatischen Generator][GitHubGenerator] der eine Webseite mit Hilfe einer wählbaren Vorlage erstellt.
So erhält man einen einfachen und schnellen Einstieg.

[GitHub pages][] liefert aber auch noch ein komplettes Blog-System mit: [Jekyll][]

Jedes [GitHub pages][] Repository unterstützt automatisch [Jekyll][], sofern die Verzeichnisstrukturen richtig angelegt
werden.

# Was ist Jekyll?

[Jekyll][] ist ein Generator für statische Webseiten im Blog Stil. [Jekyll][] ist geschrieben in [Ruby][] und besteht
aus dynamischen Komponenten wie z.B. Templates und statischen Daten wie z.B. [Markdown][] Dokumenten.

Jekyll benötigt keine Datenbank. Die Artikel werden mit einem einfachen Texteditor geschrieben, in git verwaltet und
hoch geladen. Durch die Templates wird eine optisch ansprechende Webseite daraus generiert. Ein Backup ist nicht nötig,
da das git Repository bereits auf unserem Rechner und in [GitHub][] existiert.

## Lokale Installation

Man sollte [Jekyll][] auf dem lokalen Rechner installieren. Zum einen möchte man sich ja in die Besonderheiten
einarbeiten und verschiedenes testen, aber zusätzlich braucht man die lokale Installation auch als Vorschau, damit man
die Artikel prüfen kann, bevor man sie nach [GitHub][] hoch lädt.

Meine Versuche der Installationsanleitung zu folgen und Ruby und alle benötigten Komponenten zu installieren sind
fehlgeschlagen. Immer wieder traten merkwürdige Fehlermeldungen auf. Dank goolge-Suche kam ich zwar immer einen Schritt
weiter, aber als dann nach 2 Stunden mehrere Komponenten in (für Jekyll) inkompatiblen Versionen installiert waren, war
ich kurz davor aufzugeben.

Dann fand ich aber [PortableJekyll][]. Ein git Repository, dass alle nötigen Komponenten in kompatiblen Versionen
zusammen gestellt hat, so dass man sofort loslegen kann. Einfach das repository mittels git auf den lokalen Rechner
clonen oder als Zip herunter laden und entpacken. Da es wirklich alles enthält was man braucht ist das Paket leider 2 GB
groß. Aber es läuft auf Anhieb und kann natürlich auch auf einer externen Festplatte betrieben werden.

## Eine Instanz erzeugen
Um eine neue Jekyll-Umgebung zu erzeugen startet man eine Eingabeaufforderung wechselt in das Verzeichnis von
[PortableJekyll][] und startet folgende Batch Datei:

    setpath.cmd

Dadurch werden die Umgebungsvariable in der Eingabeaufforderung so angepaßt, dass Jekyll ausgeführt werden kann.
Außerhalb der Eingabeaufforderung hat das keinen Einfluss. D.h. es wird keine Einstellung des Betriebssystems dauerhaft
verändert!

Dann erzeugt man sich eine neue Jekyll Instanz mittels

    jekyll new myblog

Dieser Befehl erzeugt einen Ordner mit dem Namen myblog, wobei myblog durch einen beliebigen Namen ersetzt werden kann.
Darin wird die komplette Verzeichnisstruktur für Jekyll, inklusive aller benötigten Dateien, erstellt.

Als nächstes wollen wir uns diese neu erstellte Instanz mal ansehen. Wir wechseln in den Ordner myblog und führen
folgenden Befehl aus:

    jekyll serve

Dieser Befehl startet einen Jekyll Server auf den lokalen Rechner der unter http://localhost:4000 erreichbar ist.

Jetzt kann man sich in Ruhe in Jekyll einarbeiten und verschiedene Dinge ausprobieren.

## Weitere Informationen
Auf der Jekyll Webseite findet sich eine ausführliche Dokumentation in englisch. Ich werde in zukünftigen Blog-Artikeln
weiter auf die Einrichtung und Anpassung von Jekyll eingehen und beschreiben wie man Inhalte hinzufügen kann.

[Joomla]: http://www.joomla.de/ "Joomla"
[kaltokri.github.io]: http://kaltokri.github.io/ "Private Webseite von kaltokri"
[GitHub]: https://github.com/ "GitHub"
[GitHub pages]: https://pages.github.com/ "GitHub pages"
[GitHubGenerator]: https://help.github.com/articles/creating-pages-with-the-automatic-generator/ "GitHub Generator"
[Jekyll]: http://jekyllrb.com/ "Jekyll"
[Ruby]: https://www.ruby-lang.org/de/ "Ruby"
[Markdown]: http://markdown.de/ "Markdown"
[PortableJekyll]: https://github.com/madhur/PortableJekyll "PortableJekyll"
