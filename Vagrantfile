# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/bionic64"

  config.vm.define "fleet" do |fleet|
    fleet.vm.provision "file", source: "./redis.service", destination: "redis.service"
    fleet.vm.provision "file", source: "./mysql.service", destination: "mysql.service"
    fleet.vm.hostname = "fleet"
    fleet.vm.network "private_network", ip: "192.168.48.10"
    fleet.vm.network "forwarded_port", guest: 8080, host: 8080
    fleet.vm.provision :shell, inline: <<-SHELL
      mv /home/vagrant/mysql.service /etc/systemd/system/
      mv /home/vagrant/redis.service /etc/systemd/system/
      apt-get remove -y docker docker-engine docker.io containerd runc
      apt-get install -y apt-transport-https ca-certificates gnupg-agent software-properties-common mysql-client-5.7 unzip
      curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
      add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
      apt-get update
      apt-get install -y docker-ce docker-ce-cli containerd.io
      systemctl daemon-reload
      systemctl start mysql
      systemctl start redis
      openssl req -newkey rsa:4096 -x509 -sha256 -days 3650 -nodes -out /root/keypair.crt -keyout /root/keypair.key -subj "/C=SI/ST=Ljubljana/L=Ljubljana/O=Security/OU=IT Department/CN=192.168.48.10"
      wget https://github.com/kolide/fleet/releases/latest/download/fleet.zip
      unzip fleet.zip 'linux/*' -d fleet
      cp fleet/linux/fleet* /usr/bin/
      chmod +x /usr/bin/fleet*
    SHELL

  end

  config.vm.define "osquery" do |osquery|
    osquery.vm.hostname = "osquery"
    osquery.vm.network "private_network", type: "dhcp"
    osquery.vm.provision :shell, inline: <<-SHELL
      apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 1484120AC4E9F8A1A577AEEE97A80C63C9D8B80B
      add-apt-repository 'deb [arch=amd64] https://pkg.osquery.io/deb deb main'
      apt-get update
      apt-get install -y osquery
    SHELL
  end

  config.vm.provision "shell", inline: <<-SHELL
    apt-get update
    apt-get install -y telnet vim net-tools wget traceroute openssl curl
  SHELL
end
