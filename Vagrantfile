# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  $master1_ip = "192.168.1.10"
  $master2_ip = "192.168.1.11"

  $database_name = "testdb"
  $repl_user = "repl"

  def start_replication_script(opposite_masterip)
    return "
      #!/bin/sh
      ret=\\$(echo show master status | mysql -h "+opposite_masterip+" -u repl | grep "+$database_name+")
      file=\\$(echo \\$ret | awk '{print \\$1}') 
      position=\\$(echo \\$ret | awk '{print \\$2}')
      echo slave stop | mysql
      echo \"change master to master_host = '"+opposite_masterip+"', 
        master_user = '"+$repl_user+"',
        master_password = '',
        master_log_file = '\\$file',
        master_log_pos = \\$position;
        \" | mysql
        echo slave start | mysql
    "
  end

  def master_provisionning(masterid, masterip, opposite_masterip)
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
      echo \"use mysql; delete from user where User='"+$repl_user+"'; flush privileges;\" | mysql
      echo \"create user '"+$repl_user+"'@'%';\" | mysql
      echo \"grant replication client on *.* to '"+$repl_user+"'@'%';\" | mysql
      echo \"grant replication slave on *.* to '"+$repl_user+"'@'%';\" | mysql
      echo \"flush privileges;\" | mysql
      echo \"create database if not exists "+$database_name+";\" | mysql
      cat > /root/start_replication.sh <<EOF 
      "+start_replication_script(opposite_masterip)+"
EOF
      chmod +x /root/start_replication.sh
    "
  end

  ########## MYSQL nodes ##########
  config.vm.define "master1" do |master1|
    master1.vm.box = "debian/jessie64"
    master1.vm.network "private_network", ip: $master1_ip
    master1.vm.provision "shell",
      inline:master_provisionning(1, $master1_ip, $master2_ip)
  end

  config.vm.define "master2" do |master2|
    master2.vm.box = "debian/jessie64"
    master2.vm.network "private_network", ip: $master2_ip
    master2.vm.provision "shell",
      inline:master_provisionning(2, $master2_ip, $master1_ip)
  end

end
