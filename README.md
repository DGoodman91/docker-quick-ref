# Docker Quick Reference

## Building images

---

Build an imagine called myimage using the Dockerfile in the current directory

```
docker build -t myimage .
```

---

## Running containers

---

Run a container and map port 3306 in the container to port 3307 on the Docker host

```
docker run -p 3307:3306 mysql:5.7
```

---

## Useful Containers

---

### nicolaka/netshoot network tools

The [nicolaka/netshoot](https://github.com/nicolaka/netshoot) image is an alpine distribution with a bunch of networking tools. We can deploy a netshoot container and join it to a network to explore, troubleshoot and debug networking issues. Check the github link, but contains stuff like dig, netcat, nmap, tcpdump, ssl_client.

```
docker run -it --network todo-app nicolaka/netshoot
```

Note -it puts us straight into an interactive shell on the container.

---

### alpine/git

We can run an [alpine/git](https://hub.docker.com/r/alpine/git) container to check out a repository, then use the [*docker cp*](https://docs.docker.com/engine/reference/commandline/cp/) tool to copy the repo off the container.

```
docker run --name repo alpine/git clone https://github.com/nicolaka/netshoot.git
docker cp repo:/git/netshoot/ .
```

---