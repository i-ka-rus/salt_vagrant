# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  os = "ubuntu/bionic64"
  network = "192.168.50"
  config.vagrant.plugins = ["vagrant-hostmanager"]

  config.hostmanager.enabled = true
  config.hostmanager.manage_host = true
  config.hostmanager.manage_guest = true
  config.hostmanager.ignore_private_ip = false
  config.hostmanager.include_offline = true

  config.vm.define :master ,primary: true do |master|
    master.vm.provider "virtualbox" do |vm|
      vm.memory = "2048"
      vm.cpus = 1
      vm.name = "salt"
    end
    master.vm.hostname = "salt"
    master.vm.box = "#{os}"
    master.vm.network "forwarded_port", guest: 22, host: 2222, id: "ssh"
    master.vm.network "private_network", ip: "#{network}.10"
    master.vm.synced_folder "saltstack/pillar/", "/srv/pillar/"
    master.vm.synced_folder "saltstack/salt/", "/srv/salt/"
    master.vm.provision "salt" do |salt|
      salt.install_type = "stable"
      salt.install_master = true
      salt.no_minion = false
      salt.verbose = true
      salt.colorize = true
      salt.bootstrap_options = "-P -c /tmp"
    end
  end

  [
    ["minion1",    "#{network}.11",    "512",    os ],
    ["minion2",    "#{network}.12",    "512",    os ],
  ].each do |vmname,ip,mem,os|
    config.vm.define "#{vmname}" do |minion|
      minion.vm.provider "virtualbox" do |vb|
          vb.memory = "#{mem}"
          vb.cpus = 1
          vb.name = "#{vmname}"
      end
      minion.vm.hostname = "#{vmname}"
      minion.vm.box = "#{os}"
      minion.vm.network "private_network", ip: "#{ip}"

      minion.vm.provision "salt" do |salt|
        salt.install_type = "stable"
        salt.verbose = true
        salt.colorize = true
        salt.bootstrap_options = "-P -c /tmp"
      end

      minion.trigger.after ["up", "provision", "reload"] do |tu|
        tu.name = "accept keys"
        tu.info = "accept minion key on master"
        tu.run = {inline: "vagrant ssh master -- sudo /usr/bin/salt-key -y -A"}
      end

      minion.trigger.after "destroy" do |tu|
        tu.name = "remove keys"
        tu.info = "remove minion key on master"
        tu.run = {inline: "vagrant ssh master -- sudo /usr/bin/salt-key -y -d #{vmname}"}
      end

    end
  end
end
