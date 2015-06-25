---
layout: post
category : DevOps
title: chef-solo complete
tagline: "chef-solo: Umgebung richtig einrichten"
tags : [Chef. chef-solo, Provisioning ]
---
{% include JB/setup %}
<div class="toc"></div>

# Vorwort

Wenn man in [Vagrant][] dem [chef-solo][]-Provisioner nutzt konfiguriert man alle Einstellungen im Vagrantfile.
[Vagrant][] übernimmt dann die komplette Einrichtung der Umgebung. Aber was passiert, wenn man einen physikalischen
Server mittels [chef-solo][] einrichten möchte? Dann muss man sich die Informationen mühselig zusammen suchen, denn
sie sind in der Chef Dokumentation auf verschiedenen Seiten verstreut.

Ich möchte anhand eines kleinen Beispiels die Einrichtung der Umgebung Schritt für Schritt erläutern. Dazu verwende ich
zwar eine [Vagrant][]-Box, aber ohne den [chef-solo][]-Provisioner zu nutzen. Wir werden einen Jenkins installieren und
dazu das Community [cookbook jenkins][] verwenden.

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

Alternativ kann man auch das Chef-DK installieren. Das ist zwar deutlich größer, liefert aber auch direkt Tools wie
Berkshelf mit. Die folgenden Befehle sollte man als `root` oder mittels `sudo` ausführen. Wir erstellen die
Default-Ablage für Cookbooks:

    mkdir -p /var/chef/cookbooks
    cd /var/chef/cookbooks

Danach laden wir das [cookbook jenkins][] von [Supermarket][] mittels [Knife][] herunter:

    knife cookbook site download jenkins
    tar -xvzf jenkins-2.3.1.tar.gz


# Schlusswort


[Vagrant]: https://www.vagrantup.com/
[chef-solo]: http://docs.vagrantup.com/v2/provisioning/chef_solo.html
[PuTTY]: http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html
[cookbook jenkins]: https://github.com/opscode-cookbooks/jenkins
[Supermarket]: https://supermarket.chef.io/
[Knife]: https://docs.chef.io/knife.html