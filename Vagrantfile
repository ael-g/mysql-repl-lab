# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  $master1_ip = "192.168.1.10"
  $master2_ip = "192.168.1.11"

  $database_name = "testdb"

  def master_provisionning(masterid, masterip)
    return "
      su -
      export DEBIAN_FRONTEND=noninteractive 
      apt-get -y install mysql-server

      export MYSQL_CONF=/etc/mysql/my.cnf
      echo '
        [mysqld]
        bind-address        = "+masterip+"
        log_bin             = /var/log/mysql/mysql-bin.log
        server-id           = "+masterid.to_s+"
        binlog_do_db        = "+$database_name+"
      ' > /etc/mysql/conf.d/repl.cnf
      service mysql restart
      echo \"use mysql; delete from user where User='repl'; flush privileges;\" | mysql
      echo \"create user 'repl'@'%';\" | mysql
      echo \"grant replication slave on *.* to 'repl'@'%';\" | mysql
      echo \"flush privileges;\" | mysql
      echo \"create database if not exists "+$database_name+";\" | mysql
    "
  end

  $master_shell_provisionning = 
  ########## MYSQL nodes ##########
  config.vm.define "master1" do |master1|
    master1.vm.box = "debian/jessie64"
    master1.vm.network "private_network", ip: $master1_ip
    master1.vm.provision "shell",
      inline:master_provisionning(1, $master1_ip)
  end

  config.vm.define "master2" do |master2|
    master2.vm.box = "debian/jessie64"
    master2.vm.network "private_network", ip: $master2_ip
    master2.vm.provision "shell",
      inline:master_provisionning(2, $master2_ip)
  end


  ########## CLIENTS ##########
  config.vm.define "client1" do |client1|
    client1.vm.box = "debian/jessie64"
    client1.vm.network "private_network", ip: "192.168.1.100" 
  end

end
