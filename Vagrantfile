# -*- mode: ruby -*-
# vi: set ft=ruby :

# Install required vagrant plugin
require File.dirname(__FILE__)+"/dependency_manager"
check_plugins ["vagrant-hostmanager", "vagrant-vbguest"]

class FixGuestAdditions < VagrantVbguest::Installers::RedHat
    def dependencies
        packages = super
        # If there's no "kernel-devel" package matching the running kernel in the
        # default repositories, then the base box we're using doesn't match the
        # latest CentOS release anymore and we have to look for it in the archives...
        if communicate.test('test -f /etc/centos-release && ! yum -q info kernel-devel-`uname -r` &>/dev/null')
            env.ui.warn("[#{vm.name}] Looking for the CentOS 'kernel-devel' package in the release archives...")
            packages.sub!('kernel-devel-`uname -r`', 'http://mirror.centos.org/centos' \
                                                     '/`grep -Po \'\b\d+\.[\d.]+\b\' /etc/centos-release`' \
                                                     '/{os,updates}/`arch`/Packages/kernel-devel-`uname -r`.rpm')
        end
        packages
    end
end

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

# Require YAML module
require 'yaml'

# Configure scripts path variable
scriptsPath = File.dirname(__FILE__) + '/scripts'

# Read YAML file with box details
servers = YAML.load_file('hosts.yaml')

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  servers.each do |servers|

    # Hostmanager configuration
    config.hostmanager.enabled = true
    config.hostmanager.manage_host = true
    config.hostmanager.manage_guest = true
    config.hostmanager.ignore_private_ip = false
    config.hostmanager.include_offline = true

    config.vm.define servers["name"], autostart: servers["autostart"] ||= false, primary: servers["primary"] ||= false do |srv|
      srv.vm.box = servers["box"] ||= 'centos/7'
      srv.vm.hostname = servers["hostname"]
      srv.hostmanager.aliases = servers["aliases"]
      srv.vm.network "private_network", ip: servers["ip"]
      srv.ssh.shell = "bash -c 'BASH_ENV=/etc/profile exec bash'"
      srv.vm.synced_folder '.', '/vagrant', disabled: true

      if servers["enable_sync_folder"] == true
        srv.vbguest.no_install = false
        srv.vbguest.installer = FixGuestAdditions
        srv.vm.synced_folder "./data_dir", "/projects", type: "virtualbox", mount_options: ["dmode=777","fmode=777"], disabled: false
      else
        srv.vbguest.no_install = true
      end

      srv.vm.provider "virtualbox" do |vb|
        vb.name = servers["name"]
        vb.gui = servers["enable_gui"] ||= false
        vb.customize ['modifyvm', :id, '--memory', servers["memory"] ||= 1024]
        vb.customize ['modifyvm', :id, '--cpus', servers["cpus"] ||= 1]
        vb.customize ['modifyvm', :id, '--cpuexecutioncap', '80']
        vb.customize ['modifyvm', :id, '--natdnsproxy1', 'on']
        vb.customize ['modifyvm', :id, '--natdnshostresolver1', 'on']
      end

      #config.vm.provision "bootstrap", type: "shell", :path => "scripts/bootstrap.sh"

    end

  end

end
