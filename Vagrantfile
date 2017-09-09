# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "geerlingguy/centos7"
  config.ssh.insert_key = false

  config.vm.provider :virtualbox do |v|
    v.memory = 1024
    v.cpus = 2
    v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    v.customize ["modifyvm", :id, "--ioapic", "on"]
  end

  # AWX build VM.
  config.vm.define "awx" do |awx|
    awx.vm.hostname = "awx-container.local"
    awx.vm.network :private_network, ip: "192.168.6.68"

    awx.vm.provision :ansible do |ansible|
      ansible.playbook = "prebuild/prebuild.yml"
      ansible.galaxy_role_file = "prebuild/requirements.yml"
      ansible.galaxy_roles_path = "prebuild/roles"
      ansible.become = true
    end
  end

end
