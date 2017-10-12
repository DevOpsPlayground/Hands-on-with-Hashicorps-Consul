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

`docker run -d -P --name=nginx nginx`

or

`docker run -d -P --name=nginx -e "SERVICE_TAGS=webserver" nginx`
`docker run -d -P --name=nginx2 -e "SERVICE_TAGS=webserver" nginx`


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
  least_conn;
  {{range service "nginx"}}server {{.Address}}:{{.Port}} max_fails=3 fail_timeout=60 weight=1;
  {{else}}server 127.0.0.1:65535; # force a 502{{end}}
}

server {
  listen 80 default_server;

  location / {
    proxy_pass http://app;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
  }
}



```

### Render nginx.conf file
```
nohup consul-template  -template "/tmp/nginx.conf.template:/tmp/nginx.conf" &
cat /tmp/nginx.conf
docker stop nginx2
cat /tmp/nginx.conf
docker start nginx2
cat /tmp/nginx.conf
```

### Use nginx.conf
```
docker run --name nginx -v /tmp/nginx.conf:/etc/nginx/nginx.conf:ro -d nginx
docker run --name nginx-lb -v /tmp/nginx.conf:/etc/nginx/nginx.conf -e "SERVICE_TAGS=loadbalancer" -d nginx
```






