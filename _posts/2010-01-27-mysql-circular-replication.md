---
layout: post
title: "MySQL Load-Balanced Cluster Guide - Part 1"
permalink: /2010/01/mysql-circular-replication
---

<strong><span style="color: #ff0000;">Warning: </span></strong><span style="color: #ff0000;">This is a very old post. I have re-posted it from an old blog as people were linking to it, but I would strongly advise you to make sure you understand what you are doing before following any of the steps herein.</span>

I’m currently working on setting up a load-balanced many-master MySQL cluster at work, without using MySQL Cluster as it’s incompatible with our needs. Finding good guides on how to do this has proven incredibly difficult, so I thought I would document the process here.

This first post is going to cover setting up the servers themselves and configuring MySQL replication.

**1. Set up the following three servers:
**


	* **Cluster Load Balancer** – One Public IP, One Internal IP – Ubuntu Jaunty Basic installation
	* **Cluster MySQL 1** – One Public IP, One Internal IP – Ubuntu Jaunty Basic + MySQL Server 5.x
	* **Cluster MySQL 2** – One Public IP, One Internal IP – Ubuntu Jaunty Basic + MySQL Server 5.x
</ul>
We’re using VPS.NET. If you’re going to do the same, use the ‘Ubuntu 8.04 LTS 64-bit Basic Installation’ template for the Load Balancer and ‘Ubuntu 8.04 x64 MySQL’ for the MySQL nodes. You’ll then need to do the following to upgrade to Jaunty:


	* Edit the file ‘*/etc/update-manager/release-upgrades</em>‘ and change the line ‘*Prompt=lts*‘ to ‘<em>Prompt=normal*‘.
	* Run the command ‘*do-release-upgrade*‘. Let it go through the entire process of installing Intrepid, then reboot.
	* Repeat the step above again to install of Jaunty.
</ul>
**2. On the two MySQL servers, edit /etc/mysql/my.cnf and comment out the line ‘*bind-address 127.0.0.1*‘ – the servers need to allow remote TCP/IP connections.
**
**3. On each server, add the following to the /etc/mysql/my.cnf file:
**
```
server-id = 1 # Increment this number by one for each server, so 1 for cluster-1, 2 for cluster-2 auto_increment_increment = 2 # Set to the number of nodes you have (or are likely to have) auto_increment_offset = 1 # Set to the same as the server-id replicate-same-server-id = 0 # To ensure the slave thread doesn't try to write updates that this node has produced. log-bin # Turn on binary logging (neccessary for replication) log-slave-updates # Neccessary for chain or circular replication relay-log # As above relay-log-index # As above
```


**Note:** On our servers, we’ve used an *auto_increment_increment *of 4, to allow us to add two more nodes to our cluster at a later date, without hitting auto-increment problems.

**4. Restart MySQL on both servers.
**
**5. On Cluster 1, log into the MySQL Console and run the following queries:
**
```
STOP SLAVE; CHANGE master TO master_host="cluster-2.yourdomain.com", master_user="{username}", master_password="{password}", master_log_file="mysqld-bin.000001", master_log_pos=98; START SLAVE;
```


**Note:** To verify the master_log_pos value, execute the query: “SHOW MASTER STATUS;” and check the Position column. Also, you may wish to use your internal IP addresses for the master_host option.

**6. On Cluster 2, log into the MySQL Console and run the following queries:
**
```
STOP SLAVE; CHANGE master TO master_host="cluster-1.yourdomain.com", master_user="{username}", master_password="{password}", master_log_file="mysqld-bin.000001", master_log_pos=98; START SLAVE;
```


**Note:** If on either server you get an error message at this point, verify that:


	* a) the master_log_file name in the query above matches the ones your servers are generating
	* b) your username and password for the other server are correct
	* c) that your servers can communicate with each other using the host names you specified.
	* Additionally, you may wish to remove AppArmor, as that can interfere with MySQL replication and cause all kinds of miscellaneous problems. To do so, run ‘*apt-get remove apparmor</em>‘ followed by ‘<em>apt-get purge apparmor*‘.
</ul>
You should now have master to master replication set up and running across your two MySQL servers. To test, create a database on cluster-1, then create a table within that database and insert some data into it. Next, log on to cluster-2 and you should be able to see that newly created table, complete with data.

If you want to ensure that it works both ways, insert some data on cluster-2 and you should see it on cluster-1. You will also want to test that your auto-increment settings are working correctly.

If you’ve got any questions about this article, or have any problems setting up the replication, please leave a comment below and I’ll do my best to help. The next article in this guide will cover setting up a proxy on the load balancer and hopefully, implementing failover.


	* Next Step: [Setting up MySQL Monitoring on the cluster nodes](/2010/01/mysql-monitoring-with-xinetd).
</ul>
