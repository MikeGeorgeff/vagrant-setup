# -*- mode: ruby -*-
# vi: set ft=ruby :
require "yaml"

# Load the config file
CONF = YAML.load(File.open("ansible/config.yml", File::RDONLY).read)

# Check which os the host machine is using
# Used to launch ansible
def which(cmd)
    exts = ENV['PATHEXT'] ? ENV['PATHEXT'].split(';') : ['']
    ENV['PATH'].split(File::PATH_SEPARATOR).each do |path|
        exts.each { |ext|
            exe = File.join(path, "#{cmd}#{ext}")
            return exe if File.executable? exe
        }
    end
    return nil
end

Vagrant.configure(2) do |config|

  # Load and configure the vagrant box
  config.vm.box = CONF['base_box']
  config.vm.hostname = CONF['hostname']

  config.vm.network :private_network, ip: CONF['ip_address']
  config.vm.network :forwarded_port, guest: 80, host: 8000  
  config.vm.network :forwarded_port, guest: 3306, host: 33060
  config.ssh.forward_agent = true

  config.vm.synced_folder "www", "/home/vagrant/www", create: true

  # Configure virtual box
  config.vm.provider "virtualbox" do |v|
    v.name = CONF['hostname']
    v.customize ["modifyvm", :id, "--memory", CONF['memory']]
    v.customize ["modifyvm", :id, "--cpus", CONF['cpus']]
    v.customize ["modifyvm", :id, "--natdnsproxy1", "on"]
    v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    v.customize ["modifyvm", :id, "--ostype", "Ubuntu_64"]
  end

  # Provision the box with ansible
  if which('ansible-playbook')
    config.vm.provision "ansible" do |ansible|
      ansible.playbook = "ansible/playbook.yml"
      ansible.inventory_path = "ansible/inventories/dev"
      ansible.limit = "all"
    end
  else
    config.vm.provision :shell, path: "ansible/install.sh", args: [CONF['hostname']]
  end      

end
