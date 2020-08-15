# -*- mode: ruby -*-
# vi: set ft=ruby :

###############################################################################
#
# DEVOPS
# by Mariusz Wąsik (Bronto)
#
# CENTOS
# Docker SWARM Claster
# 
###############################################################################
ENV['VAGRANT_NO_PARALLEL'] = 'yes'

Vagrant.configure("2") do |config|
  
  # SETUP ///////////////////////////////////////////////////////////////////////
  VM_DOMAIN        = "torek.local"
  VM_BOX           = "centos/7" 

  # Array(hash) of master nodes
  SWARM_MASTER_NODES = {'sm01' => '192.168.56.5'}                     
  
  # Array(hash) of master nodes
  SWARM_WORKER_NODES = {'sw01' => '192.168.56.6'}
            
  SWARM_MASTER_MEMORY = 4096
  SWARM_MASTER_CPU    = 2

  SWARM_WORKER_MEMORY = 2048
  SWARM_WORKER_CPU    = 2
  #///////////////////////////////////////////////////////////////////////////////

  ##############################################################################                   
  # SWARM MASTERS PREPARE SERVERS
  ##############################################################################
  SWARM_MASTER_NODES.each do |node_name, node_ip|
    
    config.vm.define node_name.to_sym do |masternode|

      VM_FQDN = node_name + "." + "#{VM_DOMAIN}"

      masternode.vm.box = VM_BOX
      masternode.vm.hostname = VM_FQDN
      masternode.vm.hostname = VM_FQDN
      masternode.vm.network "private_network", ip: node_ip
      masternode.vm.network :forwarded_port, guest: 22, host: 2201, id: "ssh", auto_correct: true
      #masternode.hostsupdater.remove_on_suspend = false
      
      # Replace hostname with FQDN
      masternode.vm.provision "shell", inline: "hostnamectl set-hostname #{VM_FQDN}"
    
      masternode.vm.provider "virtualbox" do |v|
        v.name  = node_name
        v.memory = SWARM_MASTER_MEMORY
        v.cpus   = SWARM_MASTER_CPU
        # Prevent VirtualBox from interfering with host audio stack
        v.customize ["modifyvm", :id, "--audio", "none"]
      end

      # docker swarm master init 
      masternode.vm.provision "shell", path: "scripts/swarm_init.sh"  
    
    end
  end 

  #############################################################################                    
  # SWARM WORKERS PREPARE SERVERS
  #############################################################################
  last_vm = SWARM_WORKER_NODES.keys.last
  swarm_master = SWARM_MASTER_NODES.keys.first+'.'+VM_DOMAIN

  SWARM_WORKER_NODES.each do |node_name, node_ip|
    
    config.vm.define node_name.to_sym do |workernode|

      VM_FQDN = node_name + "." + "#{VM_DOMAIN}"

      workernode.vm.box = VM_BOX
      workernode.vm.hostname = VM_FQDN
      workernode.vm.network "private_network", ip: node_ip
      workernode.vm.network :forwarded_port, guest: 22, host: 2201, id: "ssh", auto_correct: true
      #workernode.hostsupdater.remove_on_suspend = false

      # Replace hostname with FQDN
      workernode.vm.provision "shell", inline: "hostnamectl set-hostname #{VM_FQDN}"
    
      workernode.vm.provider "virtualbox" do |v|
        v.name  = node_name
        v.memory = SWARM_WORKER_MEMORY
        v.cpus   = SWARM_WORKER_CPU
        # Prevent VirtualBox from interfering with host audio stack
        v.customize ["modifyvm", :id, "--audio", "none"]
      end

      #docker swarm join workers to swarm master 
      workernode.vm.provision "shell", path: "scripts/swarm_join.sh", :args => [swarm_master] 
      
      # Run triger when last VM was deployed
      workernode.trigger.after :up do |trigger|
        trigger.only_on = last_vm
        trigger.info = "That was a last created node"
        trigger.run_remote = {path: "scripts/swarmpit.sh", :args => [swarm_master] }
      end

    end
  end 

  #############################################################################                    
  # FOR ALL NODES
  #############################################################################

  # Make /etc/hosts 
  $repairHost = <<-'SCRIPT'
    cat /dev/null > /etc/hosts
    echo "127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4" >> /etc/hosts
    echo "::1         localhost localhost.localdomain localhost6 localhost6.localdomain6" >> /etc/hosts
  SCRIPT
  config.vm.provision "shell", inline: $repairHost

  SWARM_MASTER_NODES.each do |node_name, node_ip|
    config.vm.provision :shell do |s|
      MASTER_FQDN = node_name + "." + "#{VM_DOMAIN}"
      s.inline = 'echo -e "$1\t$2\t$3" | sudo tee -a /etc/hosts'
      s.args = "#{node_ip} #{MASTER_FQDN} #{node_name}" 
    end
  end
  SWARM_WORKER_NODES.each do |node_name, node_ip|
    config.vm.provision :shell do |s|
      WORKER_FQDN = node_name + "." + "#{VM_DOMAIN}"
      s.inline = 'echo -e "$1\t$2\t$3" | sudo tee -a /etc/hosts'
      s.args = "#{node_ip} #{WORKER_FQDN} #{node_name}" 
    end
  end

  # Prepere OS - tools,environment  
  config.vm.provision "shell", path: "scripts/bootstrap.sh"
  # Install docker
  config.vm.provision "shell", path: "scripts/docker.sh"

end
