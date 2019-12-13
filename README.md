### HA Proxy config for knox LB :

--> Create SSL cert for haproxy :
```
$ sudo openssl genrsa -out /etc/haproxy/haproxy.key 2048
$ sudo openssl req -new -key /etc/haproxy/haproxy.key -out /etc/haproxy/haproxy.csr -subj "/C=US/ST=North Carolina/L=Raleigh/O=HWX/OU=Support/CN=$(hostname -f)"
$ sudo openssl x509 -req -days 365 -in /etc/haproxy/haproxy.csr -signkey /etc/haproxy/haproxy.key -out /etc/haproxy/haproxy.crt 
$ sudo cat /etc/haproxy/haproxy.key /etc/haproxy/haproxy.crt > /etc/haproxy/haproxy.pem
```
--> Configure HA proxy to use ssl certs and port 443 to loadbalance knox hosts with persistent cookie connection:
```
#vi /etc/haproxy/haproxy.cfg
global
 log 127.0.0.1 local0 notice
 maxconn 2000
 user haproxy
 group haproxy
defaults
 log     global
 mode    http
 option  httplog
 option  dontlognull
 retries 3
 option redispatch
 timeout connect  5000
 timeout client  10000
 timeout server  10000
frontend knox_ssl
    bind *:80
    bind *:443 ssl crt /etc/haproxy/haproxy.pem
    default_backend knox_servers
backend knox_servers
 mode http
 balance roundrobin
 cookie SERVERID insert indirect nocache
 server knox1 saurabh-2.example.com:8443 ssl verify none cookie knox1
 server knox2 saurabh-3.example.com:8443 ssl verify none cookie knox2
 http-request set-header X-Forwarded-Port %[dst_port]
 http-request add-header X-Forwarded-Proto https if { ssl_fc }
```
