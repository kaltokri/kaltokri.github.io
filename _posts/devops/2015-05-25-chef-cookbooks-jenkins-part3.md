---
layout: post
category : DevOps
title: Chef Cookbooks mit Jenkins prüfen, Teil3
tagline: "RuboCop: Murphy's Law mal anders"
tags : [Chef, Ruby, Provisioning, Jenkins, RuboCop]
---
{% include JB/setup %}
<div class="toc"></div>

# Vorwort

Diesmal schauen wir uns das Tool [RuboCop][] an. [Chef][] Cookbooks sind [Ruby][]-Code. Zwar verwendet man
hauptsächlich eine DSL-Beschreibungssyntax um die einzelnen Ressourcen zu beschreiben, aber es bleibt
trotzdem [Ruby][]-Code der den gleichen Qualitätsanforderungen unterworfen werden sollte. Um die Einheitlichkeit von
[Ruby][]-Code sicherzustellen wurde der [Ruby Style Guide][] geschrieben. Er soll dabei helfen, dass Ruby-Code möglichst
immer gleich aussieht, so dass ein Entwickler fremden Code besser versteht und pflegen oder erweitern kann.

<img src="/assets/images/devops/rubocop.jpg" alt="RuboCop" style="box-shadow: none; background-color: #FFF;
margin: 20px 10px 10px 0px; border: 1px solid black;vertical-align: bottom;" align="left" width="150">

[RuboCop][] ist ein Tool mit dem man geschriebenen Ruby-Code gegen einen Satz von Regeln prüfen lässt, der auf dem
[Ruby Style Guide][] basiert. Das bereits im
[ersten Teil der Artikelreihe]({% post_url /devops/2015-04-07-chef-cookbooks-jenkins-part1 %}) vorgestellte
[Foodcritic][] ist eigentlich nur eine
Erweiterung von [RuboCop][] um ein paar Regeln die für die Besonderheiten der Chef Beschreibungs-DSL wichtig sind.
[RuboCop][] ist im [Chef-DK][] enthalten und kann somit nach der Installation direkt verwendet werden.

Es ist wichtig [RuboCop][] von Anfang an bei einem Projekt zu verwenden und die Einhaltung der Regeln ständig zu
überprüfen. Somit macht es Sinn unseren [Jenkins][]-Job um [RuboCop][] zu erweitern. Wer RuboCop bei einem großen
Projekt erst spät einführt wird feststellen, dass der Code unzählige Abweichungen von den Regeln beinhaltet. Deren
Beseitigung verschlingt viel Aufwand, erschwert u.U. spätere Analysen von Code-Änderungen (z.B. wenn man die Einrückung
global ändert) und bringt das Risiko einer Destabilisierung mit sich. Ob in einem späten Stadium die Einführung noch
sinnvoll ist, muss individuell entschieden werden. Allerdings kann es u.U. sinnvoll sein nur bestimmte Regeln global zu
erzwingen und/oder den Status Quo fest zu halten (siehe eigener Abschnitt) und nur bei neuem Code auf die Einhaltung zu
bestehen.

# Erster Lauf
Öffnen Sie eine Eingabeaufforderung und gehen Sie in den Ordner des Cookbooks. Der folgende Befehl startet die Analyse:

    rubocop

Das Ergebnis bei einem kleinen Projekt sieht z.B. so aus:

    Inspecting 3 files
    CCC

    Offenses:

    Berksfile:1:8: C: Prefer single-quoted strings when you don't need string interpolation or special symbols.
    source "https://supermarket.getchef.com"
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    metadata.rb:1:5: C: Put one space between the method name and the first argument.
    name             'chefdk_getting_started'
        ^^^^^^^^^^^^^
    metadata.rb:2:11: C: Put one space between the method name and the first argument.
    maintainer       'The Authors'
              ^^^^^^^
    metadata.rb:4:8: C: Put one space between the method name and the first argument.
    license          'all_rights'
           ^^^^^^^^^^
    metadata.rb:5:12: C: Put one space between the method name and the first argument.
    description      'Installs/Configures chefdk_getting_started'
               ^^^^^^
    metadata.rb:7:8: C: Put one space between the method name and the first argument.
    version          '0.1.0'
           ^^^^^^^^^^
    metadata.rb:9:9: C: Prefer single-quoted strings when you don't need string interpolation or special symbols.
    depends "tomcat"
            ^^^^^^^^
    recipes/default.rb:8:16: C: Final newline missing.
    package 'httpd'

In der ersten Zeile wird die Anzahl der Dateien ausgegeben. Darunter wird für jede analysierte Datei ein Buchstabe
geschrieben. Dabei steht `.` für eine saubere Datei, die Großbuchstaben stehen für den Typ des gefundenen Problems:
`C`onvention, `W`arning,`E`rror oder `F`atal.

Dann wird für jeden Verstoß die Datei, Zeile, Spalte, Typ und die betroffene Regel ausgegeben. Gefolgt von einer
Code-Ausgabe mit Markierung. Wenn man sich ein wenig an diese Auflistung gewöhnt hat und nicht zu viele Verstöße
gefunden werden kann man damit schon ganz gut arbeiten. Für den [Jenkins][] taugt diese Ausgabe aber nichts.

# Formatter
Es gibt verschiedene Ausgabeformate, die man mittels Formatter erzeugen lassen kann. Sehr schön finde ich den **HTML
Formatter**, mit dem man eine übersichtliche Darstellung der Verstöße als HTML-Datei erhält:

    rubocop --format html -o target/rubocop.html

