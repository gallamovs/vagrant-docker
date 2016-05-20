# -*- mode: ruby -*-
# vi: set ft=ruby :

hostname   = '.dev'
domain     = 'vagrant-docker'
aliases    = %w()

box        = 'vagrant-docker'
box_url    = 'vagrant-docker.box'

ip         = '192.168.33.19'
ram        = '1024'
cpu        = '2'

Vagrant.require_version ">= 1.8.1"

aliases_domain = []
if not aliases.empty?
  aliases.each do |i|
    aliases_domain.push(i + hostname)
  end
end

required_plugins = %w(vagrant-hostmanager vagrant-nfs_guest)

plugins_to_install = required_plugins.select { |plugin| not Vagrant.has_plugin? plugin }
if not plugins_to_install.empty?
  puts "Installing plugins: #{plugins_to_install.join(' ')}"
  if system "vagrant plugin install #{plugins_to_install.join(' ')}"
    exec "vagrant #{ARGV.join(' ')}"
  else
    abort "Installation of one or more plugins has failed. Aborting."
  end
end

Vagrant.configure("2") do |config|
  config.vm.box = box
  config.vm.box_url = box_url

  if not aliases_domain.empty?
    config.vm.box_url = box_url
  end

  config.vm.hostname = domain
  config.vm.network "private_network", ip: ip

  config.vm.provider "virtualbox" do |v|
      v.name = domain
      v.memory = ram
      v.cpus = cpu
  end

  config.vm.synced_folder ".", "/vagrant",
      disabled: false

  config.vm.synced_folder 'public/', '/var/www/public',
    type: 'nfs_guest',
    create: true,
    disabled: false

  config.hostmanager.enabled = true
  config.hostmanager.manage_host = true
  config.hostmanager.manage_guest = false
  config.hostmanager.ignore_private_ip = false
  config.hostmanager.include_offline = true

  config.vm.define domain + hostname do |node|
    node.vm.hostname = domain + hostname
    node.vm.network :private_network, ip: ip
    node.hostmanager.aliases = aliases_domain
  end

  config.ssh.username = "vagrant"
  config.ssh.password = "vagrant"
  config.ssh.insert_key = true

  # Provision
  config.vm.provision "file", source: "~/.gitconfig", destination: "~/.gitconfig"
  config.vm.provision "file", source: "~/.ssh/id_rsa", destination: "~/.ssh/id_rsa.pub"
  config.vm.provision "file", source: "~/.ssh/id_rsa", destination: "~/.ssh/id_rsa"
  # config.vm.provision "file", source: "~/.ssh/config", destination: "~/.ssh/config"
  # config.vm.provision "file", source: "~/.bash_aliases", destination: "~/.bash_aliases"

  config.vm.provision "docker" do |d|
    d.build_image "/vagrant/docker-images/nginx",
      args: "-t docker-images/nginx"

    d.build_image "/vagrant/docker-images/php",
      args: "-t docker-images/php"

    d.pull_images "mysql:5.5"

    d.run "mysql",
      args: "-v '/var/www/db:/var/lib/mysql' -p 3306:3306 -e MYSQL_ROOT_PASSWORD=secret -e MYSQL_DATABASE=mySchema"

    d.run "docker-images/php",
      args: "-v '/var/www/public:/var/www/public' --link mysql:mysql"

    d.run "docker-images/nginx",
      args: "-p 80:80 --volumes-from docker-images-php --link docker-images-php:phpfpmupstream"
  end
end