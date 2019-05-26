---
layout: post
title: "MySQL Load-Balanced Cluster Guide – Part 3"
permalink: /2010/01/load-balancing-mysql-with-ha-proxy
---

<span style="color: #ff0000;">**Warning: **This is a very old post. I have re-posted it from an old blog as people were linking to it, but I would strongly advise you to make sure you understand what you are doing before following any of the steps herein.</span>

This article is the last in my MySQL Load-Balanced Cluster Guide series, following on from my earlier post today on [setting up MySQL Monitoring on the cluster nodes](/2010/01/mysql-monitoring-with-xinetd), this article is a guide to setting up the load balancer with HAProxy, using the monitoring scripts we’ve created to check the node status.

HAProxy is a well known, powerful load balancer. Best known for use with HTTP servers, but it can work with any kind of TCP traffic, including MySQL. Upcoming versions will support native MySQL monitoring, so we’ll be able to retire the custom scripts we created in the previous step, when they’re available in apt-get.

**1. Install HAProxy using the following apt-get command: **apt-get install haproxy

**2. Delete the file /etc/haproxy/haproxy.cfg and create a new one with the following contents:**

```
global log 127.0.0.1 local0 maxconn 4096 user haproxy group haproxy daemon

defaults log global mode tcp option tcplog option dontlognull retries 3 option redispatch maxconn 2000 contimeout 4000 clitimeout 50000 srvtimeout 30000 stats enable stats scope .

frontend mysql_cluster bind {load-balancer-ip}:3306 default_backend mysql_cluster

backend mysql_cluster mode tcp balance roundrobin stats enable option tcpka option httpchk server {node-name} {ip-address}:3306 weight 1 check port 9200 inter 5s rise 2 fall 2 server {node-name} {ip-address}:3306 weight 1 check port 9200 inter 5s rise 2 fall 2

listen stats {load-balancer-ip}:80 mode http option httpclose balance roundrobin stats uri / stats realm Haproxy\ Statistics stats auth {username}:{password} 
```

Once you’ve modified this file with settings appropriate to your environment, you’ll have a ready to go load balancer for MySQL. It’ll open port 3306 on the load balancer and forward those connections using round-robin load balancing to your two cluster servers. If either server goes down, it’ll be removed from the pool and no connections will be sent to it until it comes back up.

Additionally, it’ll create a statistics interface similar to the following, which you can access by going to your load balancing server’s address in your browser.

Each cluster node should be listed under the first “mysql_cluster” section. They should all appear in green, as they should all be live. If a server is down, it’ll appear red, as explained in the key at the top of the page. From this interface you can also see how long the load balancer has been up, how long each server has been up, how much downtime each have had, as well as the individual number of downtime incidents. As you can see in the image above, our ‘cluster-1′ server has had 21 separate downtime incidents, totalling just over 9 minutes of downtime.

So what do you need to change in your haproxy.cfg file? Here’s a quick rundown:

* Firstly, you’ll need to change the line starting ‘bind’ to use your load balancer’s public IP address.

* Secondly, make the same change on the line starting ‘listen stats’. You may also choose to put this service on a different port than port 80.

* Next, change the {username} and {password} section of the ‘stats auth’ line to ones of your choosing. This provides password protection to your web interface.

* Finally, for each of your MySQL cluster nodes, you’ll need an appropriate ‘server’ line in the ‘mysql_cluster’ backend group. The things you need to change are {node-name}, to the name of the cluster node you defined earlier (e.g. cluster-1, cluster-2) and {ip-address} to the cluster node’s internal IP address.

You may also wish to change the interval at which the load balancer will poll the MySQL servers, by changing ‘inter 5′ to a different number. For more information on HAProxy configuration, their detailed – if a little unfriendly – documentation is available [here](http://haproxy.1wt.eu/download/1.3/doc/configuration.txt" rel="nofollow).

The final step to get this running, is to restart haproxy: /etc/init.d/haproxy restart - You should now be able to access both your web interface on port 80 and your brand new MySQL cluster on port 3306!

If you’ve got any questions about these guides or setting up a similar cluster, please leave a comment below and I’ll do my best to answer it.
