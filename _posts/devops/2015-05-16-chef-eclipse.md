---
layout: post
category : DevOps
title: Chef Cookbooks mit Eclipse bearbeiten
tagline: "Eclipse fit für Ruby machen"
tags : [Chef, Ruby, Provisioning, Eclipse]
---
{% include JB/setup %}
<div class="toc"></div>

# Vorwort

Ich arbeite viel mit [Notepad++][] und bin mit dem Editor sehr zufrieden. Allerdings kann man dort leider nicht pro
Programmiersprache unterschiedliche Abstände für die Einrückung definieren. Also habe ich Google gefragt ob es möglich
ist [Eclipse][] so zu erweitern, dass es [Ruby][]-Code mit Highlighting usw. unterstützt. Und siehe da, es gibt ein
Plugin dafür.

Ich möchte in diesem Artikel erklären wie ich die Umgebung eingerichtet habe.
Das dient für mich selbst als Nachschlagewerk damit ich es nachlesen kann, wenn ich es nochmal machen muss.
Aber vielleicht hilft es ja auch anderen die diesen Artikel lesen.

# Aptana Studio

Zur Unterstützung von Ruby installieren wir [Aptana Studio][]. Im Download-Bereich wählen wir
**Eclipse Plug-in Version** und klicken auf **Download**. Auf der folgenden Seite erhalten wir eine Installationsleitung
der wir folgen:

* Wir starten Eclipse.
* Dann öffnen wir das Menü `Help`.
* Dort wählen wir `Install New Software...` aus.
* Im Textfeld `Work with` fügen wir die folgende URL ein und drücken `Enter`:
```
    http://download.aptana.com/studio3/plugin/install
```

* Wir aktivieren `Aptana Studio 3` und klicken auf `Next >`.
* Dann nochmal auf `Next >` klicken, die Lizenz lesen und bestätigen.
* Ein Klick auf `Finish` und der Download und die Installation läuft.
* Es kommt eine Meldung die uns auf ein Problem hinweist. Wir bestätigen mit `Ok` und probieren es einfach nochmal.
* Beim zweiten Versuch klappt es dann und wir starten Eclipse neu.

# Import der Cookbooks

Als erstes legen wir uns einen Workspace an in dem die Einstellungen gespeichert und später unsere Cookbooks importiert
werden. Man kann entweder jedes einzelne Cookbook importieren oder den übergeordneten Ordner. Ich bevorzuge letzteres,
da ich die `.project` Datei nicht in die Repositories der Cookbooks aufnehmen möchte. Beispiel:

* Im Menü `File` wählen wir `Import` aus.
* Wir klappen `General` auf und wählen `Existing Folder as New Project`, dann `Next >`.
* Im Textfeld `Select Folder` geben wir den Pfad an. Bei mir `C:\chef\cookbooks`.
* Unter `Project Type` aktivieren wir nur `Ruby`, dann `Finish`.

Im `Package Explorer` sollte das jetzt ungefähr so aussehen:

![Package Explorer](/assets/images/devops/2015-05-16-chef-eclipse-01.jpg)

Die Dateien mit der Endung `.rb` werden mit einem Ruby-Icon angezeigt und wenn man sie öffnet sieht man, dass Syntax
Highlighting funktioniert.

# Weitere Dateien als Ruby-Dateien markieren

Aber manche Dateien werden nicht als Ruby-Dateien erkannt. So z.B. `Vagrantfile`s. Um das zu ändern müssen wir folgendes
machen:

* Wir öffnen `Window` > `Preferences` > `General` > `Content Types`.
* Dann `Text` aufklappen und in der Liste `Ruby` anklicken.
* Über den Knopf `Add` können wir weitere Dateinamen angeben die dann als Ruby-Dateien erkannt werden.

# Zeilenbegrenzung anzeigen

Als [Notepad++][] Benutzer habe ich mich daran gewöhnt die Zeilenbegrenzung durch eine Linie angezeigt zu bekommen.
Um das auch in [Eclipse][] zu aktivieren ist folgendes zu tun:

* Diesmal in `Window` > `Preferences` > `General` > `Editors` > `Text Editors` > `Show Print Margin` aktivieren.

# Versteckte Dateien anzeigen

Per Default blendet [Eclipse][] Dateien aus die mit einem Punkt beginnen. Dann können wir aber unsere `.kitchen.yml`
Dateien nicht sehen und bearbeiten. Daher schalten wir das ab:

* Im `Package Explorer` das kleine Dreieck anwählen, dass nach unten zeigt.
* `Filters...` öffnen.
* `.* resources` deaktivieren.

# Command line here

Jetzt wollen wir noch einrichten, dass man im ausgewählten Cookbook schnell eine Kommandozeile öffnen kann. Das können
wir auf Unterschiedliche Weise realisieren. Ganz praktisch finde ich folgende Plugins:
[tarlog](https://code.google.com/p/tarlog-plugins/)

# Run configurations hinzufügen

Es ist aber noch lästig immer wieder zwischen Eclipse und der Eingabeaufforderung hin und her wechseln zu müssen.
Wäre es nicht schön bestimmte Kommandos direkt aus Eclipse laufen zu lassen? Das geht auch. Hier ein Beispiel
mit [Vagrant][]:

* Wir öffnen `Run` > `External Tools` > `External Tools Configurations...`.
* Jetzt klicken wir auf `New launch configuration` und tragen auf der Karte `Main` folgende Werte ein:
  * Name: `Vagrant up`
  * Location: `C:\HashiCorp\Vagrant\bin\vagrant.exe`
  * Working Directory: `${container_loc}`
  * Arguments: `up`

Wenn wir nur in einem Cookbook eine Datei anklicken und `Vagrant_up` unter `Run` > `Externalt Tools` aufrufen, wird
[Vagrant][] in diesem Ordner mit dem Parameter `up` gestartet. Falls ein `Vagrantfile` vorhanden ist, wird die VM
gestartet bzw. erstellt. Allerdings gehen nicht alle Befehle. `vagrant destroy` klappt nur wenn man den Parameter
`--force` anhängt. Falls man dies nicht tut kommt eine Warnung Vagrant bräuchte ein TTY. `vagrant ssh` will gar
nicht. Aber:

Eine Run-Configuration kann man auch mit `C:\Windows\System32\cmd.exe` machen, so dass man eine interaktive
Kommandozeile im aktuellen Ordner (als Eclipse-View) erhält. Und für alles was in einer Eclipse-View nicht läuft haben
wir ja auch noch das die tarlog-plugins.

# Schlusswort

So eingerichtet kann man schon recht komfortabel mit [Vagrant][], [Chef][] und [Eclipse][] arbeiten. [Aptana Studio][]
unterstützt auch noch [Rake][]. Damit kann man auch noch eine Menge machen. Aber das schaue ich mir später an. \*g\*

[Notepad++]: https://notepad-plus-plus.org/ "Notepad++"
[Eclipse]: https://eclipse.org/ "Eclipse"
[Ruby]: https://www.ruby-lang.org/de/ "Ruby"
[Aptana Studio]: http://www.aptana.com/products/studio3/download.html "Aptana Studio"
[Vagrant]: https://www.vagrantup.com/ "Vagrant"
[Chef]: https://www.chef.io/chef/ "Chef"
[Rake]: http://rake.rubyforge.org/ "Rake"
