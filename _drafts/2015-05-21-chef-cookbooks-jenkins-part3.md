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

[Chef][] Cookbooks sind [Ruby][]-Code. Zwar verwendet man hauptsächliche eine DSL-Beschreibungssyntax, aber es bleibt
trotzdem [Ruby][]-Code der den gleichen Qualitätsanforderungen unterworfen werden sollte. Um die Einheitlichkeit von
Ruby-Code sicherzustellen wurde der [Ruby Style Guide][] geschrieben. Es soll dabei helfen, dass Ruby-Code möglichst
immer gleich aussieht, so dass ein Entwickler fremden Code besser versteht und pflegen oder erweitern kann.

![RuboCop](/assets/images/devops/rubocop.jpg)

[RuboCop][] ist ein Tool mit dem man geschriebenen Ruby-Code gegen einen Satz von Regeln prüfen lässt, der auf dem
[Ruby Style Guide][] basiert. Das bereits im [ersten Teil der Artikelreihe]
({% post_url /devops/2015-04-07-chef-cookbooks-jenkins-part1 %}) vorgestellte [Foodcritic][] ist eigentlich nur eine
Erweiterung von [RuboCop][] um ein paar Regeln die für die Besonderheiten der Chef Beschreibungs-DSL wichtig sind.
[RuboCop][] ist im [Chef-DK][] enthalten und kann somit nach der Installation direkt verwendet werden.

Es ist wichtig [RuboCop][] von Anfang an bei einem Projekt zu verwenden und die Einhaltung der Regeln dauerhaft zu
überprüfen. Somit macht es Sinn unseren [Jenkins][]-Job um [RuboCop][] zu erweitern. Wer RuboCop bei einem großen
Projekt erst spät einführt wird feststellen, dass der Code unzählige Abweichungen von den Regeln beinhaltet, deren
Beseitigung viel Aufwand verschlingt, u.U. spätere Analysen von Code-Änderungen erschwert (z.B. wenn man die Einrückung
global ändert und das Risiko einer Destabilisierung mit sich bringt. Ob in einem späten Stadium die Einführung noch
sinnvoll ist, muss individuell entschieden werden. Allerdings kann es u.U. sinnvoll sein nur bestimmte Regeln global zu
erzwingen und/oder den Status Quo fest zu halten und nur bei neuem Code auf die Einhaltung zu bestehen.

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
geschrieben. Dabei steht `.` für eine saubere Datei, die Großbuchstaben stehen für den Typ (`C`onvention, `W`arning,
`E`rror oder `F`atal).

Dann wird für jeden Verstoß die Datei, Zeile, Spalte, Typ und Regel ausgegeben. Gefolgt von einer Code-Ausgabe mit
Markierung. Wenn man sich ein wenig an diese Auflistung gewöhnt hat und nicht zu viele Verstöße gefunden werden, kann
man damit schon ganz gut arbeiten. Für den [Jenkins][] taugt diese Ausgabe aber nicht.

# Formatter
Es gibt verschiedene Ausgabeformate, die man mittels Formatter erzeugen lassen kann. Sehr schön finde ich den HTML
Formatter, mit dem man eine schöne Darstellung der Verstöße erhält:

    rubocop --format html -o rubocop.html

Oder den Offense Count Formatter der eine Übersicht erzeugt, der die Anzahl der Verstöße pro Regel auflistet:

    rubocop --format offenses

Beide sind im [Chef-DK][] enthalten und können sofort genutzt werden.

Aber für unsere automatisierte Überprüfung brauchen wir den [Rubocop::Junit::Formatter][].

# Rubocop::Junit::Formatter


# Schlusswort


[Chef]: https://www.chef.io/ "Chef"
[Ruby]: https://www.ruby-lang.org/de/ "Ruby"
[Ruby Style Guide]: https://github.com/bbatsov/ruby-style-guide/blob/master/README.md "Ruby Style Guide"
[RuboCop]: https://github.com/bbatsov/rubocop "RuboCop"
[Foodcritic]: http://acrmp.github.io/foodcritic/ "Foodcritic"
[Jenkins]: https://jenkins-ci.org/ "Jenkins"
[Rubocop::Junit::Formatter]: https://github.com/mikian/rubocop-junit-formatter "Rubocop::Junit::Formatter"
