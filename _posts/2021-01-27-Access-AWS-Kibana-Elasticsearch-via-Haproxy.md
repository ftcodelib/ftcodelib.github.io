---
layout: post
title: Access AWS Elasticsearch Kibana UI via Haproxy + AWS ALB
published: true
tags: AWS Elasticsearch Kibana HAproxy
---

If you deploy the AWS Elasticsearch into private network, you might also want the Kibana UI to be reachable on the public network.

There are few ways to archive this. You use either via SSH tunnel or a proxy server such as NGINX and HAproxy.

In this example, we will use HAProxy for us to access the Kibana UI.

## Steps to configure

First, install the HAProxy into your OS.

```
# For CentOS / Amazon Linux
sudo yum install haproxy -y

# For Ubuntu
sudo apt install haproxy -y
```

Once the haproxy installed cd to /etc/haproxy. Follow the following command:

```
cd /etc/haproxy
mv haproxy.cfg haproxy.cfg.ori
vi haproxy.cfg
```

Once you are in the editor, you may follow below configuration. Go to the server* line and change to your Elasticsearch VPC Endpoint accordingly.

```
global
    log         127.0.0.1 local2

    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    maxconn     4000
    user        haproxy
    group       haproxy
    daemon

    # turn on stats unix socket
    stats socket /var/lib/haproxy/stats

defaults
    timeout connect 5000ms
    timeout client 50000ms
    timeout server 50000ms

frontend ft_http
        bind :80
        mode http
        default_backend bk_http

frontend ft_https
        bind :443
        mode tcp
        default_backend bk_https

backend bk_http
        mode http
        balance roundrobin
        stick on src table bk_https
        default-server inter 1s
        server s1 vpc-<es_name><random_num>.<region>.es.amazonaws.com:80 check id 1

backend bk_https
        mode tcp
        balance roundrobin
        stick-table type ip size 200k expire 30m
        stick on src
        default-server inter 1s
        server s1 vpc-<es_name><random_num>.<region>.es.amazonaws.com:443 check id 1
```

Once done you may start the service.

```
systemctl enable haproxy
systemctl start haproxy
```

Once done you may try test the URL by curl the rest api.

```
curl -X GET https://localhost/_cat/health?v
```

Next, you can add your haproxy server into the AWS ALB with SSL termination. The ALB target group health check port you can add 200 and 302. The path would be "/".
