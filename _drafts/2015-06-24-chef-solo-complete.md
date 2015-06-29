---
layout: post
category : DevOps
title: chef-solo complete
tagline: "chef-solo: Umgebung richtig einrichten"
tags : [Chef, chef-solo, Provisioning ]
---
{% include JB/setup %}
<div class="toc"></div>

# Vorwort

Wenn man in [Vagrant][] den [chef-solo][]-Provisioner verwendet konfiguriert man alle Einstellungen im Vagrantfile.
[Vagrant][] übernimmt dann die komplette Einrichtung der Umgebung. Aber was passiert, wenn man einen physikalischen
Server mittels [chef-solo][] einrichten möchte? Dann muss man sich die Informationen mühselig zusammen suchen, denn
sie sind in der [Chef][] Dokumentation auf verschiedenen Seiten verstreut.

Ich möchte anhand eines kleinen Beispiels die Einrichtung der Umgebung Schritt für Schritt erläutern. Dazu verwende ich
zwar eine [Vagrant][]-Box, aber ohne den [chef-solo][]-Provisioner zu nutzen. Wir werden einen Jenkins installieren und
dazu das Community [cookbook jenkins][] verwenden.

# Testumgebung erstellen

Zuerst erstellen wir ein Vagrantfile für unsere Testumgebung:

{% highlight ruby linenos %}
VAGRANTFILE_API_VERSION = '2'

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = 'chef/centos-6.5'

  config.vm.hostname = 'chef-solo-demo'
  config.vm.network 'forwarded_port', guest: 8080, host: 8080

  # Use VBoxManage to customize the VM.
  config.vm.provider 'virtualbox' do |vb|
    vb.name = 'chef-solo-demo'
    vb.memory = 1024
  end
end
{% endhighlight %}

Nach dem `vagrant up` können wir uns mittels `vagrant ssh` oder einer [PuTTY][]-Session am Rechner anmelden.
Zunächst installieren wir den chef-client. Dazu nutzen wir das Script der chef Webseite:

    curl -L https://www.chef.io/chef/install.sh | sudo bash

Danach sollte folgendes Kommando die Hilfe von chef-solo ausgeben:

    chef-solo -h

<div class="note info">
Alternativ kann man auch das <a href="https://downloads.chef.io/chef-dk">Chef-DK</a> installieren. Das ist zwar
deutlich größer, liefert aber auch direkt Tools wie Berkshelf mit.
</div>

# Cookbook und seine Abhängigkeiten herunter laden

Die folgenden Befehle sollte man alle als `root` oder mittels `sudo` ausführen. Wir erstellen die Default-Ablage für
Cookbooks:

    mkdir -p /var/chef/cookbooks
    cd /var/chef/cookbooks

Danach laden wir das [cookbook jenkins][] von [Supermarket][] mittels [Knife][] herunter und entpacken es:

    knife cookbook site download jenkins
    tar -xvzf jenkins-2.3.1.tar.gz

Auf der Detailseite vom Cookbook jenkins im [Supermarket][] finden wir im Reiter [Dependancies][] die Angabe welche
anderen Community Cookbooks wir noch herunter laden müssen, weil das jenkins Cookbook diese verwendet:

* yum ~> 3.0
* runit ~> 1.5
* apt ~> 2.0

Folgende Befehle müssen ausgeführt werden (die Dateinamen können wegen der Version abweichen):

    cd /var/chef/cookbooks
    knife cookbook site download yum
    tar -xvzf yum-3.6.1.tar.gz

    knife cookbook site download runit
    tar -xvzf runit-1.7.2.tar.gz

    knife cookbook site download apt
    tar -xvzf apt-2.7.0.tar.gz


# Konfiguration

Als wir [Knife][] verwendet haben und die Cookbooks heruter zu laden wurde folgende Warnung angezeigt:

    "WARNING: No knife configuration file found"

Damit diese Warung verschwindet legen wir die Datei [knife.rb][] an:

    mkdir /etc/chef
    touch /etc/chef/knife.rb

Theoretisch reicht eine leere Datei, aber folgender Inhalt läßt sich später um eigene Pfade erweitern und kann daher
auch genommen werden:

{% highlight ruby linenos %}
cookbook_path [
               '/var/chef/cookbooks',
               '/var/chef/site-cookbooks'
]
{% endhighlight %}

Ab jetzt zeigt [Knife][] die Warnung nicht mehr an und wir können die Datei [knife.rb][] später mit anderen
Konfigurationen erweitern. Theoretisch könnten wir mittels `chef-solo -o jenkins` schon ein Provisioning durchführen.
Aber wir wollen das Cookbook mit ein paar Konfigurationen versorgen. Außerdem führt das Kommando zur Ausgabe von
Warnungen weil wir die Run-List überschreiben. In der Run-List stehen alle Cookbooks, die auf den aktuellen Rechner
angewendet werden sollen. Diese Run-List und einige Attribute wollen wir nun festlegen.

Dazu erstellen wir die Konfigurationsdatei [solo.rb][] für [chef-solo][]:

    touch /etc/chef/solo.rb

Jetzt füllen wir sie mit folgendem Inhalt:

{% highlight ruby linenos %}
cookbook_path [
               '/var/chef/cookbooks',
               '/var/chef/site-cookbooks'
              ]
role_path '/var/chef/roles'
json_attribs '/etc/chef/node.json'
solo true
{% endhighlight %}

Die beiden wichtigen Zeilen sind:

1. die Pfad-Angabe wo die Rollen gesucht werden sollen (Zeile 5) und
2. die JSON-Datei in der wir konfigurieren welche Rolle(n) unser Rechner verwenden soll.

# Schlusswort


[Vagrant]: https://www.vagrantup.com/
[chef-solo]: http://docs.vagrantup.com/v2/provisioning/chef_solo.html
[Chef]: https://www.chef.io/chef/
[PuTTY]: http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html
[Chef-DK]: https://downloads.chef.io/chef-dk/
[cookbook jenkins]: https://github.com/opscode-cookbooks/jenkins
[Supermarket]: https://supermarket.chef.io/
[Knife]: https://docs.chef.io/knife.html
[solo.rb]: https://docs.chef.io/config_rb_solo.html
[Dependancies]: https://supermarket.chef.io/cookbooks/jenkins#dependencies
[knife.rb]: https://docs.chef.io/config_rb_knife.html
