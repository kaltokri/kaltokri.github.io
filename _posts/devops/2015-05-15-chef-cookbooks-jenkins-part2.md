---
layout: post
category : DevOps
title: Chef Cookbooks mit Jenkins prüfen, Teil2
tagline: "ChefSpec-Auswertung hinzufügen"
tags : [Chef, Ruby, Provisioning, Jenkins]
---
{% include JB/setup %}
<div class="toc"></div>

# Vorwort

Im [ersten Teil der Artikelreihe]({% post_url /devops/2015-04-07-chef-cookbooks-jenkins-part1 %}) haben wir einen
[Jenkins][] Job erstellt, der mittels [Knife][] die [Ruby][] Syntax prüft. Wenn er einen Fehler findet wird der Job
abgebrochen und erhält den Status fehlgeschlagen.

Im zweiten Build-Schritt definierten wir die Prüfung auf Schwachstellen im Code mittels [Foodcritic][]. Je nach
übergebenen Parametern und Konfiguration vom [Warnings Plugin][] bricht der Job entweder ab oder läuft weiter und führt
auch die weiteren Analysen durch. Letzteres würde ich immer empfehlen um möglichst viele Probleme auf einmal
aufzudecken.

Zuletzt haben wir einen Buildschritt für [ChefSpec][] eingebaut. Schlägt ein (oder mehrere) Test(s) fehl wird der Job
abgebrochen und ist fehlgeschlagen.

# ChefSpec-Auswertung hinzufügen
Den letzten Punkt möchte ich ändern. Wenn ein Unit-Test fehlschlägt möchte ich die nachfolgenden Analysen trotzdem
laufen lassen und möglichst viele Problem im Build aufzudecken. Das hat den Nachteil, dass u.U. Folgefehler
angezeigt werden. Andererseits erspart man sich oft mehrfach fehlschlagende Builds weil sich der Entwickler von einer
Fehlerbehebung zur nächsten schichtweise durchkämpft.

Um das zu erreichen fügen wir folgenden Parameter dem Aufruf hinzu:

    --failure-exit-code 0

Um eine schöne Auswertung der Test zu erhalten wollen wir das Ergebnis in einer Datei im JUnit-Format abspeichern. Dafür
gibt es ein [gem][] mit dem Namen [rspec_junit_formatter][]. Dieses [gem][] ist bereits im [Chef-DK][] enthalten.

<div class="note info">
<h5>Versionskonflikt im Chef-DK 0.4.0</h5>
Bei der Ausführung von RSpec mit dem rspec_junit_formatter kommt immer folgender Fehler:
<pre>Unable to activate rspec-2.14.1, because rspec-core-3.1.7 conflicts with rspec-core (~> 2.14.0) (Gem::ConflictError)</pre>
Glücklicherweise lässt sich das recht einfach beheben. Folgender Befehl lädt die neueste Version herunter:
<pre>gem install rspec_junit_formatter</pre>
ACHTUNG: Das gem wird aber im Home-Verzeichnis des Benutzers abgelegt! Das Problem ist also nicht für alle Benutzer
zentral behoben. Jeder Benutzer muss den Befehl bei sich ausführen. D.h. man muss es auch bei dem Benutzer ausführen,
unter dem der Jenkins Dienst läuft.
</div>

Jetzt können wir den Aufruf von [RSpec][] wie folgt ändern:

    rspec --format RspecJunitFormatter --out chefspec-results.xml --failure-exit-code 0 ./cookbooks

Als letztes installieren wir das [Violations Plugin][] für den [Jenkins][]. Danach können wir den neuen
Post-Build-Schritt `Veröffentlich JUnit-Testergebnisse` hinzufügen. Im Textfeld `Testberichte in XML-Format` geben wir
`chefspec-results.xml` ein und speichern.

Ab dem nächsten Build erhalten wir eine detaillierte Auswertung der Tests. Und ab dem zweiten Build auch eine
Trend-Grafik:

![ChefSpec Trendauswertung](/assets/images/devops/chefspec-trend.jpg)

# Schlusswort
Ich werde dazu über gehen immer nur kleine Artikel zu schreiben und online zu stellen, da größere Artikel zu lange
brauchen um geschrieben zu werden und daher offline bleiben. Also werde ich Continous Integration auch bei meinem Blog
anwenden und möglichst kleine abgeschlossene Inkremente (Artikel) veröffentlichen.

[Chef-DK]: https://downloads.chef.io/chef-dk/ "Chef-DK"
[Foodcritic]: http://www.foodcritic.io/ "Foodcritic"
[Knife]: https://docs.chef.io/knife.html "Knife"
[ChefSpec]: https://docs.chef.io/chefspec.html "ChefSpec"
[Ruby]: https://www.ruby-lang.org/de/ "Ruby"
[Jenkins]: https://jenkins-ci.org/ "Jenkins"
[RSpec]: http://rspec.info/ "RSpec"
[Warnings Plugin]: https://wiki.jenkins-ci.org/display/JENKINS/Warnings+Plugin "Warnings Plugin"
[Violations Plugin]: https://wiki.jenkins-ci.org/display/JENKINS/Violations "Violations Plugin"
[gem]: https://rubygems.org/ "gem"
[rspec_junit_formatter]: https://rubygems.org/gems/rspec_junit_formatter "rspec_junit_formatter"
