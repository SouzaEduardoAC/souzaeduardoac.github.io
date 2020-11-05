---
layout: post
title: Clustering RabbitMQ
author: Eduardo Souza
date: '2020-11-05 20:21:23 -0300'
category:
        - high-availability
summary: Setting a HA RabbitMQ!
thumbnail: /assets/img/posts/rabbitmq.jpeg
---

For this, we will use at least a couple VMs for single point failure mitigation.

### First, connect via ssh:

> ssh first-server-username@first-sever-ip

In case you don't know your server's system distro/version, check with: 
>cat /etc/*release

In this example, I will use a Cent OS 7 machine.

### Once connected, we need to download and install erlang, rabbitmq gives us a package ready of erlang for rabbitmq, we will use that.

> wget https://github.com/rabbitmq/erlang-rpm/releases/download/v23.0.2/erlang-23.0.2-1.el7.x86_64.rpm
>
> sudo yum localinstall </path/to/package>

### Then, download, install and enable rabbit-mq server

> wget https://github.com/rabbitmq/rabbitmq-server/releases/download/v3.8.5/rabbitmq-server-3.8.5-1.el7.noarch.rpm
>
> sudo yum localinstall </path/to/package>
>
> chkconfig rabbitmq-server on

### I like to use a different disk to keep RabbitMQ data, so we are going to look for available disks

> lsblk

*IF NEED FORMAT/MOUNT*

> Format into de desired partition type
> * sudo /sbin/mkfs -t ext4 /dev/sdX
>
> Add a label to it
> * sudo /sbin/e2label /dev/sdX /mnesia
> * sudo /sbin/tune2fs -l /dev/sdX |grep volume
>
> Edit /etc/fstab and add the following line:
> * LABEL=/mnesia          /mnesia                ext4    defaults        1 2
> 
> Create a directory for mount the partition
> * sudo mkdir /mnesia
> 
> Grant permission to it
> * sudo chown <group:user> /mnesia
>
> Mount the new partition
> * sudo mount /dev/sdX /mnesia
>
> Check if its everyone ok
> * df -k

## **Repeat everything for the others machines**

### Now we need to reset one of the node on the cluster by running the following commands:
> * sudo /usr/sbin/rabbitmqctl cluster_status
> * sudo /usr/sbin/rabbitmqctl status
> * sudo /usr/sbin/rabbitmqctl stop_app
> * sudo /usr/sbin/rabbitmqctl reset

### Then create and move the mnesia folder using the following commands:

> * sudo mkdir /mnesia/data
> * sudo cp -r /var/lib/rabbitmq/mnesia/* /mnesia/data/
> * sudo chown -R rabbitmq:rabbitmq /mnesia/data
> * sudo chmod 766 /mnesia/data

### Update the RabbitMQ Environment to set the **RABBITMQ_MNESIA_BASE** to the new folder that we created:

> * touch /tmp/rabbitmq-env.conf
> * cat <> /tmp/rabbitmq-env.conf
>   * CONFIG_FILE=/tmp/rabbitmq
>   * RABBITMQ_MNESIA_BASE=/opt/mnesia
>
> End
> 
> sudo mv /tmp/rabbitmq-env.conf /etc/rabbitmq/

### Reboot the system and now join this node to the existing cluster nodes:

> * sudo /usr/sbin/rabbitmqctl status
> * sudo /usr/sbin/rabbitmqctl stop_app
> * sudo /usr/sbin/rabbitmqctl join_cluster
> * sudo /usr/sbin/rabbitmqctl join_cluster
> * sudo /usr/sbin/rabbitmqctl join_cluster
> * sudo /usr/sbin/rabbitmqctl start_app

### Set memory watermark to 0.7
> sudo rabbitmqctl set_vm_memory_high_watermark 0.7

### Check limits the OS limits
> * ulimit -a
> * ulimit -n

### Increase them as desired: edit **/etc/sysctl.conf** add
> fs.file-max = 100000

### Execute for reloading that config
> sysctl -p

### Edit limits at **/etc/security/limits.conf**
> * soft nproc 65535
> * hard nproc 65535
> * soft nofile 65535
> * hard nofile 65535

