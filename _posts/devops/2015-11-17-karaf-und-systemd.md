---
layout: post
category : DevOps
title: Karaf und systemd
tagline: "Karaf in CentOS 7 als Systemdienst einrichten"
tags : [CentOS7, Karaf]
---
{% include JB/setup %}
<div class="toc"></div>

# Vorwort
Vor einiger Zeit habe ich ein [Chef Cookbook][] zur Provisionierung von
[Karaf][] unter [CentOS][] 6.5 erstellt. Nun wurde es Zeit Karaf auch unter
CentOS 7 zu betreiben.

# systemd ersetzt SysV
Mit CentOS 7 wurde das SysV-Init-System durch [systemd][] ersetzt.
systemd ist zwar abwärtskompatibel zu SysV, aber da es Probleme im
Zusammenspiel mit Chef gab wollte ich Karaf auf systemd umstellen.

Nach der ersten Recherche fand ich das Jira [Issue 3858][] in dem es um die
Erweiterung des Karaf Service Wrappers geht. Dort wurde der Wrapper so
erweitert, dass er eine systemd Konfigurationsdatei (Unit) im Unterverzeichnis
`$KARAF_HOME\bin` erstellt.
Um die Unit zu erhalten muss zunächst das Feature installiert werden:

    feature:install service-wrapper

Karaf zieht das Feature aus dem Internet und installiert es.
Der folgende Befehl sorgt für die Erstellung aller Dateien, die nötig sind um
Karaf als Systemdienst zu betreiben:

    wrapper:install

Leider funktionierte der folgende (in der Karaf Dokumentation
erwähnte) Befehl zur Aktivierung des Dienstes nicht:

    systemctl enable /opt/apache-karaf-3.0.5/bin/karaf.service

Später fand ich heraus, dass in einer neueren Version des systemd dieses
Kommando funktioniert. Aber die in meiner CentOS 7 Installation enthaltenen
Version war dafür leider zu alt.

Also musste ich mein Wissen bezüglich systemd erweitern. In einem
Artikel der Red Hat Enterprise Linux Dokumentation wird erklärt wie man
[Konfigurationsdateien für den systemd][] erstellt.

Laut Dokumentation sollen Unit's die vom Administrator erstellt werden unter
`/etc/systemd/system/` abgelegt werden. Also erstellte ich einen symbolischen
Link von `$KARAF_HOME/bin/karaf.service` nach
`/etc/systemd/system/karaf.service`. Damit war es zwar mir möglich den Dienst per
`systemctl start karaf.service` zu starten, aber die Aktivierung des
Dienststarts im Bootvorgang von CentOS mit unten aufgeführtem Befehl führte zu
einer Fehlermeldung:

    systemctl enable karaf.service

Hier die irreführende Fehlermeldung:

    Failed to issue method call: No such file or directory

Alle meine Versuche die Ursache des Problem zu analysieren scheiterten, bis ich
schließlich auf [Bug 955379][] gestoßen bin. Dort wird beschrieben,
dass symbolische Links nur eingeschränkt unter `/etc/systemd/system/`
funktionieren. Da sich dort bereits symbolische Links befanden bin ich nicht
auf die Idee gekommen, dass nur bestimmte Ziele zulässig sind.

# Die Lösung
... ist mal wieder einfach, wenn man sie erst kennt. Anstatt die Datei zu
verlinken muss man sie einfach kopieren. Und schon klappt es auch mit
systemd.

# Schlusswort
Es hat wieder viel Zeit und Nerven gekostet auf die richtige Spur zu kommen.
Und wie so oft liegt es an mangelnder Dokumentation. Gewürzt wird
das Ganze mit einer missverständlichen Fehlermeldung und fertig ist ein
typischer DevOps-Arbeitstag. :wink:

[Chef Cookbook]: https://docs.chef.io/cookbooks.html
[Karaf]: http://karaf.apache.org/
[CentOS]: https://www.centos.org/
[systemd]: https://de.wikipedia.org/wiki/Systemd
[Issue 3858]: https://issues.apache.org/jira/browse/KARAF-3858
[Konfigurationsdateien für den systemd]: https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/System_Administrators_Guide/sect-Managing_Services_with_systemd-Unit_Files.html
[Bug 955379]: https://bugzilla.redhat.com/show_bug.cgi?id=955379
