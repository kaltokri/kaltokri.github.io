---
layout: post
category : DevOps
title: Eclipse und Bash-Skripte
tagline: "Bash-Skripte mit Eclipse bearbeiten"
tags : [Provisioning, Eclipse, Bash]
---
{% include JB/setup %}
<div class="toc"></div>

# Vorwort

Wenn man mit [Vagrant][] und [Chef][] automatisiert Software testet wird man meist Linux-Systeme als Grundlage
verwenden. Das liegt daran, dass keine Lizenzkosten anfallen (egal wie viele VM's laufen), die Basis-Installation viel
weniger Speicherplatz auf der Festplatte benötigt und das System sicherer ist.

Daher kommt es vor, dass man [BASH][]-Skripte unter Windows bearbeiten muss. [Notepad++][] leistet da sehr gute Dienste,
aber im Artikel [Chef Cookbooks mit Eclipse bearbeiten]({% post_url /devops/2015-05-16-chef-eclipse %}) habe ich
bereits erklärt, wie man [Eclipse][] erweitert um [Ruby][]-Code mit Syntax-Highlighting bearbeiten zu können.
Da möchte ich natürlich auch [BASH][]-Skripte in [Eclipse][] bearbeiten und nicht das Tool wechseln müssen.

# ShellEd

Die Erweiterung [ShellEd][] ist die Lösung. Hier ein Beispiel wie ein einfaches "Hello world!"-Skript in [Eclipse][]
aussieht:

![Package Explorer](/assets/images/devops/2015-05-30-eclipse-bash.jpg)

Um die Erweiterung zu installieren sind folgende Schritte nötig:

* Man lädt die Datei `net.sourceforge.shelled-site-\<version\>.zip` von der [ShellEd][] Projektwebseite herunter.
* Die Datei wird in einen beliebigen Ordner entpackt.
* In [Eclipse][] öffnen wir das Menü `Help`.
* Dort wählen wir `Install New Software...` aus.
* Wir klicken auf der rechten Seite den Knopf `Add...` an und im daraufhin geöffneten Dialog `Local...`.
* Dann navigieren wir zum entpackten Ordner und wählen ihn aus.
* Als `Name` können wir beispielsweise `ShellEd` verwenden und klicken `Ok` an.
* In der Auswahl aktivieren wir `Shell Script` und `Next >`.
* Jetzt nochmal `Next >`, die Lizenzbestimmungen lesen, akzeptieren und `Finish` drücken.

Die Software wird jetzt herunter geladen und installiert. [Eclipse][] möchte einen Neustart und danach können wir
[ShellEd][] nutzen. In den `Preferences` findet man einen neuen Knotenpunkt `Shell Skript` in dem verschiedene
Einstellungen vorgenommen werden können.

# Schlusswort
Leider funktioniert der Hover-Funktion unter Windows nicht, da 'man' nur unter Linux verfügbar ist. Trotzdem finde ich
das Plug-in sehr sinnvoll.

[Vagrant]: https://www.vagrantup.com/ "Vagrant"
[Chef]: https://www.chef.io/chef/ "Chef"
[BASH]: http://www.linuxdoc.org/HOWTO/Bash-Prog-Intro-HOWTO.html "BASH"
[Notepad++]: https://notepad-plus-plus.org/ "Notepad++"
[Eclipse]: https://eclipse.org/ "Eclipse"
[Ruby]: https://www.ruby-lang.org/de/ "Ruby"
[ShellEd]: http://sourceforge.net/projects/shelled/ "ShellEd"
