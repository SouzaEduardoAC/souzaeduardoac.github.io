---
layout: post
title: "Load Balancer with High Availability"
summary: "Load balacing with high availability!"
author: souzaeduardoac
date: '2020-11-9 20:21:23 -0300'
category: high-availability
thumbnail: /assets/img/posts/nginx-keepalived.png
permalink: /blog/load-balancer-with-high-availability/
---

Load balancing consists in receiving a request and redirect it to N places, so, how can we do it? 

If we add another layer to handle this job, what happens if that one stop working? How can we mitigate it?

I'll be using two applications for it: [**nginx**](https://www.nginx.com/) for load balancer and [**keepalived**](https://www.keepalived.org/) for high availability.

## First things first, installation time!

Before we jump into configurations, we need to install those resources!

*`This part is specific for centos 7, both resources (keepalived and nginx) are available to all linux distributions.`*

## NGINGX

### Add nginx repository
{% highlight shellscript %}
sudo yum install epel-release
{% endhighlight %}

### Install nginx
{% highlight shellscript %}
sudo yum install nginx
{% endhighlight %}

### Start nginx
{% highlight shellscript %}
sudo systemctl start nginx
{% endhighlight %}

If everything went well, we will be able to see a default page for testing purpose at: *`http://server_ip`*

### Enable nginx to start when system boots
{% highlight shellscript %}
sudo systemctl enable nginx
{% endhighlight %}

## KEEPALIVED

### Install keepalived
{% highlight shellscript %}
sudo yum install keepalived
{% endhighlight %}

### Enable keepalived to start when system boots
{% highlight shellscript %}
sudo systemctl status keepalived
{% endhighlight %}

## Now let the fun begin, configuration time!

Nginx will be responsible for receiving a request and send it to another server, as we are talking about load balancing, it will redirect to at least 2 servers.

We already had up and running at the previous section, so lets jump to configuration.

Nginx default location is **/etc/nginx**. Inside that folder, we want to add our specific configuration file.
***Add a file at the conf.d folder: /etc/nginx/conf.d/my-conf-file.conf***

This file will handle how nginx should work.

>     upstream application_servers {
>         server my_first_servers_ip
>         server my_second_servers_ip        
>     }
>
>     server {
>         listen 80;        
>         listen [::]:80;
>         server_name my_domain.com;
>         location / {
>              proxy_pass http://aplication_servers;
>         }
>     }

The upstream sections, contains our application servers, while the server sections, contains the configuration for the server running the load balancer, itself in this case.

We are telling nginx to get every request at port 80 and redirect it to that couple of servers specified at the upstream section. Each request will be sending to a diferent server, i.e.: > First request goes to the first server, second request to the second, third request to the first again, logic follows.

Once it is configured, we need to restart nginx service to make it effective.

{% highlight shellscript %}
sudo systemctl stop nginx
sudo systemctl start nginx
{% endhighlight %}

Now that we have our load balancer up and running, what happens if it stops working?
Well, for that we have another server running nginx with the configuration. But how this second server knows when to take the lead and start handling those requests? ***The answer lies with keepalived***

**Keepalived** configuration enables a shared IP address between two servers, our application domain will be bound to a virtual IP address and both servers will be listen to this virtual IP.

>     #MAIN SERVER
>     vrrp_instance VI_1 {
>        state MASTER
>        interface eth0
>        virtual_router_id 51
>        priority 255
>        advert_int 1
>        authentication {
>              auth_type PASS
>              auth_pass 12345
>        }
>        virtual_ipaddress {
>              virtual_ip_address
>        }
>     }

>     #BACKUP SERVER
>     vrrp_instance VI_1 {
>        state BACKUP
>        interface eth0
>        virtual_router_id 51
>        priority 254
>        advert_int 1
>        authentication {
>              auth_type PASS
>              auth_pass 12345
>        }
>        virtual_ipaddress {
>              virtual_ip_address
>        }
>     }

There is a very small detail between both server's configuration files, that is the priority and the state of each.

The state MASTER of the first server's configuration makes the it the main responsible for that virtual ip and the BACKUP makes it keep waiting the main falls to assume.

The priority here is the line of responsibility of each server, the higher the number, the first it will assume. If we have 3 servers running as load balancers and 2 as backup server, the one with higher priority will assume, and lowest will wait until that one falls too.