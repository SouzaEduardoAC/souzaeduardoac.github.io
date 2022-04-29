---
layout: post
title: "Clustering RabbitMQ"
summary: "Setting a HA RabbitMQ!"
author: souzaeduardoac
date: '2020-11-05 20:21:23 -0300'
category: high-availability
thumbnail: /assets/img/posts/rabbitmq.jpeg
permalink: /blog/clustering-rabbitmq/
---

For this, we will use at least a couple VMs for single point failure mitigation.

### First, connect via ssh:

{% highlight shellscript %}
ssh first-server-username@first-sever-ip
{% endhighlight %}

In case you don't know your server's system distro/version, check with: 
{% highlight shellscript %}
cat /etc/*release
{% endhighlight %}

In this example, I will use a Cent OS 7 machine.

### Once connected, we need to download and install erlang, rabbitmq gives us a package ready of erlang for rabbitmq, we will use that.

{% highlight shellscript %}
wget https://github.com/rabbitmq/erlang-rpm/releases/download/v23.0.2/erlang-23.0.2-1.el7.x86_64.rpm
{% endhighlight %}

{% highlight shellscript %}
sudo yum localinstall </path/to/package>
{% endhighlight %}

### Then, download, install and enable rabbit-mq server

{% highlight shellscript %}
wget https://github.com/rabbitmq/rabbitmq-server/releases/download/v3.8.5/rabbitmq-server-3.8.5-1.el7.noarch.rpm
{% endhighlight %}

{% highlight shellscript %}
sudo yum localinstall </path/to/package>
{% endhighlight %}

{% highlight shellscript %}
chkconfig rabbitmq-server on
{% endhighlight %}

### I like to use a different disk to keep RabbitMQ data, so we are going to look for available disks

{% highlight shellscript %}
lsblk
{% endhighlight %}

*IF NEED FORMAT/MOUNT*

{% highlight shellscript %}
#Format into de desired partition type
sudo /sbin/mkfs -t ext4 /dev/sdX

#Add a label to it
sudo /sbin/e2label /dev/sdX /mnesia
sudo /sbin/tune2fs -l /dev/sdX |grep volume

#Edit /etc/fstab and add the following line:
LABEL=/mnesia          /mnesia                ext4    defaults        1 2
 
#Create a directory for mount the partition
sudo mkdir /mnesia
 
#Grant permission to it
sudo chown <group:user> /mnesia

#Mount the new partition
sudo mount /dev/sdX /mnesia

#Check if its everyone ok
df -k
{% endhighlight %}

## **Repeat everything for the others machines**

### Now we need to reset one of the node on the cluster by running the following commands:
{% highlight shellscript %}
sudo /usr/sbin/rabbitmqctl cluster_status
sudo /usr/sbin/rabbitmqctl status 
sudo /usr/sbin/rabbitmqctl stop_app 
sudo /usr/sbin/rabbitmqctl reset
{% endhighlight %}

### Then create and move the mnesia folder using the following commands:

{% highlight shellscript %}
sudo mkdir /mnesia/data
sudo cp -r /var/lib/rabbitmq/mnesia/* /mnesia/data/
sudo chown -R rabbitmq:rabbitmq /mnesia/data
sudo chmod 766 /mnesia/data
{% endhighlight %}

### Update the RabbitMQ Environment to set the **RABBITMQ_MNESIA_BASE** to the new folder that we created:

{% highlight shellscript %}
touch /tmp/rabbitmq-env.conf
cat <> /tmp/rabbitmq-env.conf
   CONFIG_FILE=/tmp/rabbitmq
   RABBITMQ_MNESIA_BASE=/opt/mnesia
End

sudo mv /tmp/rabbitmq-env.conf /etc/rabbitmq/
{% endhighlight %}

### Reboot the system and now join this node to the existing cluster nodes:

{% highlight shellscript %}
sudo /usr/sbin/rabbitmqctl status
sudo /usr/sbin/rabbitmqctl stop_app
sudo /usr/sbin/rabbitmqctl join_cluster
sudo /usr/sbin/rabbitmqctl join_cluster
sudo /usr/sbin/rabbitmqctl join_cluster
sudo /usr/sbin/rabbitmqctl start_app
{% endhighlight %}

### Set memory watermark to 0.7
{% highlight shellscript %}
sudo rabbitmqctl set_vm_memory_high_watermark 0.7
{% endhighlight %}

### Check limits the OS limits
{% highlight shellscript %}
ulimit -a
ulimit -n
{% endhighlight %}

### Increase them as desired: edit **/etc/sysctl.conf** add
{% highlight shellscript %}
fs.file-max = 100000
{% endhighlight %}

### Execute for reloading that config
{% highlight shellscript %}
sysctl -p
{% endhighlight %}

### Edit limits at **/etc/security/limits.conf**
{% highlight shellscript %}
soft nproc 65535
hard nproc 65535
soft nofile 65535
hard nofile 65535
{% endhighlight %}

