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


# AWS
## Pre-reqs
t2.micro, with consul-playground security group

## Become root
`sudo su`

## Install Docker
`apt install -y docker.io`

## Start consul server
### Get your private IP


### Start your consul server
```
docker run -d \
  --name consul --net=host \
  -p 8500:8500 \
  -e 'CONSUL_LOCAL_CONFIG={"skip_leave_on_interrupt": true, "ui": true,  "dns_config": { "allow_stale": false }}' \
  consul agent -server -bind="$(curl -s http://169.254.169.254/latest/meta-data/local-ipv4)" -client=0.0.0.0  -bootstrap
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
Name: {{ keyOrDefault \"playground/server1\" \"server1 name missing\" }}" >> /tmp/index.html.1.template
echo "Nginx 2
Name: {{ keyOrDefault \"playground/server2\" \"server2 name missing\" }}" >> /tmp/index.html.2.template

```

### Download Consul Template and install it

```
wget https://releases.hashicorp.com/consul-template/0.19.3/consul-template_0.19.3_linux_amd64.tgz -O /tmp/consul-template.tar.gz
tar -xvzf /tmp/consul-template.tar.gz -C /bin
```

### Render templates


`nohup consul-template -template "/tmp/index.html.1.template:/tmp/index.html.1:/bin/bash -c 'docker restart nginx || true'" -template "/tmp/index.html.2.template:/tmp/index.html.2:/bin/bash -c 'docker restart nginx || true'" &`

### Look at the two index pages generated
```
cat /tmp/index.html.1
cat /tmp/index.html.2
```

### Start nginx containers

```
docker run -d -P --name=nginx -v /tmp/index.html.1:/usr/share/nginx/html/index.html -e "SERVICE_NAME=webserver" nginx
docker run -d -P --name=nginx2 -v /tmp/index.html.2:/usr/share/nginx/html/index.html -e "SERVICE_NAME=webserver" nginx
```

### Query our two nginx servers
#### Get the ports on which they are accessible
`docker ps -f name=nginx`

This will display your two new nginx containers, and see the port mapping as seen below, where nginx and nginx2 are respectively running on the ports `32768` and `32769`. 
![](./images/nginx-ports.png)

#### Query the servers
`curl localhost:32768`

### Create the keys in the key/value store of Consul
Go to `http://<public IP>:8500/ui/`

1. In the top menu, click on the KEY/VALUE menu item.
2. In the Create Key box, add the `playground/server1` key and set its value to the name you want to give to your first server
3. Click the Create button
4. Repeat steps 2 and 3 for the key `playground/server2`


#### Query the servers again
`curl localhost:32768` and you should see your keys at work!


## Setup Load Balancer

### Setup nginx.conf template

```
cat <<EOT >> /tmp/nginx.conf.template
upstream app {
  {{range service "webserver"}}server {{.Address}}:{{.Port}} max_fails=3 fail_timeout=60 weight=1;
  {{else}}server 127.0.0.1:65535; # force a 502{{end}}
}

server {
  listen 80 default_server;
  resolver 172.16.0.23;
  set \$upstream_endpoint http://app;

  location / {
    proxy_pass \$upstream_endpoint;
    proxy_set_header X-Forwarded-For \$proxy_add_x_forwarded_for;
    proxy_set_header Host \$host;
    proxy_set_header X-Real-IP \$remote_addr;
  }
}
EOT

```

### Render nginx.conf file
```
nohup consul-template -template "/tmp/nginx.conf.template:/tmp/nginx.conf:/bin/bash -c 'docker restart nginx-lb || true'" &
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


Need to automate creation of servers
sudo apt install -y docker.io
sudo docker pull consul
sudo docker pull nginx

security group: 8500, 80
Region: London

need to chance user/passwd