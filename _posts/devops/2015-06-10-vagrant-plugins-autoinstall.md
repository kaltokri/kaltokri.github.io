---
layout: post
category : DevOps
title: Vagrant plug-in autoinstall
tagline: "Vagrant plug-in's automatisch installieren lassen"
tags : [Vagrant]
---
{% include JB/setup %}
<div class="toc"></div>

# Vorwort

Für [Vagrant][] gibt es einige nützliche Plug-In's. Wenn man diese verwenden möchte muss aber jeder Nutzer der VM sie
manuell per Kommandozeile installieren. Fehlt ein Plug-In wird eine Fehlermeldung ausgegeben. Gerade wenn man eine
[Vagrant][]-VM erstellt, die von Leuten mit wenig [Vagrant][]-Erfahrung benutzt wird stellt dies ein Problem.

Zunächst habe ich meine [Vagrant][]-VM's so aufgebaut, dass überprüft wurde ob das benötigte Plug-In installiert ist.
Falls nicht wurde eine aussagekräftige Fehlermeldung ausgegeben die direkt den benötigten Befehl enthielt um das
Plug-In per Copy & Paste zu installieren.

Dann bin ich per Zufall über einen Code-Schipsel im [Vagrant][] [Issue 5199][] gestolpert. Das brachte mich auf die Idee
über eine For-each-Schleife direkt alle benötigten Plug-In's installieren zu lassen. Das Ergebnis sieht wie folgt aus:

{% highlight ruby linenos %}
# Install all required plug in's for Vagrant
%w(triggers berkshelf cachier omnibus hostsupdater).each do |plugin_name |
  next if Vagrant .has_plugin?('vagrant-' + plugin_name)
  # Attempt to install ourself.
  # Bail out on failure so we don't get stuck in an infinite loop.
  system('vagrant plugin install vagrant-' + plugin_name) || exit!

  # Relaunch Vagrant so the new plugin(s) are detected.
  # Exit with the same status code.
  exit system('vagrant', *ARGV)
end
{% endhighlight %}

Diesen Code fügt man vor dem `Vagrant.configure` Block ein. Es funktioniert einwandfrei.

# Schlusswort
Ich verwende diese wenigen Zeilen jetzt in allen meinen VM's und passe nur noch die Liste der Plug-In's an. Das ist sehr
komfortabel und erleichtert die Verteilung der VM's.

[Vagrant]: https://www.vagrantup.com/ "Vagrant"
[Issue 5199]: https://github.com/mitchellh/vagrant/issues/5199#issuecomment-95805705 "Issue 5199"