Oder den **Offense Count Formatter** der eine Übersicht erzeugt, welche die Anzahl der Verstöße pro Regel auflistet:

    rubocop --format offenses

Beide sind im [Chef-DK][] enthalten und können sofort genutzt werden.

Aber für unsere automatisierte Überprüfung brauchen wir den [CheckstyleFormatter][].

# CheckstyleFormatter
Dieser Formatter ist leider nicht im [Chef-DK][] enthalten. Aber man kann ihn leicht nachinstallieren:

    gem install rubocop-checkstyle_formatter

Diese Installation und das folgende Kommando bindet man als Buildschritte mit in den [Jenkins][]-Build-Job ein:

    rubocop --require rubocop/formatter/checkstyle_formatter --format RuboCop::Formatter::CheckstyleFormatter --no-color --rails --out target/checkstyle.xml

Die Auswertung erfolgt dann über das [Violations plugin][]. Damit erhält man eine sehr schöne Übersicht der gefundenen
Probleme.

# Keine Regel ohne Ausnahme
Es gibt Situationen in denen man bestimmte Regeln nicht einhalten kann oder sich bewusst gegen die Einhaltung
entscheidet. Dann hat man in seinem Projekt Warnmeldungen die man abschalten möchte. Die Konfiguration von [RuboCop][]
gibt einem verschiedene Lösungsmöglichkeiten dafür an die Hand.

## .rubocop.yml

Diese Datei kann man überall im Projekt oder im Home-Verzeichnis des Benutzers erstellen und über den Inhalt das
Verhalten von [RuboCop][] steuern. [RuboCop][] sucht zunächst diese Datei in dem Ordner in dem die aktuell zu
analysierende Datei liegt. Danach geht er Ebene für Ebene höher, bis er auf das Hauptverzeichnis trifft. Und so könnte
die Datei aussehen:

{% highlight yaml %}
inherit_from: ../.rubocop.yml

Style/Encoding:
  Enabled: false

Metrics/LineLength:
  Max: 99
{% endhighlight %}

Über `inherit_from` können alle Regeln einer anderen Datei importiert und dann mit den weiter unten aufgeführten Regeln
überschrieben werden. Man kann auch von mehreren Dateien Regeln erben. Diese Angabe ist optional!

Unter dem `inherit_from` kann man dann die Regeln konfigurieren und/oder deaktivieren. Im Beispiel oben wird das
Encoding nicht mehr überprüft und die zulässige Zeilenlänge wird von 80 auf 99 Zeichen erweitert. Eine Übersicht der
Konfigurationsmöglichkeiten findet man in der Datei [config/default.yml](https://github.com/bbatsov/rubocop/blob/master/config/default.yml)

## Inline Konfiguration
Manchmal macht es aber auch Sinn spezielle Regeln nur in bestimmten Bereichen einer Datei zu deaktivieren. Dass kann
über spezielle Kommentare in der Datei selbst erledigen:

{% highlight ruby %}
# rubocop:disable Metrics/LineLength, Style/StringLiterals
[...]
# rubocop:enable Metrics/LineLength, Style/StringLiterals
{% endhighlight %}

Die Zeile zwischen den beiden Kommentaren kann nun beliebig lang sein und es wird nicht mehr überprüft ob die richtigen
Anführungszeichen verwendet werden.

# Status Quo
Wie in der Einführung bereits erwähnt kann es passieren, dass [RuboCop][] erst spät in einem Projekt verwendet wird.
Plötzlich erkennt man, dass hunderte von Warnungen ausgegeben werden. Wenn man jetzt nur neue Probleme erkennen
möchte und die Alten später Stück für Stück in geplanten Blöcken beseitigen will, bringt [RuboCop][] eine schönes
Feature dafür mit:

    rubocop --auto-gen-config

Dieser Befehl erzeugt die Datei `.rubocop_todo.yml`. Darin werden alle gefundenen Problem als Ausschlüsse definiert.
Wenn man jetzt in der Datei `.rubocop.yml` folgenden Eintrag hinzufügt werden nur noch neue Probleme gefunden:

    inherit_from: .rubocop_todo.yml

# Schlusswort
Ich habe in diesem Artikel nur eine kleine Übersicht über die Konfigurationsmöglichkeiten aufgezeigt. Die
Projekt-Webseite enthält weitere Informationen, die sehr gut aufbereitet und umfangreich sind. Ich halte [RuboCop][] für
ein sehr wichtiges Tool, dass in jedem [Ruby][]- oder [Chef][]-Projekt verwendet werden sollte.

[Chef]: https://www.chef.io/ "Chef"
[Ruby]: https://www.ruby-lang.org/de/ "Ruby"
[Ruby Style Guide]: https://github.com/bbatsov/ruby-style-guide/blob/master/README.md "Ruby Style Guide"
[RuboCop]: https://github.com/bbatsov/rubocop "RuboCop"
[Foodcritic]: http://acrmp.github.io/foodcritic/ "Foodcritic"
[Jenkins]: https://jenkins-ci.org/ "Jenkins"
[CheckstyleFormatter]: https://github.com/eitoball/rubocop-checkstyle_formatter "CheckstyleFormatter"
[Chef-DK]: https://downloads.chef.io/chef-dk/ "Chef-DK"
[Violations plugin]: https://wiki.jenkins-ci.org/display/JENKINS/Violations "Violations plugin"
