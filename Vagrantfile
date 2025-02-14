Vagrant.configure("2") do |config|
  config.vm.define "pxeserver" do |pxeserver|
    pxeserver.vm.box = 'bento/ubuntu-22.04'
    pxeserver.vm.host_name = 'pxeserver'
    pxeserver.vm.network "forwarded_port", guest: 80, host: 8080
    pxeserver.vm.network :private_network, ip: "10.0.0.20", virtualbox__intnet: 'pxenet'
    pxeserver.vm.network :private_network, ip: "192.168.57.10", adapter: 3
    pxeserver.vm.provider "virtualbox" do |vb|
      vb.memory = "1024"
      vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    end
    ssh_pub_key = File.readlines("./id_rsa/id_rsa.pub").first.strip
    pxeserver.vm.provision "shell", inline: <<-SHELL
        echo #{ssh_pub_key} >> ~vagrant/.ssh/authorized_keys
        echo #{ssh_pub_key} >> ~root/.ssh/authorized_keys
        sudo sed -i 's/\#PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
        systemctl restart sshd
    SHELL
  end

  config.vm.define "pxeclient" do |pxeclient|
    pxeclient.vm.box = 'bento/ubuntu-22.04'
    #pxeclient.vm.box = 'seskion/ubuntu-20.04-efi'
    pxeclient.vm.host_name = 'pxeclient'
    pxeclient.vm.network :private_network, ip: "10.0.0.21"
    pxeclient.vm.provider :virtualbox do |vb|
      vb.memory = "4096"
      vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
      vb.customize [
        'modifyvm', :id,
        '--nic1', 'intnet',
        '--intnet1', 'pxenet',
        '--nic2', 'nat',
        '--boot1', 'net',
        '--boot2', 'none',
        '--boot3', 'none',
        '--boot4', 'none'
      ]
      vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    end
  end
end

# Разворачивание дополнительной машины для Ansible при работе в среде Windows+Vagrant
Vagrant.configure("2") do |config|
  config.vm.define "ansible" do |ansible|
    ansible.vm.box = "bento/ubuntu-22.04"
    ansible.vm.network "private_network", ip: "192.168.57.100"
    ansible.vm.hostname = "ansible"
    ansible.vm.provider "virtualbox" do |vb|
      vb.memory = "1024"
      vb.cpus = "1"
    end

    # Установка Ansible внутри виртуальной машины и копирование публичного ключа
    ansible.vm.provision "shell", inline: <<-SHELL
      sudo apt-get update
      sudo apt-get install -y unrar
      sudo apt-get install -y ansible
    SHELL

    #file_to_copy = "./id_rsa/id_rsa"
    #ansible.vm.provision "file", source: file_to_copy, destination: "/home/vagrant/.ssh/id_rsa"
    # ansible.vm.provision "shell", inline: <<-SHELL
       #sudo mkdir -p /root/.ssh
    #   sudo cp /home/vagrant/.ssh/id_rsa /root/.ssh/
       #sudo chmod 600 /root/.ssh/id_rsa
    # SHELL
    
    #file_to_copy = "./id_rsa/id_rsa.pub"
    #ansible.vm.provision "file", source: file_to_copy, destination: "/home/vagrant/.ssh/id_rsa.pub"
     #ansible.vm.provision "shell", inline: <<-SHELL
       #sudo mkdir -p /root/.ssh
       #sudo cp /home/vagrant/.ssh/id_rsa.pub /root/.ssh/
       #sudo chmod 600 /root/.ssh/id_rsa.pub
     #SHELL
    
    #file_to_copy = "./ansible.rar"
    #ansible.vm.provision "file", source: file_to_copy, destination: "/home/vagrant/ansible.rar"
  end
end