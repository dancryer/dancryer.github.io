---
layout: post
title: "MySQL Load-Balanced Cluster Guide - Part 2"
permalink: /2010/01/mysql-monitoring-with-xinetd
---

<span style="color: #ff0000;">**Warning: **This is a very old post. I have re-posted it from an old blog as people were linking to it, but I would strongly advise you to make sure you understand what you are doing before following any of the steps herein.</span>

Following on from yesterday’s guide to [setting up MySQL master-master replication](/2010/01/mysql-circular-replication), the first of today’s guides is how to set up a script to monitor the status of your MySQL cluster nodes, which we’ll use in the next guide to set up our proxy.

On every one of your MySQL cluster nodes, follow the steps below to get it ready for status reporting:

**1. Run the following apt-get command: *apt-get install xinetd php5-cli php5-mysql*** – We need xinetd to host our status server, and php5 for the status checking script.

**2. Put the following code into a new file: /etc/xinetd.d/mysqlchk** – This creates an xinetd “server” on port 9200, using the /opt/mysql-status.php file to generate the response. Our proxy will connect to this server to check the status of the node.

```
# 
# /etc/xinetd.d/mysqlchk 
# 
service mysqlchk { 
    flags = REUSE 
    socket_type = stream 
    protocol = tcp 
    port = 9200 
    wait = no 
    user = mysql 
    server = /opt/mysql-status.php 
    log_on_failure += USERID 
    disable = no 
}
```


**3. Add the following line at the bottom of /etc/services** – This registers your newly created xinetd service with the system to allow it to function.

```
mysqlchk 9200/tcp # MySQL Monitoring
```

**4. Create the PHP file /opt/mysql-status.php with the code below.**

This script tries to connect to your MySQL server, and if it succeeds, checks the ‘Seconds_Behind_Master’ value of the slave status to make sure it is not too far behind it’s master. If either of these checks fail, it will return a 503 error to the client, telling it that the server is down.

```php
#!/usr/bin/php 
<?php 
/** 
* MySQL Replication Monitor 
*/

// Connection details: 
$_host = 'localhost'; 
$_username = '{username}'; 
$_password = '{password}'; 
$_timeout = 60; // Number of seconds behind master this node can fall before being marked as "down".

try { 
    $pdo = new PDO('mysql:host='.$_host, $_username, $_password); 
    $pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);

    $stmt = $pdo->query('SHOW SLAVE STATUS'); $status = $stmt->fetch(PDO::FETCH_ASSOC);

    if($status == false) { 
        serverIsDown('Querying slave status failed.'); 
    }

    if(intval($status['Seconds_Behind_Master']) > $_timeout) { 
        serverIsDown('Slave is behind.'); 
    } 
} catch(Exception $ex) { 
    if(!is_null($pdo)) unset($pdo);
    serverIsDown('Failed to connect to MySQL.'); 
}
    
unset($pdo);
serverIsUp();

function serverIsDown($message) { 
    $output = 'Server is currently down. Message:' . $message; 
    print "HTTP/1.1 503 Service Unavailable\r\n"; 
    print "Date: " . date('r') . "\r\n"; 
    print "Server: MySQL Monitor/1.0\r\n"; 
    print "Connection: close\r\n"; 
    print "Content-Type: text/plain\r\n"; 
    print "Content-Length: " . strlen($output) . "\r\n"; 
    print "\r\n"; 
    print $output; 
    die; 
}

function serverIsUp() { 
    $output = 'Server is up. All is okay.'; 
    print "HTTP/1.1 200 OK\r\n"; 
    print "Date: " . date('r') . "\r\n"; 
    print "Server: MySQL Monitor/1.0\r\n"; 
    print "Connection: close\r\n"; 
    print "Content-Type: text/plain\r\n"; 
    print "Content-Length: " . strlen($output) . "\r\n"; 
    print "\r\n"; 
    print $output; 
    die; 
}
```

**5. chown -fv mysql:mysql /opt/mysql-status.php** – Give ownership of the status checking script to mysql.
**6. chmod -fv +x /opt/mysql-status.php** – Give execute permissions on the script.
**7. /etc/init.d/xinetd restart** – Restart xinetd, to make all of the changes we’ve just made live.

Having completed the steps above, you will now have a script that checks the current status of your MySQL server, responding to connections on port 9200 of your server. In the next article, I’ll describe how to set up your load balancing server with HAProxy, and utilise the server we’ve just created to check which nodes are live at any given time.