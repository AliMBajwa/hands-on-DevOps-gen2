# -*- mode: ruby -*-
# vi: set ft=ruby :

# Copyright (C) 2021 Michael Joseph Walsh - All Rights Reserved
# You may use, distribute and modify this code under the
# terms of the the license.
#
# You should have received a copy of the license with
# this file. If not, please email <github.com@nemonik.com>

$VERBOSE = nil

## Set the values in the dotenv (.env) file at the root of this project
##
def get_value_from_dotenv(var)
  line=open('./.env') { |f| f.lines.find { |line| line.include?(var) } }
  line=line.chomp
  return line.split("=").last.split(' ').first.tr('"','')
end

## This class uses VirtualBox and therefor expects Windows HyperV to be disabled.
##
if Vagrant::Util::Platform.windows? and Vagrant::Util::Platform.windows_hyperv_enabled?
  puts "Windows HyperV is expected to be disabled."
  exit(false)
end

uninstall_plugins = %w( vagrant-cachier vagrant-alpine )
required_plugins = %w( vagrant-vbguest vagrant-share vagrant-timezone vagrant-disksize vagrant-reload ) 

## Uninstall the following plugins
##
plugin_uninstalled = false
uninstall_plugins.each do |plugin|
  if Vagrant.has_plugin?(plugin)
    system "vagrant plugin uninstall #{plugin}"
    plugin_uninstalled = true
  end
end

## Require the following plugins
##
plugin_installed = false
required_plugins.each do |plugin|
  unless Vagrant.has_plugin?(plugin)
    system "vagrant plugin install #{plugin}"
    plugin_installed = true
  end
end

system "vagrant plugin update"

## if plugins were installed, restart
##
if plugin_installed || plugin_uninstalled
  puts "restarting"
  exec "vagrant #{ARGV.join' '}"
end

Vagrant.configure("2") do |config|
  
  config.vm.provider :virtualbox do |virtualbox|
    virtualbox.name = "Hands-on DevOps class Gen 2"
     virtualbox.gui = false

    # disable audio
    virtualbox.customize ['modifyvm', :id, '--audio', 'none']

    virtualbox.customize ['modifyvm', :id, '--nic1', 'nat']
    virtualbox.customize ['modifyvm', :id, '--cableconnected1', 'on']
    virtualbox.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    virtualbox.customize ["modifyvm", :id, "--natdnsproxy1", "on"]

    virtualbox.customize [ "guestproperty", "set", :id, "/VirtualBox/GuestAdd/VBoxService/--timesync-interval", 10000 ]
    virtualbox.customize [ "guestproperty", "set", :id, "/VirtualBox/GuestAdd/VBoxService/--timesync-min-adjust", 100 ]
    virtualbox.customize [ "guestproperty", "set", :id, "/VirtualBox/GuestAdd/VBoxService/--timesync-set-on-restore", 1 ]
    virtualbox.customize [ "guestproperty", "set", :id, "/VirtualBox/GuestAdd/VBoxService/--timesync-set-start", 1 ]
    virtualbox.customize [ "guestproperty", "set", :id, "/VirtualBox/GuestAdd/VBoxService/--timesync-set-threshold", 1000 ]

    virtualbox.memory = get_value_from_dotenv("vagrant_memory").to_i
    virtualbox.cpus = get_value_from_dotenv("vagrant_cpus").to_i

  end

  config.vm.synced_folder ".", "/vagrant"

  config.vm.box = get_value_from_dotenv("vagrant_box")

  config.vm.box_check_update = true

  config.vm.network "private_network", ip: "192.168.0.10"

  config.vm.synced_folder ".", "/vagrant_data"

  config.disksize.size = get_value_from_dotenv("vagrant_disksize")

  config.vm.hostname = "k3d-cluster"

  config.vm.provision "shell", inline: <<-SCRIPT
  sudo pacman -Syu --noconfirm 
  SCRIPT

  config.vm.provision :reload

  config.vm.provision "shell", inline: <<-SCRIPT
  sudo pacman -S --noconfirm docker
  sudo systemctl enable docker 
  sudo systemctl start docker 
  sudo usermod -aG docker vagrant
  SCRIPT

  config.vm.provision :reload

end
