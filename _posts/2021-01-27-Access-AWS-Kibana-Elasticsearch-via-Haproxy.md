---
layout: post
title: Access AWS Elasticsearch Kibana UI via Haproxy + AWS ALB
published: false
tags: test markdown blog
---

##


```haproxy
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
