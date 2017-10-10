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
  -e 'CONSUL_LOCAL_CONFIG={"skip_leave_on_interrupt": true, "ui": true,  "dns_config": { "allow_stale": false }}' \
  consul agent -server -bind={{GetPrivateIP}} -client=0.0.0.0 -bootstrap
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












