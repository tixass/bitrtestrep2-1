# -*- mode: ruby -*-
# vi: set ft=ruby :

# Префикс для LAN сети
BRIDGE_NET="192.168.1."
# Префикс для Internal сети
INTERNAL_NET="192.168.15."

DOMAIN="agima.rus"
# Массив настроек для битриксовых машин
serversbx=[
  {
    :hostname => "BX1." + DOMAIN,
	:ip => BRIDGE_NET + "150",
    :ip_int => INTERNAL_NET + "2",
    :ram => 1024,
	:frwd_host => 8080
 },
  {    :hostname => "BX2." + DOMAIN,
	:ip => BRIDGE_NET + "151",
   :ip_int => INTERNAL_NET + "3",
   :ram => 1024,
	:frwd_host => 8080
  }
]
# Массив настроек для балансировщиков
serversbl=[
  {
    :hostname => "BCR1." + DOMAIN,
	:ip => BRIDGE_NET + "152",
    :ip_int => INTERNAL_NET + "4",
    :ram => 512,
	:frwd_host => 80
  }
]

unless Vagrant.has_plugin?("vagrant-reload")
  raise 'vagrant-reload is not installed! Please do { vagrant plugin install vagrant-reload } and try again.'
end

unless Vagrant.has_plugin?("vagrant-vbguest")
  raise 'vagrant-vbguest is not installed! Please do { vagrant plugin install vagrant-vbguest } and try again.'
end

unless Vagrant.has_plugin?("vagrant-disksize")
  raise 'vagrant-disksize is not installed! Please do { vagrant plugin install vagrant-disksize } and try again.'
end

Vagrant.configure(2) do |config|
    	#config.disksize.size = '5GB'
	
    serversbx.each do |machine|
        # Применяем конфигурации для каждой машины. Имя машины в переменной "machine[:hostname]"
        config.vm.box_check_update = false
		config.vm.define machine[:hostname] do |node|
			node.vm.box = "centos/7"
            node.vm.hostname = machine[:hostname]
			#node.vm.network "public_network", type: "dhcp"
            node.vm.network "private_network", ip: machine[:ip_int]
			node.vm.network :forwarded_port, guest: 80, host: 8888, auto_correct: true
			node.vm.network :forwarded_port, guest: 3306, host: 8889, auto_correct: true
			node.vm.network :forwarded_port, guest: 5432, host: 5433, auto_correct: true
			node.vm.synced_folder ".", "/vagrant", disabled: true
			node.vm.synced_folder "vagshare/", "/home/vagrant/provision", type: "rsync"
            node.vm.provider "virtualbox" do |vb|
                vb.customize ["modifyvm", :id, "--memory", machine[:ram]]
				#vb.customize ["modifyhd", :id, "--resize", "5120"]
				vb.cpus = 1
                vb.name = machine[:hostname]
            end
			
			node.vm.provision "shell", inline: <<-'END'
			
			sudo sed -i 's/enforcing/disabled/g' /etc/selinux/config
			
			END

			node.vm.provision :reload
			
			node.vm.provision "ansible_local" do |ansible|
				ansible.provisioning_path = "/home/vagrant/provision/"
				ansible.playbook = "playbook-ansible-bitrix.yml"				
			end
						
        end
	end
	
	serversbl.each do |machine|	
		config.vm.define machine[:hostname] do |node|
			node.vm.box = "ubuntu/bionic64"
            node.vm.hostname = machine[:hostname]
			#node.vm.network "public_network", type: "dhcp"
            node.vm.network "private_network", ip: machine[:ip_int]
			#node.vm.network :forwarded_port, guest: 80, host: 8888, auto_correct: true
			node.vm.synced_folder ".", "/vagrant", disabled: true
			node.vm.synced_folder "vagshare/", "/home/vagrant/provision", type: "rsync"
            node.vm.provider "virtualbox" do |vb|
                vb.customize ["modifyvm", :id, "--memory", machine[:ram]]
				#vb.customize ["modifyhd", :id, "--resize", "5120"]
				vb.cpus = 1
                vb.name = machine[:hostname]
            end
			
			node.vm.provision "ansible_local" do |ansible|
				ansible.provisioning_path = "/home/vagrant/provision/"
				ansible.playbook = "playbook-ansible-balancer.yml"
			end
	    end
	end
	
end