### Enable pam: Edit **/etc/pam.d/**login and add
> session required pam_limits.so

### Increase limits on .conf: edit **etc/systemd/system/rabbitmq-server.service.d/limits.conf** and add
> [Service]
>
> LimitNOFILE=65535

### Create new user for be user by interested party
> sudo rabbitmqctl add_user desired-user-name 'desired-password'
> sudo rabbitmqctl set_permissions -p "desired-vhost" "desired-user-name" ".*" ".*" ".*"
> sudo rabbitmqctl set_user_tags desired-user-name desired-role

### Set up a cluster, ensure that all nodes have the same Erlang cookie file.
Starting independent nodes
> * rabbit@server1:~$ rabbitmq-server -detached
> * rabbit@server2:~$ rabbitmq-server -detached

### Copy cookie file
> * rabbit@server1:~$ scp /var/lib/rabbitmq/.erlang.cookie rabbitmq@server2:/var/lib/rabbitmq/.erlang.cookie

### Set up proper permission
Each of the nodes must have correct owner, group and permissions of the file erlang.cookie
> * rabbit@server:~$ sudo chown rabbitmq:rabbitmq /var/lib/rabbitmq/.erlang.cookie
> * rabbit@server:~$ sudo chmod 400 /var/lib/rabbitmq/.erlang.cookie

### Verify individual status
> rabbit@server1:~$ sudo rabbitmqctl cluster_status

> Cluster status of node 'rabbit@server1' ...
>
> [{nodes,[{disc,['rabbit@server1']}]},
>
> {running_nodes,['rabbit@server1']},
>
> {partitions,[]}]
> ...done.

> rabbit@server2:~$ sudo rabbitmqctl cluster_status
>
> Cluster status of node 'rabbit@server2' ...
>
> [{nodes,[{disc,['rabbit@server2']}]},
>
> {running_nodes,['rabbit@server2']},
> 
> {partitions,[]}]
>...done.

### Creating the cluster
> rabbit@server2:~$ sudo rabbitmqctl stop_app
> 
> Stopping node rabbit@server2 ...done.
>

> rabbit@server2:~$ sudo rabbitmqctl join_cluster rabbit@server1
> Clustering node rabbit@server2 with [rabbit@server1] ...done.
>
> rabbit@server2:~$ sudo rabbitmqctl start_app
>
> Starting node rabbit@server2 ...done.

### Check status
> rabbit@server1:~$ sudo rabbitmqctl cluster_status

> Cluster status of node rabbit@server1 ...
>
> [{nodes,[{disc,[rabbit@server1,rabbit@server2]}]},
>
> {running_nodes,[rabbit@server1]},
>
> {partitions,[]}]
>...done.

### Check status of queues and messages
> rabbit@server1:~$ sudo rabbitmqctl list_queues name consumers messages messages_ready messages_unacknowledged

> Listing queues ...
>
> amq.gen-QpH_mAGLAfcK9RDesq-7Xw 0 0 0 0
>
> amq.gen-cJHYJpqVVpgdtwSHTzBS2g 0 0 0 0
>
> amq.gen-rDbxpsjGtK73EMcwBZ3mRA 0 0 0 0
>
> test_mailer 0 6 6 0
>
> message_send 0 0 0 0
>
> test_msg_queue 0 0 0 0
>
> wont_reveal_queue_name 0 0 0 0
>
> test_queue_123 0 3515 3515 0
>
> test_queue_321 2 29 27 2
> ...done.

### In case deleting queue from cluster is needed
> rabbit@server1:~$ sudo /usr/local/bin/rabbitmqadmin delete queue name=queuename;

#### Detach a node from a cluster
> rabbit@server:~$ sudo rabbitmqctl stop_app

> rabbit@server:~$ sudo rabbitmqctl reset

> rabbit@server:~$ sudo rabbitmqctl start_app

### Check status
> rabbit@server:~$ sudo rabbitmqctl cluster_status

### Set up mirroring policy
> rabbit@server:~$ sudo rabbitmqctl set_policy ha-prod "^[A-Z]" \ '{"ha-mode":"all","ha-params":2,"ha-sync-mode":"automatic"}'

### Check it
> rabbit@server:~$ sudo rabbitmqctl list_policies;