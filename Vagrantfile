# Defines our Vagrant environment
#
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  # create ansible node
  config.vm.define :"master.local" do |master_config|
    master_config.vm.box = "centos/7"

    master_config.vm.hostname = "master.local"
    master_config.vm.define "master.local"
    master_config.vm.network :private_network, ip: "192.168.1.10", hostsupdater: "skip"
    master_config.vm.provider "virtualbox" do |vb|
      vb.memory = "8192"
      vb.name = "master.local"

      vb.customize ["modifyvm", :id, "--nicpromisc2", "allow-all"]
    end

    master_config.vm.synced_folder ".", "/vagrant", type: "nfs"

    if Vagrant.has_plugin?("vagrant-hosts")
      master_config.vm.provision :hosts do |provisioner|
        provisioner.sync_hosts = true
        provisioner.add_localhost_hostnames = false

        provisioner.add_host '192.168.1.10',  ['master.local', 'master']
        provisioner.imports = ['global', 'virtualbox']
        provisioner.exports = {
          'virtualbox' => [
            ['@vagrant_private_networks', ['@vagrant_hostnames']],
          ],
        }
      end
    end

    if Vagrant.has_plugin?("vagrant-vbguest")
      master_config.vbguest.auto_update = true
      master_config.vbguest.no_remote   = true
      master_config.vbguest.no_install  = false
    end

    if Vagrant.has_plugin?("vagrant-proxyconf")
      master_config.proxy.http     = ""
      master_config.proxy.https    = ""
      master_config.proxy.no_proxy = ""
    end

    master_config.vm.provision "shell" do |s|
      ssh_pub_key = File.readlines("../ssh/id_rsa.pub").first.strip
      s.inline = <<-SHELL
        echo #{ssh_pub_key} >> /home/vagrant/.ssh/authorized_keys
      SHELL
      s.privileged = false
    end
  end

  # slave
  (0..1).each do |i|
    config.vm.define "slave#{i}.local" do |worker_config|
      worker_config.vm.box = "centos/7"

      worker_config.vm.hostname = "slave#{i}.local"
      worker_config.vm.define "slave#{i}.local"
      worker_config.vm.network :private_network, ip: "192.168.1.10#{i}", hostsupdater: "skip"
      worker_config.vm.provider "virtualbox" do |vb|
        vb.memory = "4096"
	      vb.name = "slave#{i}.local"

        vb.customize ["modifyvm", :id, "--nicpromisc2", "allow-all"]
      end

      if Vagrant.has_plugin?("vagrant-hosts")
        worker_config.vm.provision :hosts do |provisioner|
          provisioner.sync_hosts = true
          provisioner.add_localhost_hostnames = false

          provisioner.add_host "192.168.1.10#{i}", ["slave#{i}.local", "slave#{i}"]
          provisioner.imports = ['global', 'virtualbox']
          provisioner.exports = {
            'virtualbox' => [
              ['@vagrant_private_networks', ['@vagrant_hostnames']],
            ],
          }
        end
      end

      if Vagrant.has_plugin?("vagrant-vbguest")
        worker_config.vbguest.auto_update = true
        worker_config.vbguest.no_remote   = true
        worker_config.vbguest.no_install  = false
      end

      if Vagrant.has_plugin?("vagrant-proxyconf")
        worker_config.proxy.http     = ""
        worker_config.proxy.https    = ""
        worker_config.proxy.no_proxy = ""
      end

      worker_config.vm.provision "shell" do |s|
        ssh_pub_key = File.readlines("../ssh/id_rsa.pub").first.strip
        s.inline = <<-SHELL
          echo #{ssh_pub_key} >> /home/vagrant/.ssh/authorized_keys
        SHELL
        s.privileged = false
      end
    end
  end
end
