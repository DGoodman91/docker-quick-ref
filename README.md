# Docker Quick Reference

## Image Basics

Build an imagine called myimage using the Dockerfile in the current directory.

```
docker build -t myimage .
```


List all images.
```
docker image ls
```

---

## Container Basics

Run a container in detached mode and map port 3306 in the container to port 3307 on the Docker host. Use the *name* argument to name it, else it'll be given a random name by the docker service.
```
docker run --name my-db -d -p 3307:3306 mysql:5.7
```
Can use the *-e* argument to specify environment variables in the container.
```
docker run -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=password -e MYSQL_DATABASE=myfirstdb mysql:5.7
```


List all containers, all states.
```
docker ps -a
```
Stop and remove a container by id using the force flag.
```
docker rm -f bb41eae63413
```

Tail the container logs by id.
```
docker logs -f bb41eae63413
```

Execute commands inside the container specified by the id. In this case we connect to a db container and run the mysql command line tool.
```
docker exec -it dfda05c2c20b mysql -p password
```

---

## Volumes

### Named volumes

Create a new named volume.
```
docker volume create my-named-volume
```
When running a container, mount the named volume *my-named-volume* into the container at */var/lib/mysql*.
```
docker run -dp 3306:3306 -v my-named-volume:/var/lib/mysql mysql:5.7
```
Get details of the volume including the created date and the location on the docker host's filesystem
```json
docker volume inspect my-named-volume

[
    {
        "CreatedAt": "2021-05-18T23:42:36Z",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/todo-db/_data",
        "Name": "my-named-volume",
        "Options": {},
        "Scope": "local"
    }
]
```
Clean up volumes not being used by any containers.
```
docker volume prune
```

### Bind mounts

Can use bind mounts to mount a portion of the host filesystem into the container. Useful for development.
```
docker run -dit -v C:/Users/david/Documents/docker/folder:/directory/in/container my-app
```

---

## Networks

Create a new network.
```
docker network create my-network
```
Run a container and join it onto the network.
```
docker run -d --network my-network --network-alias my-networked-db mysql:5.7
```

---

## Useful Containers

### nicolaka/netshoot network tools

The [nicolaka/netshoot](https://github.com/nicolaka/netshoot) image is an alpine distribution with a bunch of networking tools. We can deploy a netshoot container and join it to a network to explore, troubleshoot and debug networking issues. Check the github link, but contains stuff like dig, netcat, nmap, tcpdump, ssl_client.

```
docker run -it --network todo-app nicolaka/netshoot
```

Note -it puts us straight into an interactive shell on the container.

### alpine/git

We can run an [alpine/git](https://hub.docker.com/r/alpine/git) container to check out a repository, then use the [*docker cp*](https://docs.docker.com/engine/reference/commandline/cp/) tool to copy the repo out of the container.

```
docker run --name repo alpine/git clone https://github.com/nicolaka/netshoot.git
docker cp repo:/git/netshoot/ .
```

### httpd

Can use the [official docker httpd](https://hub.docker.com/_/httpd) image to quickly spin up a website by bind-mounting the host's filesystem
```
docker run -d -p 8080:80 -v C:/Users/david/Documents/docker/simple-webapp:/usr/local/apache2/htdocs/ httpd:2.4
```

### mysql

Use the [official docker mysql](https://hub.docker.com/_/mysql) image to get a database up and running rapidly. The named volume will be auto-created if it doesn't exist.
```
docker run -d -p 3306:3306 -v my-mysql-data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=password -e MYSQL_DATABASE=dbname mysql:5.7
```