### Enable pam: Edit **/etc/pam.d/**login and add
{% highlight shellscript %}
session required pam_limits.so
{% endhighlight %}

### Increase limits on .conf: edit **etc/systemd/system/rabbitmq-server.service.d/limits.conf** and add
{% highlight shellscript %}
[Service]
LimitNOFILE=65535
{% endhighlight %}

### Create new user for be user by interested party
{% highlight shellscript %}
sudo rabbitmqctl add_user desired-user-name 'desired-password'
sudo rabbitmqctl set_permissions -p "desired-vhost" "desired-user-name" ".*" ".*" ".*"
sudo rabbitmqctl set_user_tags desired-user-name desired-role
{% endhighlight %}

### Set up a cluster, ensure that all nodes have the same Erlang cookie file.
{% highlight shellscript %}
Starting independent nodes
rabbit@server1:~$ rabbitmq-server -detached
rabbit@server2:~$ rabbitmq-server -detached
{% endhighlight %}

### Copy cookie file
{% highlight shellscript %}
rabbit@server1:~$ scp /var/lib/rabbitmq/.erlang.cookie rabbitmq@server2:/var/lib/rabbitmq/.erlang.cookie
{% endhighlight %}

### Set up proper permission
Each of the nodes must have correct owner, group and permissions of the file erlang.cookie
{% highlight shellscript %}
rabbit@server:~$ sudo chown rabbitmq:rabbitmq /var/lib/rabbitmq/.erlang.cookie
rabbit@server:~$ sudo chmod 400 /var/lib/rabbitmq/.erlang.cookie
{% endhighlight %}

### Verify individual status
{% highlight shellscript %}
rabbit@server1:~$ sudo rabbitmqctl cluster_status
Cluster status of node 'rabbit@server1' ...
[{nodes,[{disc,['rabbit@server1']}]},
{running_nodes,['rabbit@server1']},
{partitions,[]}]
...done.
rabbit@server2:~$ sudo rabbitmqctl cluster_status
Cluster status of node 'rabbit@server2' ...
[{nodes,[{disc,['rabbit@server2']}]},
{running_nodes,['rabbit@server2']},
{partitions,[]}]
..done.
{% endhighlight %}

### Creating the cluster
{% highlight shellscript %}
rabbit@server2:~$ sudo rabbitmqctl stop_app
Stopping node rabbit@server2 ...done.
rabbit@server2:~$ sudo rabbitmqctl join_cluster rabbit@server1
Clustering node rabbit@server2 with [rabbit@server1] ...done.
rabbit@server2:~$ sudo rabbitmqctl start_app
Starting node rabbit@server2 ...done.
{% endhighlight %}

### Check status
{% highlight shellscript %}
rabbit@server1:~$ sudo rabbitmqctl cluster_status
Cluster status of node rabbit@server1 ...
[{nodes,[{disc,[rabbit@server1,rabbit@server2]}]},
{running_nodes,[rabbit@server1]},
{partitions,[]}]
...done.
{% endhighlight %}

### Check status of queues and messages
{% highlight shellscript %}
rabbit@server1:~$ sudo rabbitmqctl list_queues name consumers messages messages_ready messages_unacknowledged
Listing queues ...
amq.gen-QpH_mAGLAfcK9RDesq-7Xw 0 0 0 0
amq.gen-cJHYJpqVVpgdtwSHTzBS2g 0 0 0 0
amq.gen-rDbxpsjGtK73EMcwBZ3mRA 0 0 0 0
test_mailer 0 6 6 0
message_send 0 0 0 0
test_msg_queue 0 0 0 0
wont_reveal_queue_name 0 0 0 0
test_queue_123 0 3515 3515 0
test_queue_321 2 29 27 2
...done.
{% endhighlight %}

### In case deleting queue from cluster is needed
{% highlight shellscript %}
rabbit@server1:~$ sudo /usr/local/bin/rabbitmqadmin delete queue name=queuename;
{% endhighlight %}

#### Detach a node from a cluster
{% highlight shellscript %}
rabbit@server:~$ sudo rabbitmqctl stop_app
rabbit@server:~$ sudo rabbitmqctl reset
rabbit@server:~$ sudo rabbitmqctl start_app
{% endhighlight %}

### Check status
{% highlight shellscript %}
rabbit@server:~$ sudo rabbitmqctl cluster_status
{% endhighlight %}

### Set up mirroring policy
{% highlight shellscript %}
rabbit@server:~$ sudo rabbitmqctl set_policy ha-prod "^[A-Z]" \ '{"ha-mode":"all","ha-params":2,"ha-sync-mode":"automatic"}'
{% endhighlight %}

### Check it
{% highlight shellscript %}
rabbit@server:~$ sudo rabbitmqctl list_policies;
{% endhighlight %}
