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
  config.vm.define "awx_container" do |awx_container|
    awx_container.vm.hostname = "awx-container.local"
    awx_container.vm.network :private_network, ip: "192.168.6.68"

    awx_container.vm.provision :ansible_local do |ansible|
      ansible.install_mode = "pip"
      ansible.provisioning_path = "/vagrant/prebuild"
      ansible.playbook = "prebuild.yml"
      ansible.galaxy_role_file = "requirements.yml"
      ansible.galaxy_roles_path = "roles"
      ansible.become = true
    end
  end

end
