---
layout: post
title: HAPROXY & KeepAlived

---

{{ page.title }}
================

<p class="meta">13 oct 2011 - Los Angeles</p>

HAProxy is load balancer software that allows you to proxy HTTP and TCP connections to a pool of back-end servers; Keepalived – among other uses – allows you to create a redundant pair of HAProxy servers by moving an IP address between HAProxy hosts in an active-passive configuration.

Example Network

For this example, the following services are present, or will be configured here:

192.168.1.34: Syslog server
192.168.1.68: Web backend #1
192.168.1.69: Web backend #2
192.168.1.80: HAProxy host #1 management address
192.168.1.81: HAProxy host #2 management address
192.168.1.84: Load balancer “front end” – the IP address users will ultimately connect to, managed by Keepalived
192.168.1.119: SMTP server
Note that all interfaces are on the same subnet (192.168.1.0/24); the load balancer communicates with the backends on the same interface it uses for client connections.

HAProxy

To install HAProxy on Ubuntu, simply install the “haproxy” package; as described above, this example uses two hosts with IP addresses of 192.168.1.80 and 192.168.1.81. Set “ENABLED=1″ in /etc/default/haproxy to have the init script that comes with the package start HAProxy. HAProxy’s configuration lives at /etc/haproxy/haproxy.cfg:

#### this config needs haproxy-1.1.28 or haproxy-1.2.1
 
global
        # log 127.0.0.1 local0
        # log 127.0.0.1 local1 notice
        log 192.168.1.34 local0
        user haproxy
        group haproxy
        daemon
        maxconn 20000
 
defaults
        log global
        option dontlognull
        balance leastconn
        clitimeout 60000
        srvtimeout 60000
        contimeout 5000
        retries 3
        option redispatch
 
listen stats 192.168.1.80:80 # or 192.168.1.81:80 for second HAProxy host
        mode http
        stats enable
        stats uri /stats
        stats realm HAProxy\ Statistics
        stats auth admin:supersecret
 
listen http 192.168.1.84:80
        mode tcp
        option tcplog
        balance source
        maxconn 10000
        server web01 192.168.1.68:80 maxconn 5000
        server web02 192.168.1.69:80 maxconn 5000
 
 listen https 192.168.1.84:443
        mode tcp
        option tcplog
        balance roundrobin
        maxconn 10000
        server web01 192.168.1.68:443 maxconn 5000
        server web02 192.168.1.69:443 maxconn 5000


 global
        # log 127.0.0.1 local0
        # log 127.0.0.1 local1 notice
        log 192.168.1.34 local0
        user haproxy
        group haproxy
        daemon
        maxconn 20000
 
defaults
        log global
        option dontlognull
        balance leastconn
        clitimeout 60000
        srvtimeout 60000
        contimeout 5000
        retries 3
        option redispatch
 
listen stats 192.168.1.80:80 # or 192.168.1.81:80 for second HAProxy host
        mode http
        stats enable
        stats uri /stats
        stats realm HAProxy\ Statistics
        stats auth admin:supersecret
 
listen http 192.168.1.84:80
        mode tcp
        option tcplog
        balance source
        maxconn 10000
        server web01 192.168.1.68:80 maxconn 5000
        server web02 192.168.1.69:80 maxconn 5000
 
 listen https 192.168.1.84:443
        mode tcp
        option tcplog
        balance roundrobin
        maxconn 10000
        server web01 192.168.1.68:443 maxconn 5000
        server web02 192.168.1.69:443 maxconn 5000
Config Notes
06. Send log messages to the syslog server at 192.168.1.34 using the local0 facility
22. Only stats are provided on this interface – eth0 on the host.
27. Replace “supersecret” with your password for the statistics interface.
29. HTTP listener – 192.168.1.84 is the IP address Keepalived will manage.
30. We are passing HTTP through – not terminating it on – the HAProxy host.
31. In case you’re wondering why we’re not using Keepalived alone, given that we’re not terminating HTTP at HAProxy: The log format options available with HAProxy. (This also gives us the option of changing to HTTP termination in HAProxy later, without installing new software, of course.)
32. Balance based on source address.
34-35. Two HTTP backends.
37. HTTPS listener, again on the Keepalived IP address.
38. HTTPS is also passed through to the web back-ends.
39. See 31 above.
40. Use a round-robin balancing algorithm.
42-43. Two HTTPS backends.

Start HAProxy as follows:

# service haproxy start
Keepalived

Keepalived is installed via the Ubuntu “keepalived” package, intuitively enough. The following entry needs to be made in /etc/sysctl.conf:

net.ipv4.ip_nonlocal_bind=1
(net.ipv4.ip_forward does not need to be set to “1″ for this type of configuration.) Run “sysctl -p” or reboot to apply the sysctl setting.

The keepalived file – /etc/keepalived/keepalived.conf – contains:

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
global_defs {
    notification_email {
        sysadmin@example.com
    }
    notification_email_from keepalived@haproxy01.example.com
    smtp_server 192.168.1.119
    smtp_connect_timeout 30
}
 
vrrp_script chk_haproxy { # Requires keepalived-1.1.13
    script "killall -0 haproxy" # widely used idiom
    interval 2 # check every 2 seconds
    weight 2 # add 2 points of prio if OK
}
 
vrrp_instance VI_1 {
    interface eth0
    state MASTER # or "BACKUP" on backup
    priority 101 # 101 on master, 100 on backup
    virtual_router_id 51
 
    smtp_alert # Activate SMTP notifications
 
    authentication {
        auth_type AH
        auth_pass supersecret
    }
 
    virtual_ipaddress {
        192.168.1.84
    }
 
    track_script {
        chk_haproxy
    }
}
Config Notes
02-07. Configuration for email notifications on master/backup state changes.
10-14. Script to check if HAProxy is still alive. (“killall -0” is a common idiom seen in Keepalived configurations; I’m curious to know the original source.)
18-19. Configure master (or backup).
22. Send SMTP notifications on master/backup state change for this interface.
26. Set authentication password to something better than “supersecret” here.
30. IP address of the virtual interface – in this case, the IP address in front of the load balancer.
34. Use the chk_haproxy script defined above to check whether to fail over this interface or not.

Start the keepalived daemon:

service keepalived start
