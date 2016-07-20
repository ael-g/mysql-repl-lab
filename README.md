# mysql-repl-lab
This repository is a lab to play with mysql master/master replication, see how to make it works, how to check it keeps working and what to do in case it's broken.

Mysql replication looks like the simplest way to provide high availability, load-balancing and data security capabilities to a mysql database.
There are other technics to achieve the same goals like MySQL Fabric and MySQL cluster, but look more complex or would need more machines to bootstrap.

## Things to know about mysql replication
- It's **asynchronous**, which means that if a write query is made on a replicated server, this server will acknoweldge the query straight away and won't wait for replicated servers to do the write too. This is good because it will have almost no performance impacts on the number of queries you can process, but it means that if your replicated server dies, you will still be aknowledging queries that are not replicated any more.
- You should still do **backups**. Because shit can always happens, like you have a couple of replicated servers, the first one dies and before fixing it the second one dies too. Or somebody does some drop tables that he should be doing, and your data are lost whatever the number of replicats you have.
- **Replication can break**, you need to know when this happens and you need to know what to do to make it working again. For sure it will break if you have hardware failure, or network failure, or anything that would make one of your replicate inaccessible. In those cases you'll have to bring your replicate back to life and resync your DB.

## The lab
Is actually just a Vagrantfile that will bootstrap two virtual machines, install mysql and setup master/master replication on them. So it suppose you have a functionnal vagrant installation and a virtualbox provider for it.

### 1. Start the lab
```
cd mysql-repl-lab
vagrant up
vagrant status
```

### 2. Connect to nodes and start replication
So servers are started with mysql running, we still have to manually start the sync/replication process on the 2 nodes. 
Please have a look at the content of `/root/start_replication.sh` script that's generated from the Vagrantfile during provisionning!

On a first master :
```
vagrant ssh master1
sudo su -
./start_replication.sh
```

On the second master :
```
vagrant ssh master2
sudo su -
./start_replication.sh
```

### 3. Testing that it's working


### 4. Spotting replication breaks

### 5. Fixing broken replication

## Resources
- https://www.digitalocean.com/community/tutorials/how-to-set-up-mysql-master-master-replication
- https://www.percona.com/blog/2013/01/09/how-does-mysql-replication-really-work/
