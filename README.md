
# Repurposing:
I needed something to automatically sync hosts file with the docker containers so if I need to run a service without docker it can still use the same configuration (host names) that are used if run in docker

example:
service-a connects to db-1

in docker the db host will be db1 not local host 
if I want to avoid changing the configuration when I run service-a without docker then it should access db-1 using localhost:port
so to do that I used this tool to automatically add an entry for db-1 in /etc/hosts like this:

127.0.0.1    db-1

so i don't need to change the configs

the original author used the ip address of the docker contianer which meant I had to use the container port not the published one, which doesn't work for me since I need to use the actual port that will be accessiable by the outside world, so I changed one line to always write the host ip as localhost

pushed this to my docker hub :
- clone
- build: 
```docker build -t basharlabadi/docker-etc-hosts-sync ```

- run
```
    docker run -d \
    --name hoster \
    -v /var/run/docker.sock:/tmp/docker.sock \
    -v /etc/hosts:/tmp/hosts \
    basharlabadi/docker-etc-hosts-sync
```


--------------------------------
# Hoster

A simple "etc/hosts" file injection tool to resolve names of local Docker containers on the host.

hoster is intended to run in a Docker container:

    docker run -d \
        -v /var/run/docker.sock:/tmp/docker.sock \
        -v /etc/hosts:/tmp/hosts \
        dvdarias/docker-hoster

The `docker.sock` is mounted to allow hoster to listen for Docker events and automatically register containers IP.

Hoster inserts into the host's `/etc/hosts` file an entry per running container and keeps the file updated with any started/stoped container.

## Container Registration

Hoster provides by default the entries `<container name>, <hostname>, <container id>` for each container and the aliases for each network. Containers are automatically registered when they start, and removed when they die.

For example, the following container would be available via DNS as `myname`, `myhostname`, `et54rfgt567` and `myserver.com`:

    docker run -d \
        --name myname \
        --hostname myhostname \
        --network somenetwork --network-alias "myserver.com" \
        mycontainer

If you need more features like **systemd interation** and **dns forwarding** please check [resolvable](https://hub.docker.com/r/mgood/resolvable/)

Any contribution is, of course, welcome. :)
