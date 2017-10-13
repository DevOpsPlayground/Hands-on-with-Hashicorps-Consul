# DevOps Playground #15: All Hands on Consul!
## Introduction
During this meetup, we will go through these steps:

1. [Start a Consul Server](#start-a-consul-server)
2. Configure and start Registrator
3. Create Services


# Background
t
t
t
t
t
t
t
t
t
tt


t

t

t

t

t

t

t

t

t

t
t
t
## Consul


t

t

t

t

t

t

## Registrator


t

t

t

t

t

t

## Nginx


t

t

t

t

t

t

## Load Balancer


t

t

t

t

t

t

# Hands-On!
## Start a Consul Server



```
docker run -d \
  --name consul --net=host \
  -p 8500:8500 \
  -e 'CONSUL_LOCAL_CONFIG={"skip_leave_on_interrupt": true, "ui": true,  "dns_config": { "allow_stale": false }}' \
  consul agent -server -bind={{GetPrivateIP}} -client=0.0.0.0  -bootstrap
```


## Start registrator container:

```bash
docker run -d \
    --name=registrator \
    --net=host \
    --volume=/var/run/docker.sock:/tmp/docker.sock \
    gliderlabs/registrator:latest \
      consul://localhost:8500
```

`docker run -d -P --name=nginx nginx`

In the consul server container:
`apk add jq --no-cache`


# AWS
## Install Docker
`apt install docker.io`

## Start consul server
```
docker run -d \
  --name consul --net=host \
  -p 8500:8500 \
  -e 'CONSUL_LOCAL_CONFIG={"skip_leave_on_interrupt": true, "ui": true,  "dns_config": { "allow_stale": false }}' \
  consul agent -server -bind=<PRIVATE IP> -client=0.0.0.0  -bootstrap
```

## Start registrator
```bash
docker run -d \
    --name=registrator \
    --net=host \
    --volume=/var/run/docker.sock:/tmp/docker.sock \
    gliderlabs/registrator:latest \
      consul://localhost:8500
```



## Start two nginx servers

### Create two index pages

```
echo "Nginx 1
Name: {{ keyOrDefault \"playground/test\" \"name missing\" }}" >> /tmp/index.html.1.template
echo "Nginx 2
Name: {{ keyOrDefault \"playground/test2\" \"name missing\" }}" >> /tmp/index.html.2.template

```

### Render templates

`nohup consul-template -template "/tmp/index.html.1.template:/tmp/index.html.1:docker restart nginx || true" -template "/tmp/index.html.2.template:/tmp/index.html.2:docker restart nginx2 || true" &`

### Start nginx containers

```
docker run -d -P --name=nginx -v /tmp/index.html.1:/usr/share/nginx/html/index.html -e "SERVICE_NAME=webserver" nginx
docker run -d -P --name=nginx2 -v /tmp/index.html.2:/usr/share/nginx/html/index.html -e "SERVICE_NAME=webserver" nginx
```


## Setup Load Balancer
### Downnload Consul Template and install it

```
wget https://releases.hashicorp.com/consul-template/0.19.3/consul-template_0.19.3_linux_amd64.tgz -O /tmp/consul-template.tar.gz
tar -xvzf /tmp/consul-template.tar.gz -C /bin

```
### Setup nginx.conf template

```

vi /tmp/nginx.conf.template
upstream app {
  {{range service "webserver"}}server {{.Address}}:{{.Port}} max_fails=3 fail_timeout=60 weight=1;
  {{else}}server 127.0.0.1:65535; # force a 502{{end}}
}

server {
  listen 80 default_server;
  resolver 172.16.0.23;
  set $upstream_endpoint http://app;

  location / {
    proxy_pass $upstream_endpoint;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
  }
}



```

### Render nginx.conf file
```
nohup consul-template -template "/tmp/nginx.conf.template:/tmp/nginx.conf:docker restart nginx-lb || true" &
cat /tmp/nginx.conf
docker stop nginx2
cat /tmp/nginx.conf
docker start nginx2
cat /tmp/nginx.conf
```

### Use nginx.conf
```
docker run -p 80:80 --name nginx-lb \
  -v /tmp/nginx.conf:/etc/nginx/conf.d/default.conf \
  -e "SERVICE_TAGS=loadbalancer" -d \
  nginx
```

need to change other nginx containers of course

Need to create keys in Consul

security group: 8500, 80


