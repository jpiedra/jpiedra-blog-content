+++
date = "2017-09-11T21:55:02-04:00"
title = "Connecting to MySQL on a VirtualBox Linux VM"
keywords = [ "jpiedra", "development", "virtualization", "mysql", "tips" ]
tags = [ "development", "mysql", "virtualbox", "virtualization" ]
+++

Here are a few considerations to keep in mind if you're running a VirtualBox VM with MySQL on it, and need an application on your host operating system to be able to connect to it.

<!--more-->
This brief writeup serves mostly to document some small settings here and there that I had to set in order to get this working. I have a VirtualBox VM running Ubuntu 14.04, and all of my Node.js web development projects are on there. I'm the process of moving these over to my host, now that I've found an IDE (Visual Studio Code) that I prefer to use... as well as come to the realization that Chrome won't quite work on my VM, leaving me with no choice if I want to use Node's debugging features!

<h2>The 'bind-address' Option</h2>
The first thing you'll want to do, even before forwarding ports on your VM, is to make sure your MySQL process isn't bound to localhost (127.0.0.1), which it likely might be by default. The file you'll want to edit will be <i>/etc/mysql/my.cnf</i>, and you'll change this setting:

<pre>
bind-address		= 0.0.0.0
</pre>

Setting <a href="https://dev.mysql.com/doc/refman/5.7/en/server-options.html#option_mysqld_bind-address">bind-address</a> to 0.0.0.0 allows the MySQL process' socket to accept TCP/IP traffic on all IP4 network interfaces, rather than just connections over the loopback interface. This is important if you have anything on your host that needs to be able to make database queries on a database running in the VM. 

<h2>User with Remote Access</h2>
This next step is really only appropriate in a development environment, which is what this article assumes you want to configure. As you'll soon see, we set up a new user with the same name as one we would already have, only we set indicate that this user will have access from <i>any</i> host. Not a good idea for production systems; you should be stricter with this access, specifying specific IP addresses - or, better yet, use a built-in admin account that can only connect from <i>localhost</i>.

You may have a user set up to access your server on localhost, named 'apiuser.' It may have been created with using the command:

<pre>
mysql> CREATE USER 'apiuser'@'localhost' IDENTIFIED BY 'password';
mysql> GRANT ALL PRIVILEGES ON *.* TO 'apiuser'@'localhost'
    ->     WITH GRANT OPTION;
</pre>

To set up an identical user that can access the database from any location:

<pre>
mysql> CREATE USER 'apiuser'@'%' IDENTIFIED BY 'password';
mysql> GRANT ALL PRIVILEGES ON *.* TO 'apiuser'@'%'
    ->     WITH GRANT OPTION;
mysql> FLUSH PRIVILEGES;
</pre>

Now when you attempt to connect to this server from your host OS, this account will be used instead of the 'localhost' account, and your connection will complete successfully. That is, once you make sure to...

<h2>Forward Port 3306</h2>

All the needs to happen now is VirtualBox (or whatever provider you're using for virtualization) needs to let traffic through from that port on the VM to your host OS. <a href="https://www.virtualbox.org/manual/ch06.html#natforward">VBoxManage</a>, a command-line utility for VirtualBox, makes this pretty easy. If your VM is named "WebDevTest," then the command to forward port 3306 traffic would look like this:

<pre>
VBoxManage modifyvm "WebDevTest" --natpf1 "guestmysql,tcp,,3306,,3306"
</pre>

Alternatively, you can also do this through the GUI, by <b>right-clicking your VM -> Settings... -> Network -> Adapter (NAT) -> Port Forwarding</b> and adding a rule to forward traffic from your host port to your guest port. You can keep this the same, or change the host port somewhat (3316); it's a matter of preference, so long as that other port you choose isn't being used for something else.