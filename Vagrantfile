# -*- mode: ruby -*-
# vi: set ft=ruby :
require 'yaml'

settings = YAML.load_file 'settings.yml'

# Set the amount of RAM you want to allocate per VM. The default
# (2G) is the minimum, set this to higher if you have RAM to spare
ram = settings['ram']

Vagrant.configure("2") do |config|
  # All Vagrant configuration is done here. The most common configuration
  # options are documented and commented below. For a complete reference,
  # please see the online documentation at vagrantup.com.

  # Every Vagrant virtual environment requires a box to build off of.
  config.vm.box = settings['box'] 

  # The url from where the 'config.vm.box' box will be fetched if it
  # doesn't already exist on the user's system.
  config.vm.box_url = settings['box_url']

  # Enable agent forwarding so Ansible can connect to other machines
  # using the vagrant user.
  config.ssh.forward_agent = true

  # rsync works with any provider.
  # But by default, we don't need a synced folder, so it's disabled here
  # to save the time normally spent installing rsync to the target machine.
  config.vm.synced_folder './', '/vagrant', type: 'rsync', disabled: true

  # Make sure the nodes can resolve each other's hostnames
  config.vm.provision :hosts do |provisioner|
    provisioner.add_host '192.168.122.111', ['alice']
    provisioner.add_host '192.168.122.112', ['bob']
    provisioner.add_host '192.168.122.113', ['charlie']
    provisioner.add_host '192.168.122.114', ['daisy']
    provisioner.add_host '192.168.122.115', ['eric']
    provisioner.add_host '192.168.122.116', ['frank']
  end

  # VirtualBox specific settings.
  config.vm.provider :virtualbox do |vb|
    # Boot with a GUI so you can see the screen. (Default is headless)
    vb.gui = true

    # Set the amount of memory specified above
    vb.memory = ram
  end

  # Libvirt specific settings
  config.vm.provider :libvirt do |domain|
    domain.memory = ram
    domain.nested = false
    domain.storage :file, :size => '20G'
    domain.storage :file, :size => '20G'
  end

  config.vm.define "daisy" do |machine|
    machine.vm.hostname = "daisy"
    machine.vm.network :private_network, ip: "192.168.122.114"
    machine.vm.network :private_network, ip: "192.168.133.114"
    machine.vm.network :private_network, ip: "192.168.144.114"
  end

  config.vm.define "eric" do |machine|
    machine.vm.hostname = "eric"
    machine.vm.network :private_network, ip: "192.168.122.115"
    machine.vm.network :private_network, ip: "192.168.133.115"
    machine.vm.network :private_network, ip: "192.168.144.115"
  end

  config.vm.define "frank" do |machine|
    machine.vm.hostname = "frank"
    machine.vm.network :private_network, ip: "192.168.122.116"
    machine.vm.network :private_network, ip: "192.168.133.116"
    machine.vm.network :private_network, ip: "192.168.144.116"
  end

  config.vm.define "alice" do |machine|
    machine.vm.hostname = "alice"
    machine.vm.network :private_network, ip: "192.168.122.111"
    machine.vm.network :private_network, ip: "192.168.133.111"
    machine.vm.network :private_network, ip: "192.168.144.111"

    machine.vm.provision "ansible" do |ansible|
      ansible.playbook = "ansible/site.yml"
      ansible.limit = "all"
      ansible.sudo = true
      ansible.extra_vars = {
        ansible_ssh_user: 'vagrant',
        ceph_release: settings['ceph_release']
      }
      ansible.groups = {
        "deploy" => ["alice"],
        "osds" => ["daisy", "eric", "frank"],
        "mons" => ["daisy", "eric", "frank"],
        "radosgws" => ["daisy"],
        "cephclients" => ["alice", "bob"],
        "s3clients" => ["alice", "bob"],
      }
    end
  end
end
