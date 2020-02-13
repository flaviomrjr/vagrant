$script_mysql = <<-SCRIPT
   apt-get update && \
   apt-get install mysql-server-5.7 -y && \
   mysql -e "create user 'phpuser'@'%' identified by 'pass';"
SCRIPT

Vagrant.configure("2") do |config|
   #CHOOSE THE VM BOX
   config.vm.box = "ubuntu/bionic64"

   #SET GLOBAL HARDWARE CONFIG
   config.vm.provider "virtualbox" do |vb|
      vb.memory = 1024
      vb.cpus = 2
   end

   #CREATE MYSQL VM
   config.vm.define "mysql" do |mysql|
      #CREATE PORT FORWARDED
      mysql.vm.network "forwarded_port", guest: 80, host:8080
      mysql.vm.network "private_network", ip: "192.168.10.3"
      #CREATE BRIDGE NETWORK
      mysql.vm.network "public_network"
      #PROVISIONING PUBLIC RSA KEY
      mysql.vm.provision "shell",
         inline: "cat /configs/id_rsa.pub >> .ssh/authorized_keys"
      #PROVISIONING NGINX
      mysql.vm.provision "shell",
         inline: "apt-get update && apt-get install nginx -y"
      #PROVISIONING MYSQL
      mysql.vm.provision "shell", inline: $script_mysql
      mysql.vm.provision "shell",
         inline: "cat /configs/mysqld.cnf > /etc/mysql/mysql.conf.d/mysqld.cnf"
      mysql.vm.provision "shell", inline: "service mysql restart"
      #CREATE FOLDER MAP
      mysql.vm.synced_folder "./configs", "/configs"
      #DISABLE VAGRANT DEFAULT FOLDER
      mysql.vm.synced_folder ".", "/vagrant", disabled: true
   end
   
   #CREATE PHP VM
   config.vm.define "php" do |php|
      php.vm.network "forwarded_port", guest: 8888, host:8888
      php.vm.network "private_network", ip: "192.168.10.2"
      php.vm.network "public_network"
      #INSTALL PUPPET
      php.vm.provision "shell",
         inline: "apt-get update && apt-get install puppet -y" 
      #EXECUTE MANIFEST PHPWEB
      php.vm.provision "puppet" do |puppet|
         puppet.manifests_path = "./configs/manifests"
         puppet.manifest_file = "phpweb.pp"  
      end
   end

   #CREATE MYSQLSERVER VM
   config.vm.define "mysqlserver" do |mysqlserver|
      #SET VM HARDWARE CONFIG
      mysqlserver.vm.provider "virtualbox" do |vb|
         vb.memory = 2048
         vb.cpus = 2
         vb.name = "ubuntu_bionic_php7"
      end
      mysqlserver.vm.network "private_network", ip: "192.168.10.4"
      mysqlserver.vm.network "public_network"
      mysqlserver.vm.provision "shell",
         inline: "cat /vagrant/configs/id_rsa.pub >> .ssh/authorized_keys"
   end
   
   #CREATE ANSIBLE VM
   config.vm.define "ansible" do |ansible|
      ansible.vm.network "private_network", ip: "192.168.10.5"
      ansible.vm.network "public_network"
      #INSTALL ANSIBLE
      ansible.vm.provision "shell",
         inline: "apt-get update && \
         apt-get install -y software-properties-common && \
         apt-add-repository --yes --update ppa:ansible/ansible && \
         apt-get install -y ansible"
      #CONFIGURE SSH KEY
      ansible.vm.provision "shell",
         inline: "cp /vagrant/id_rsa /home/vagrant/.ssh/ && \
         chmod 600 /home/vagrant/.ssh/id_rsa && \
         chown vagrant:vagrant /home/vagrant/.ssh/id_rsa"
      #RUN ANSIBLE PLAYBOOK
      ansible.vm.provision "shell",
         inline: "ansible-playbook -i /vagrant/configs/ansible/hosts \
         /vagrant/configs/ansible/playbook.yml"
   end

   #CREATE MEMCACHED VM ON CENTOS7
   config.vm.define "memcached" do |memcached|
      memcached.vm.network "private_network", ip: "192.168.10.6"
      memcached.vm.network "public_network"
      #SET BOX
      memcached.vm.box = "centos/7"
      memcached.vm.provider "virtualbox" do |vb|
          vb.memory = 512
          vb.cpus = 1
          vb.name = "centos7_memcached"
      end
   end

   config.vm.define "dockerhost" do |dockerhost|
      dockerhost.vm.network "private_network", ip: "192.168.10.7"
      dockerhost.vm.network "public_network"
      dockerhost.vm.provider "virtualbox" do |vb|
          vb.memory = 2048
          vb.cpus = 1
          vb.name = "ubuntu_dockerhost"
      end
      dockerhost.vm.provision "shell", 
          inline: "apt-get uptade&& apt-get install -y docker.io"
  end
   
end