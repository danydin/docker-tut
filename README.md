explain namespaces, how it works on mac and windows (linuxkit) and why are there multiple linux namespaces? mnt vs pid vs net vs ipc vs user vs utc

cgroups / control groups in containers

An image is a TAR of a file system, and a container is a file system plus a set of processes running in isolation.

ps -ef

docker container exec -it containerID bash root@containerID:/#

docker container run -t ubuntu top

docker container run --detach --publish 8080:80 --name nginx nginx

docker pull mongo vs docker container run mongo

docker container ls

who's the host in a docker container

docker container stop [container id / name] (You need to enter only enough digits of the ID to be unique.)

docker system prune

dangling, volumes and networks docker

echo 'bla bla' > app.py VS >>


* Containers are composed of Linux namespaces and control groups that provide isolation from other containers and the host, which means you can avoid potential conflicts between containers with different system or runtime dependencies. For example, you  can deploy an app that uses Java 7 and another app that uses Java 8 on the same host. Or you can run multiple NGINX containers that all have port 80 as their default listening ports. (If you're exposing on the host by using the --publish flag, the ports selected for the host must be unique.) Isolation benefits are possible because of Linux namespaces. 
Because of the isolation properties of containers, you can schedule many containers on a single host without worrying about conflicting dependencies. This makes it easier to run multiple containers on a single host, using all resources allocated to that single host and ultimately saving a lot of servers costs. 
Remember: You didn't have to install anything on your host (other than Docker) to run these processes / programs as each container includes the dependencies that it needs within the container, so you don't need to install anything on your host directly.
you should avoid using unverified content from the Docker Store when developing your own images because these images might contain security vulnerabilities or possibly even malicious software.
An image is the blueprint for spinning up containers.


