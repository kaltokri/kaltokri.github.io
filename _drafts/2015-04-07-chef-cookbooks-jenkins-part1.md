---
layout: post
category : DevOps
title: Chef Cookbooks mit Jenkins prüfen, Teil1
tagline: "Automatisierte Qualitätssicherung im Chef Umfeld"
tags : [Chef, Ruby, Provisioning, Jenkins]
---
{% include JB/setup %}
<div class="toc"></div>

# Vorwort

Ich habe begonnen mittels [Chef][] Umgebungen für die Entwicklung und Qualitätsprüfung zu provisionieren.
Wenn man mit [Chef][] beginnt sind die ersten [Cookbooks][] recht übersichtlich, doch schon bald entstehen komplexe
[Cookbooks][] die u.U. auch nativen [Ruby][]-Code enthalten können. Schnell wird klar, dass hier Code entsteht (es heißt
ja auch Infrastructure as Code). Bei Java-Projekten ist klar, dass ihr Code mittels [CheckStyle][] und [FindBugs][]
analysiert wird. Unit- und Integrationstests sind ebenfalls selbstverständlich.

Warum sollte man [Ruby][]-Code (und [Chef][] [Cookbooks][] sind nun mal [Ruby][]) anders behandeln? Die Community hat
das natürlich längst erkannt und stellt Tools bereit um alle Aspekte abzudecken:

* Syntax-Check per [Knife][] (Tool von [Chef][])
* [RuboCop][] und [Foodcritic][] um statische Code-Analysen durchzuführen (vergleichbar mit [CheckStyle][] und [FindBugs][])
* [ChefSpec][] für Unit-Tests
* [Kitchen][] zur Erstellung von Integrationstests
* Analyse der Testabdeckung mittels [ChefSpec][] und ggf. [SimpleCov][]

Damit die Qualität sicher gestellt werden kann sollten [Cookbooks][] auf einem CI-Buildserver regelmäßig geprüft
werden. Am besten nach jedem Commit damit der Entwickler frühzeitig eine Rückmeldung erhält. In dieser Beitragsreihe
werde ich erklären, wie ich dies mittels [Jenkins][] realisiert habe.

# Vorbereitung
Bei dieser Artikelreihe gehe ich davon aus, dass der [Jenkins][] bereits installiert ist. Zusätzlich habe ich das [Chef-DK][]
(Development Kit) installiert. Es installiert eine vorbereitete Umgebung mit einer ganzen Menge an benötigten Tools.
Außerdem ist eine komplett vorbereitete [Ruby][]-Umgebung enthalten. Die wichtigsten Gems sind in kompatiblen Versionen
vorinstalliert. Alternativ kann man auch selber ein [Ruby][] installieren und alle benötigen Gems nachinstallieren,
aber die Arbeit sparen wir uns.

Ich nutze [Jenkins][] auf einem Windows-Rechner. Um die Tools auf der Kommandozeile nutzen zu können muss
die Umgebungsvariable `PATH` erweitert werden. Wer den Standardpfad bei der Installation verwendet hat muss folgendes
hinzufügen: `C:\opscode\chefdk\bin;C:\opscode\chefdk\embedded\bin;`

Danach sollten die beiden folgenden Befehle funktionieren und die jeweiligen Versionen ausgeben:

    knife -v
    ruby -v

Zunächst stelle ich die Tools vor und zeige welche Befehle und Konfigurationen nötig sind. Mit diesem Wissen kann man
die Tools bereit in der Entwicklungsphase einsetzen.
Später erstellen wir dann im [Jenkins][] einen FreeStyle-Job um alle Schritte automatisch auszuführen.

# Syntax-Check mittels Knife
[Knife][] ist ein Tool das hauptsächlich im Client-/Server-Umfeld von [Chef][] verwendet wird. Mit ihm verwaltet man die
Inhalte, die man auf den Server übertragen möchte. Aber auch wenn man nur [chef-solo][] einsetzt kann [Knife][] eine
Hilfe sein. Es ist in der Lage die Syntax der Chef [Cookbooks][] auf Fehler zu überprüfen. Somit ist es natürlich der
erste Schritt in unsere Überprüfung.

Per Default-Einstellung verwendet [Knife][] immer den folgenden Pfad um [Cookbooks][] zu erstellen oder zu überprüfen:
`C:\chef\cookbooks`. D.h. wenn wir folgenden Befehl ausführen werden alle [Cookbooks][] innerhalb des genannten Ordners
überprüft:

    knife cookbook test -a

Wenn wir unsere [Cookbooks][] an anderer Stelle ablegen müssen wir die Konfiguration von [Knife][] anpassen. Dazu erstellen wir
erstmal die benötigte Ordnerstruktur:

    CD /D %USERPROFILE%
    MKDIR .chef

Im Ordner `.chef` erstellen wir eine Textdatei mit folgendem Inhalt:

    cookbook_path [ '.', './cookbooks' ]
    cache_type 'BasicFile'
    cache_options(:path => "#{ENV['HOME']}/.chef/checksums")

Jetzt sucht [Knife][] die [Cookbooks][] im aktuellen Verzeichnis oder im Unterordner `cookbooks`.

