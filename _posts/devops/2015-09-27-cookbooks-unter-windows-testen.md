---
layout: post
category : DevOps
title: Cookbooks unter Windows testen
tagline: "Mittels manuell erstellter VirtualBox VM"
tags : [Chef, chef-solo, Provisioning, Windows ]
---
{% include JB/setup %}
<div class="toc"></div>

# Vorwort
Wenn man Cookbooks für [Chef][] schreibt die sowohl unter Linux als auch unter
Windows laufen sollen, dann muss man diese natürlich auch unter beiden
Betriebssystemen testen. Für Linux stellt das weniger ein Problem dar, weil
Tools wie z.B. [vagrant][] oder [KitchenCI][] schnelles testen ermöglichen.

Warum sollte das bei Windows anders sein? Grundsätzlich liegt es an zwei
Dingen:

1. Microsoft's EULA verbietet es Base-Boxen zu erstellen und zu verteilen.
   Selbst bei Testversionen ohne Lizenzkey.
2. Während Linux Base-Boxen ca. 500 MB groß sind es bei Windows 7 etwa 20GB.
   Windows 10 ist mit 10GB deutlich schmaler, aber im Vergleich immer noch
   riesig.

Daher findet man kaum Windows Base-Boxen in der [VagrantCloud][] und von
denen die man findet sollte man die Finger lassen. Sie stellen sowohl
rechtlich als auch Sicherheitstechnisch ein großes Risiko dar, weil man
ihrer Quelle nicht trauen kann.

# KitchenCI und Windows

Früher war [KitchenCI][] nicht in der Lage Windows-VM's zu steuern. Das hat sich
seit der Version 1.4.0 endlich geändert. Im Artikel ["Test Kitchen Windows Test
Flight with Vagrant"][] wird erklärt welche Plug-In's nötig sind und auch wie man
Windows-Base-Boxen mittels [Packer][] selber erstellt.

Den Artikel habe ich erst gefunden als ich dabei war diesen Post zu schreiben.
Ich werde ihn mir in kürze genauer vornehmen und meine Erfahrungen in ein neues
Posting einfließen lassen.

Wer seine Cookbooks prüfen möchte ohne sich mit [Packer][] und der
Base-Box-Erstellung auseinander setzen zu müssen kann sich statt dessen einfach
manuell eine VM in [VirtualBox][] aufsetzen.

# Manuelle VM erstellen
"Warum soll ich mir die Arbeit machen? Ich kann die Cookbooks doch auf meinem
Entwicklersystem testen." ... sagte der leichtsinnige Entwickler kurz bevor ein
Fehler in seinem Cookbook sein komplettes Benutzerprofil löschte.

Man sollte Cookbooks immer in einer VM testen, damit bei Fehlern der Schaden
gering bleibt und zusätzlich die Testumgebung möglichst nah an die
Produktivumgebung heran reicht.

Folgende Schrite muss man durchführen um eine Test-VM zu erstellen:

* Download des ISO-Images der gewünschten Windows Version. Zum Beispiel [Windows
  10 von Winfuture][].
* Download und Installation von [VirtualBox][].
* Erstellung einer VM in [VirtualBox][] für Windows mit Default-Werten, d.h.
  32GB dynamisch wachsende Festplatte und 2GB Speicher.
* Dann bindet man das ISO-Image an das virtuelle DVD-Rom-Laufwerk und
  installiert Windows. Die Eingabe der Lizenzschlüssels überspringt man einfach.
* Nach der Installation sollte man die Gasterweiterungen installieren, damit die
  Oberfläche nicht so träge reagiert.
* Jetzt fährt man das System wieder runter und erstellt einen Sicherungspunkt,
  so dass man diesen Zustand immer wieder herstellen kann.
* Als nächstes installiert man [Chef][] und testet die Kochbücher.
  Natürlich kann man auch mehrere Sicherungspunkte erstellen, damit man nicht
  immer wieder [Chef][] installieren muss.
* Über einen gemeinsamen Ordner kann man die erstellten Cookbooks in der VM
  testen.

# Schlusswort
Mit der kompletten Einrichtung ist man unter einer Stunde fertig. Die
Einarbeitung in [Packer][] ist auch kein Hexenwerk, dauert aber garantiert
länger.

Zusätzlich muss man aber noch organisatorische und sicherheitstechnische Aspekte
bedenken. Man sollte mit der internen Administration dieses Vorgehen abklären.
Denn auch VM's sollten mittels Virenscanner geschützt werden.

In einem späteren Post kümmere ich mich dann mal im [Packer][] mit Windows und
[KitchenCI][].

[Chef]: https://www.chef.io/chef/
[Vagrant]: https://www.vagrantup.com/
[KitchenCI]: http://kitchen.ci/
[VagrantCloud]: https://vagrantcloud.com/
["Test Kitchen Windows Test Flight with Vagrant"]: http://kitchen.ci/blog/test-kitchen-windows-test-flight-with-vagrant/
[Packer]: https://www.packer.io/
[VirtualBox]: https://www.virtualbox.org/
[Windows 10 von Winfuture]: http://winfuture.de/downloadvorschalt,3226.html
