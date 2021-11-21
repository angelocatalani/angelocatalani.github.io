---
layout: post
title: Docker and Kubernetes networking
subtitle: How to reach a container without publishing ports
cover-img: /assets/img/container.png
thumbnail-img: /assets/img/networking.png
share-img: /assets/img/container.png
tags: [docker,kubernetes,networking]
comments: true
---

Reading about Kubernetes services I asked myself why containers are reachable within the pod without publishing any ports.

The answer helped me to clarify some aspects about networking in Docker and Kubenetes.

**TL;DR**  Containers are always assigned a private ip we can use to reach them. Any container in a pod shares the same pod's ip that is equivalent to the afore-mentioned private ip.

**Docker networking**

Docker containers by default are placed inside the virtual private network created by the default Docker network driver: `Bridge`.

This means containers have an IP that is reachable as long as you have access to that VPN.

Supposing we are using Docker Desktop, we can spawn a simple web server that listens on port `80`:
```bash
docker run -d --name webserver-1 nginxdemos/hello
```

If we view its private ip:
```bash
docker exec -it webserver-1  ip addr
```
we get the following output:
```bash
[...]
inet 172.17.0.2/16 brd 172.17.255.255 scope global eth
```

If we were on a Linux machine we could directly ping the server with:
```bash
ping 172.17.0.2
```

Since we are on macOS, the Docker engine is not native and it runs on top of a virtual machine based on Hyper-V. This means we first need to connect to that VM:
```bash
nc -U ~/Library/Containers/com.docker.docker/Data/debug-shell.sock
```
and then we can ping our container:
```bash
ping 172.17.0.2
```

This means we can reach the container without requiring it to publish any port as long as we use its virtual ip. However, we can publish the port of the container to map the private ip and port to a local ip and local port. 

This is how processes and namespaces work.

**Expose vs Publish**

When we specify in the Dockerfile that a port is exposed (`EXPOSE <port>`) it is just for documentation.

We are documenting the ports our container binds to and it is reachable from.

When we publish an ip and port we map a new ip and new port to the ip and port our container has inside its virtual private network.


**Why Kubernetes does not require the coninater to publish its ports**

This is the same reason why we can reach the container once we are connected to the containing node, without the container published any port.

And if we have the Kubenetes node running locally (e.g, inside Docker Desktop), we can reach the container inside the pod using its private ip (on a macOS we need first to connect to the VM).

For example, we can spawn a pod with that simple web server:
```bash
kubectl run webserver-2 --image=nginxdemos/hello
```
.
We can find the ip of that pod:
```bash
kubectl get pods -o wide
```
that is: `10.1.0.30`.

We can note it has a different subnet of the previous one since we are using the Kubernetes driver to create the VPN to host the pod.

However, the container inside that pod has the same ip of its pod:
```bash
docker ps
docker exec -it ecb8224445cb  ip addr

```

And this ip is the same private ip the container has when we run the container using Docker with the default network driver: Bridge. 

Finally, we can ping that container (or equally pod):
```bash
nc -U ~/Library/Containers/com.docker.docker/Data/debug-shell.sock
ping 10.1.0.30

```

If we have a pod with multiple containers, the containers share the same ip of the node but they must bind to different ports.


**References**:

- [StackOverflow: port publishing from Docker to Kubenetes](https://stackoverflow.com/questions/58327009/how-does-port-publishing-from-a-docker-container-to-a-kubernetes-pod-work#58327283)
- [Access to the macOS Docker VM](https://gist.github.com/BretFisher/5e1a0c7bcca4c735e716abf62afad389)

Have a nice day 🚀 

![My new bike](/assets/img/my-bike.png)