# Code-Analyse mit Foodcritic
<img src="/assets/images/foodcritic.png" alt="Foodcritic Logo" style="box-shadow: none; background-color: #FFF;
margin: 0px 10px 10px 0px; border: 1px solid black;" align="left" width="150">
[Foodcritic][] ist ein Tool zur statischen Code-Analyse von [Chef][] [Cookbooks][]. Es ist vergleichbar mit einer
Kombination aus [CheckStyle][] und [FindBugs][] oder einem [Lint][]-Tool aus der C++ Welt. Es wird mit einer Reihe an
vordefinierten Regeln ausgeliefert. Es bietet aber auch die Möglichkeit die Regeln zu erweitern oder nicht gewünschte
Regeln zu deaktivieren. Jeder Entwickler sollte vor dem pushen der Änderungen mittels [Foodcritic][] prüfen das er keine
neuen Warnungen generiert.

[Foodcritic][] ist bereits im [Chef-DK][] enthalten und bereit für den ersten Einsatz. Mit dem folgenden Befehl starten
wir eine Analyse im Unterordner `cookbooks`:

    foodcritic ./cookbooks

Eine detaillierte Beschreibung der Warnungen mit Beispielen zur Korrektur ist auf der Webseite aufgeführt.

# Unit-Tests mit ChefSpec
[ChefSpec][] basiert auf [RSpec][], einem Unittest-Framework für [Ruby][]. [ChefSpec][] erweitert [RSpec][] um die
Funktionen um Tests für [Chef][] zu schreiben. In diesem Artikel möchte ich keine umfassende Beschreibung über die
Verwendung von [ChefSpec][] einfügen. Aber ein kurzes Beispiel soll die Verwendung aufzeigen und als spätere Beispiel
dienen.

Angenommen wir haben ein simples [Cookbook][], dass nur [Perl][] installieren soll. Dazu soll der bevorzugte
Paketmanager verwendet werden. Dann sieht unsere `./recipes/default.rb` z.B. so aus:

    package 'perl'

Um einen Unittest dafür zu erstellen legen wir den Ordner `./spec` an. Für jedes `Recipe` legen wir eine passende
`*_spec.rb` Datei an. Für unser Beispiel sieht die Datei `./spec/default_spec.rb` wie folgt aus:

{% highlight ruby linenos %}
require 'chefspec'

describe 'example::default' do
    let(:chef_run) { ChefSpec::SoloRunner.converge(described_recipe) }
    
    it 'installs perl' do
        expect(chef_run).to install_package('perl')
    end
end
{% endhighlight %}

Mit folgendem Befehl veranlassen wir [ChefSpec][] für alle [Cookbooks][]s im Unterordner `cookbooks` die Unittests
auszuführen:

    rspec cookbooks

Da [ChefSpec][] nur eine Erweiterung von [RSpec][] ist, gibt es (noch) keinen eigenen Befehl um die Tests auszuführen.
Das Ergebnis sollte wie folgt aussehen:

    .

    Finished in 0.5616 seconds (files took 10.72 seconds to load)
    1 examples, 0 failures

Für jede Datei die ausgeführt wird schreibt [ChefSpec][] ein Zeichen. Ein `.` bei Erfolg und ein `F` bei
fehlgeschlagenen Tests. Danach folgt eine Zeile mit der benötigten Zeit und am Schluss eine Zusammenfassung.

Wenn Fehler aufgetreten sind werden sie ausführlich beschrieben. Sollte in der letzten Zeile `0 examples, 0 failures`
stehen, dann hat [RSpec][] keine Tests zum ausführen gefunden. Das passiert schonmal, wenn man den falschen Pfad
erwischt.

<div class="note info">
<h5>Fehlermeldung "uninitialized constant ChefSpec::SoloRunner"</h5>
Bei der Ausführung von RSpec mit dem Chef-DK Version 0.3.2 kam immer folgender Fehler:
<pre>uninitialized constant ChefSpec::SoloRunner</pre>
Google brachte keine brauchbaren Lösungsvorschläge, aber nachdem ich auf Chef-DK Version 0.4.0 aktualisiert habe ist
das Problem verschwunden und ChefSpec funktioniert einwandfrei.
</div>

# Erstellung Jenkins-Job


# Wie geht es weiter
Ich möchte den ersten Artikel an der Stelle beenden damit er nicht zu lang wird. Im zweiten Blog-Artikel dieser Reihe befassen
wir uns mit der Erstellung des Buildjob im [Jenkins][].

[Chef]: https://www.chef.io/ "Chef"
[Chef-DK]: https://downloads.chef.io/chef-dk/ "Chef-DK"
[chef-solo]: https://docs.chef.io/chef_solo.html "chef-solo"
[CheckStyle]: http://checkstyle.sourceforge.net/ "checkstyle"
[FindBugs]: http://findbugs.sourceforge.net/ "FindBugs"
[Cookbooks]: http://docs.chef.io/cookbooks.html "About Cookbooks"
[RuboCop]: https://github.com/bbatsov/rubocop "RuboCop"
[Foodcritic]: http://www.foodcritic.io/ "Foodcritic"
[Knife]: https://docs.chef.io/knife.html "Knife"
[ChefSpec]: https://docs.chef.io/chefspec.html "ChefSpec"
[Kitchen]: https://docs.chef.io/kitchen.html "Kitchen"
[Ruby]: https://www.ruby-lang.org/de/ "Ruby"
[SimpleCov]: https://github.com/colszowka/simplecov "SimpleCov"
[Jenkins]: https://jenkins-ci.org/ "Jenkins"
[Lint]: http://de.wikipedia.org/wiki/Lint_%28Programmierwerkzeug%29 "Lint"
[RSpec]: http://rspec.info/ "RSpec"
[Perl]: https://www.perl.org/ "Perl"
[SimpleCov]: https://github.com/colszowka/simplecov "SimpleCov"
[Chef recipe code coverage]: https://sethvargo.com/chef-recipe-code-coverage/ "Blog: Chef recipe code coverage"
