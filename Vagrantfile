require 'yaml'

# reference: https://www.debugcn.com/en/article/57056373.html
CONFIG = YAML.load_file(File.join(File.dirname(__FILE__), 'config.yml'))

Vagrant.configure("2") do |config|
  config.ssh.insert_key = false

  # bootstrap-server
  CONFIG['bootstrap'].each do |bootstrap|
    config.vm.define bootstrap['name'] do |cfg|
      cfg.vm.box = bootstrap['box']
      cfg.vm.hostname = bootstrap['hostname']
      cfg.vm.network "public_network", bridge: "enp7s0f0", ip: bootstrap['ip']
  
      cfg.vm.provider "virtualbox" do |v|
        v.memory = bootstrap['memory']
        v.cpus = bootstrap['cpu']
        v.name = bootstrap['name']
      end
      
      cfg.vm.provision  "shell", inline: <<-SCRIPT
        sed -i -e 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
        systemctl restart sshd
      SCRIPT

      # chrony configuration
      cfg.vm.provision "file", source: "chrony.conf", destination: "/tmp/chrony.conf"
      cfg.vm.provision  "shell", inline: <<-SCRIPT
        apt-get update
        apt-get install chrony -y
        cp /tmp/chrony.conf /etc/chrony.conf
        timedatectl set-timezone Australia/Sydney
        systemctl enable chrony
        systemctl restart chrony
        timedatectl set-ntp true
      SCRIPT
    end
  end

  # nodes
  CONFIG['nodes'].each do |node|
    config.vm.define node['name'] do |cfg|
      cfg.vm.box = node['box']
      cfg.vm.network "public_network", bridge: "enp7s0f0", ip: node['ip']
      cfg.vm.hostname = node['hostname']
      
      cfg.vm.provider "virtualbox" do |v|
        v.memory = node['memory']
        v.cpus = node['cpu']
        v.name = node['name']
      end

      cfg.vm.provision "shell", inline: <<-SCRIPT
        sed -i -e 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
        systemctl restart sshd
      SCRIPT

      # chrony configuration
      cfg.vm.provision "file", source: "chrony.conf", destination: "/tmp/chrony.conf"
      cfg.vm.provision "shell", inline: <<-SCRIPT
        apt-get update
        apt-get install chrony -y
        cp /tmp/chrony.conf /etc/chrony.conf
        timedatectl set-timezone Australia/Sydney
        systemctl enable chrony
        systemctl restart chrony
        timedatectl set-ntp true
      SCRIPT

      # common configuration for kubernetes 
      cfg.vm.provision "shell", inline: <<-SCRIPT
        sudo swapoff -a
        sudo sed -i '/swap/d' /etc/fstab
        sudo systemctl stop ufw
        sudo systemctl disable ufw
      SCRIPT
    end
  end
end