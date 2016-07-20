# mysql-repl-lab
This repository is a lab to play with mysql master/master replication, see how to make it works, how to check it keeps working and what to do in case it's broken.

Mysql replication looks like the simplest way to provide high availability, load-balancing and data security capabilities to a mysql database.
There are other technics to achieve the same goals like MySQL Fabric and MySQL cluster, but look more complex or would need more machines to bootstrap.

## Things to know about mysql replication
- It's **asynchronous**, which means that if a write query is made on a replicated server, this server will acknoweldge the query straight away and won't wait for replicated servers to do the write too. This is good because it will have almost no performance impacts on the number of queries you can process, but it means that if your replicated server dies, you will still be aknowledging queries that are not replicated any more.
- You should still do **backups**. Because shit can always happens, like you have a couple of replicated servers, the first one dies and before fixing it the second one dies too. Or somebody does some drop tables that he should be doing, and your data are lost whatever the number of replicats you have.
- **Replication can break**, you need to know when this happens and you need to know what to do to make it working again. For sure it will break if you have hardware failure, or network failure, or anything that would make one of your replicate inaccessible. In those cases you'll have to bring your replicate back to life and resync your DB.

## The lab
Is actually just a Vagrantfile that will bootstrap two virtual machines, install mysql and setup master/master replication on them. It suppose you have a functionnal vagrant installation and a virtualbox provider for it.
By looking at the Vagrantfile, you'll see that we'll create a special conf file `repl.cnf`.
You'd probably guessed already but it's absolutely not intended to be used as-is for production environments!
Because it makes mysql listen on every network interfaces and set up no password for the root account. This is only done to make testing convenient.

### 1. Start the lab
As easy as starting any Vagrant env :
```
cd mysql-repl-lab
vagrant up
vagrant status
```

### 2. Connect to nodes and start replication
So servers are started with mysql running, we still have to manually start the sync/replication process on the 2 nodes. 
Please have a look at the content of `/root/start_replication.sh` script that's generated from the Vagrantfile during provisionning! This is short and is where the real work is done, we'll come back to it later.

On the first master :
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
We now have two nodes with a master/master replication running for database `testdb`.
Let's try it! On both masters we should have an empty `testdb` database:
```
root@master1 # mysql
mysql> show databases;                                             
+--------------------+                                             
| Database           |                                             
+--------------------+                                             
| information_schema |                                             
| mysql              |                                             
| performance_schema |                                             
| testdb             |                                             
+--------------------+                                             
4 rows in set (0.00 sec)                                           

mysql> show tables from testdb; 
Empty set (0.00 sec)
mysql> 

root@master2 # mysql
mysql> show tables from testdb; 
Empty set (0.00 sec)
mysql> 
```

Ok so testdb is indeed empty, lets create a `master1_table` from master1 :
```
root@master1 # mysql
mysql> use testdb 
Database changed
mysql> create table master1_table (a integer);
Query OK, 0 rows affected (0.00 sec)           

mysql>
```

And now if we check on master2 :
```
root@master2 # mysql
mysql> show tables from testdb;
+------------------+           
| Tables_in_testdb |           
+------------------+           
| master1_table    |           
+------------------+           
1 row in set (0.00 sec)
```

So if we create a table on master1, it's replicating on master2! You're free to check the opposite or to test other types of SQL queries, either from master1 or from master2.

### 4. Spotting replication break down
From the previous points we can see that replication is quite easy to set-up.
But it looks kind of magical isn't it ? Especialy if you're just copy/pasting commands without reading the Vagrantfile.

#### Master/Master and Master/Slave replication
To understand how replication works in mysql, there already are good articles (see Resources part of this README) and I won't paraphrase them but I will show how we can monitor the replication process and see if it's working or not.
It's actually quite easy, what's important to understand is that to replicate queries from mysql node A **to** node B, we have to tell node B that master is node A, and we issue a SQL query : 
```
change master to    master_host     = node_A_I, 
                    master_user     = '...',
                    master_password = '...',
                    master_log_file = '...',
                    master_log_pos  = '...'
```
It tells node B how to connect to node A, what is the log transaction file on the master node and which position it's currently at.
And those two last fields are really the important bits. It allows node B (which is now a slave of node A) to replicates queries which are executed on node A by reading the transaction log and by knowing where it should starts thanks to the position index.
This is the case of master/slave replication: the master receives the `write` queries, slave replicates them, and then master and slave can serve read queries.

And it's not so different in the case of master/master replication. Instead of having one slave replicating what's written on the master, we'll have each node replicating what the other one is writing to its DB, in other word each node will be the slave of the other one and we'll execute the ``change master to ...`` query on each node, pointing the master as the other node.

#### `master_log_file` and `master_log_pos`

#### Monitoring replication

### 5. Fixing broken replication

## Resources
- https://www.digitalocean.com/community/tutorials/how-to-set-up-mysql-master-master-replication
- https://www.percona.com/blog/2013/01/09/how-does-mysql-replication-really-work/
