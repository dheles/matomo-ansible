# -*- mode: ruby -*-
# vi: set ft=ruby :

domain          = "test"
setup_complete  = false

# NOTE: currently using the same OS for all boxen
OS="centos" # "debian" || "centos"

# NOTE: fix for atlas.hashicorp.com 404ing all boxes
# https://github.com/hashicorp/vagrant/issues/9442#issuecomment-363080565
# may also need to edit the metadata_url file for existing boxes,
# replacing atlas.hashicorp.com with vagrantcloud.com
# but being sure to add no eol character...
# updating vagrant may also help or fix
Vagrant::DEFAULT_SERVER_URL.replace('https://vagrantcloud.com')

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure(2) do |config|
  if OS=="debian"
    config.vm.box = "debian/stretch64"
    config.vm.box_version = "9.3.0"
  elsif OS=="centos"
    config.vm.box = "centos/7"
    config.vm.box_version = "1801.02" # 7.4.1708
  else
    puts "you must set the OS variable to a valid value before continuing"
    exit
  end

  {
    'matomo'   => '10.8.21.40'
  }.each do |short_name, ip|
    config.vm.define short_name do |host|
      host.vm.network 'private_network', ip: ip
      host.vm.hostname = "#{short_name}.#{domain}"
      # presumes installation of https://github.com/cogitatio/vagrant-hostsupdater on host
      host.hostsupdater.aliases = ["#{short_name}"]
      # avoiding "Authentication failure" issue
      host.ssh.insert_key = false
      host.vm.synced_folder ".", "/vagrant", disabled: true

      host.vm.provider "virtualbox" do |vb|
        vb.name = "#{short_name}.#{domain}"
        vb.memory = 256
        vb.linked_clone = true

        if short_name == "matomo"
          # port forwarding http:
          # host.vm.network "forwarded_port", guest: 80, host: 8080, auto_correct: true
        end
      end

      if short_name == "matomo" # last in the list
        setup_complete = true
      end

      if setup_complete
        host.vm.provision "ansible" do |ansible|
          ansible.galaxy_role_file = "requirements.yml"
          ansible.inventory_path = "inventory/vagrant"
          ansible.playbook = "setup.yml"
          ansible.limit = "all,localhost"
          ansible.verbose = "v"
        end
      end
    end
  end
end
