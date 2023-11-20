explain namespaces, how it works on mac and windows (linuxkit) and why are there multiple linux namespaces? mnt vs pid vs net vs ipc vs user vs utc

cgroups / control groups in containers

An image is a TAR of a file system, and a container is a file system plus a set of processes running in isolation.

ps -ef

docker container exec -it containerID bash root@containerID:/#

docker run command

-t ubuntu top

--detach --publish 8080:80 --name nginx nginx

docker pull mongo vs docker container run mongo

docker container ls

docker image ls

who's the host in a docker container
the computer/kernel the docker run ons is the host

docker container stop [container id / name] (You need to enter only enough digits of the ID to be unique.)

docker system prune

dangling, volumes and networks docker

build --platform & buildx build --platform & docker buildx create --use

Docker Compose

docker container logs [container id] 

echo 'bla bla' > app.py VS >>

Docker Swarm 

what's a dockerfile
A Dockerfile is basically a text document that contains all the commands a user could call on the command line to assemble an image.

explain tags in docker image

What's the difference between "Shared" and "Simple" tags in docker images?

docker tag python-hello-world [dockerhub username]/python-hello-world

docker login

docker push [image name]

docker diff

image history 

RUN pip install flask
The RUN command executes commands needed to set up your image for your application, such as installing packages, editing files, or changing file permissions. In this case, you are installing Flask. The RUN commands are executed at build time and are added to the layers of your image.

CMD ["python","app.py"]
CMD is the command that is executed when you start a container. Here, you are using CMD to run your Python applcation.
There can be only one CMD per Dockerfile. If you specify more than one CMD, then the last CMD will take effect. The parent python:3.6.1-alpine also specifies a CMD (CMD python2). You can look at the Dockerfile for the official python:alpine image.
You can use the official Python image directly to run Python scripts without installing Python on your host. However, in this case, you are creating a custom image to include your source so that you can build an image with your application and ship it to other environments.

COPY app.py /app.py
This line copies the app.py file in the local directory (where you will run docker image build) into a new layer of the image. This instruction is the last line in the Dockerfile. Layers that change frequently, such as copying source code into the image, should be placed near the bottom of the file to take full advantage of the Docker layer cache. This allows you to avoid rebuilding layers that could otherwise be cached. For instance, if there was a change in the FROM instruction, it will invalidate the cache for all subsequent layers of this image. You'll see this little later in this lab.
It seems counter-intuitive to put this line after the CMD ["python","app.py"] line. Remember, the CMD line is executed only when the container is started, so you won't get a file not found error here.
And there you have it: a very simple Dockerfile. See the full list of commands that you can put into a Dockerfile. Now that you've defined the Dockerfile, you'll use it to build your custom docker image.

vi Dockerfile

:wq

 the union file system & copy-on-write

docker run -p 5001:5000 -d python-hello-world
The -p flag maps a port running inside the container to your host. In this case, you're mapping the Python app running on port 5000 inside the container to port 5001 on your host. Note that if port 5001 is already being used by another application on your host, you might need to replace 5001 with another value, such as 5002. -d XXXXX

docker stop $(docker ps -q)
docker ps -q: This command lists the IDs of all running Docker containers. The -q option stands for "quiet" and it makes the command only output the container IDs, without any additional information
docker stop: This command stops one or more running Docker containers. You can pass one or more container IDs or names to this command to stop them docs.docker.com.
$(...): This is a command substitution in bash. It runs the command inside the parentheses and replaces the command substitution with the output of the command. In this case, it runs docker ps -q and replaces $(docker ps -q) with the list of container IDs
Please note that this command will not stop containers that are not currently running. If you want to stop all containers, including those that are not currently running, you can use the following command:
docker stop $(docker ps -a -q)
docker ps -a -q lists the IDs of all containers, not just the running ones

Why Docker images are built for specific operating systems
Platform Compatibility: Docker images are designed to be platform-specific to ensure compatibility with the host operating system. For example, a Docker image built for a Linux system won't run on a Windows system without an emulation layer. Similarly, a Docker image built for an ARM architecture won't run on an x86 architecture without an emulation layer docs.docker.com.
Performance: Docker images are optimized for the specific architecture and operating system they're built for. This can lead to better performance compared to running the image on a different platform docs.docker.com.
Security: Docker images are isolated from the host system, which reduces the attack surface. This isolation is specific to the host operating system and can't be achieved if the Docker image is built for a different operating system docs.docker.com.
Consistency: Docker images provide a consistent environment for running applications. This is crucial for testing and debugging. If the Docker image is built for a different operating system, the behavior of the application might be different, leading to inconsistencies stackify.com.
Efficiency: Docker images are lightweight and take up minimal disk space. This makes it easy to distribute and deploy applications quickly. If the Docker image is built for a different operating system, it might not be as efficient stackify.com.
Building Docker images for all operating systems at once would not only be impractical due to the vast number of potential operating systems and configurations, but it could also introduce unnecessary complexity and potential compatibility issues. By building Docker images for specific operating systems, developers can ensure that their applications work correctly in those operating systems without needing to support a multitude of different configurations

What's the purpose of the FROM line in a dockerfile

## create & build images & containers
you can build an image for both linux/amd64 and linux/arm64 using the following command:
docker buildx build --platform linux/amd64,linux/arm64 .

Q: image VS container
A:

## stop & remove containers
docker container ls - Get a list of the containers running by running the command docker 
docker container stop [container id] - for each container in the list that is running (as long as id is unique it wil remove even with jsut 2 digits and can use space to inc multiple ids)
Remove the stopped containers by running docker system prune:

## orchestrating an application for production
docker swarm init --advertise-addr eth0 (You can think of Docker Swarm as a special mode that is activated by the command: docker swarm init. The --advertise-addr option specifies the address in which the other nodes will use to join the swarm). This docker swarm init command generates a join token. The token makes sure that no malicious nodes join the swarm. You need to use this token to join the other nodes to the swarm. For convenience, the output includes the full command docker swarm join, which you can just copy/paste to the other nodes.
On both node2 and node3, copy and run the docker swarm join command that was outputted to your console by the last command.
Back on node1, run docker node ls to verify your three-node cluster: This command outputs the three nodes in your swarm. The asterisk (*) next to the ID of the node represents the node that handled that specific command (docker node ls in this case). In a Docker swarm, there is a manager node and several worker nodes. for example in commands above Your node consists of one manager node and two workers nodes. Managers handle commands and manage the state of the swarm. Workers cannot handle commands and are simply used to run containers at scale. By default, managers are also used to run containers. All docker service commands for the rest of this lab need to be executed on the manager node (Node1). By default, you control the swarm directly from the manager node. This means that you need to be on the manager node to run Docker commands that control the swarm. . however you might want to control a Docker swarm that is running on a remote server, or you might want to control a Docker swarm from your local machine.
 To control a Docker swarm remotely, you can connect to the Docker Engine of the manager node using the remote API. The remote API is a REST API that allows you to interact with the Docker Engine remotely. You can use the remote API to run Docker commands that control the swarm, just like you would if you were on the manager node. Alternatively, you can activate a remote host from your local Docker installation by setting the $DOCKER_HOST and $DOCKER_CERT_PATH environment variables. The $DOCKER_HOST environment variable specifies the address of the Docker Engine, and the $DOCKER_CERT_PATH environment variable specifies the path to the certificates that are used to secure the connection to the Docker Engine. By controlling a Docker swarm remotely, you can manage your Docker applications without having to SSH into the production servers. This can make managing your Docker applications more efficient and secure. 
 To control a Docker swarm remotely using the remote API, you need to set up the Docker Engine on the manager node to accept remote connections. Here's how you can do it:
Configure Docker Engine: You need to modify the Docker Engine's configuration to accept remote connections. This can be done by editing the daemon.json file, which is located at /etc/docker on Ubuntu machines. The configuration should look something like this:
{
 "hosts": ["tcp://0.0.0.0:2375", "unix:///var/run/docker.sock"]
}
This configuration tells the Docker Engine to accept connections on port 2375. However, this configuration is unsecured and should not be used if the server is publicly hosted. For a secured connection, you can use the following configuration:

{
 "tls": true,
 "tlscert": "/var/docker/server.pem",
 "tlskey": "/var/docker/serverkey.pem",
 "hosts": ["tcp://x.x.x.y:2376", "unix:///var/run/docker.sock"]
}
In this configuration, x.x.x.y should be replaced with the IP address of the Docker Engine, and /var/docker/server.pem and /var/docker/serverkey.pem should be replaced with the paths to the certificate and key files that are used to secure the connection to the Docker Engine 

To run containers on a Docker Swarm, you need to create a service. A service is an abstraction that represents multiple containers of the same image deployed across a distributed cluster.
* docker service create --detach=true --name nginx1 --publish 80:80  --mount source=/etc/hostname,target=/usr/share/nginx/html/index.html,type=bind,ro nginx:1.12 XXX-NODEMANAGERKEY
This command statement is declarative, and Docker Swarm will try to maintain the state declared in this command unless explicitly changed by another docker service command. This behavior is useful when nodes go down, so it is being replaced instantly by antoher node to maintain the declartive state. The --mount flag is useful to have NGINX print out the hostname of the node it's running on. You will use this later in this lab when you start load balancing between multiple containers of NGINX that are distributed across different nodes in the cluster and you want to see which node in the swarm is serving the request. You are using NGINX tag 1.12 in this command. You will see a rolling update with version 1.13 later in this lab. The --publish command uses the swarm's built-in routing mesh. In this case, port 80 is exposed on every node in the swarm. The routing mesh will route a request coming in on port 80 to one of the nodes running the container.
* docker service ls - to inspect the service you just created
* docker service ps nginx1 - Check the running container of the service.  To take a deeper look at the running tasks. A task is another abstraction in Docker Swarm that represents the running instances of a service. In this case, there is a 1-1 mapping between a task and a container.
* curl localhost:80 - Curling will output the hostname where the container is running. For this example, it is running on node1, but yours might be different. Because of the routing mesh, you can send a request to any node of the swarm on port 80. This request will be automatically routed to the one node that is running the NGINX container.
* docker service update --replicas=5 --detach=true nginx1 - In production, you might need to handle large amounts of traffic to your application, so you'll learn how to scale. Update your service with an updated number of replicas. using the docker service command to update the NGINX service that you created previously to include 5 replicas. This is defining a new state for the service. When this command is run, the following events occur:  The state of the service is updated to 5 replicas, which is stored in the swarm's internal storage. Docker Swarm recognizes that the number of replicas that is scheduled now does not match the declared state of 5. Docker Swarm schedules 5 more tasks (containers) in an attempt to meet the declared state for the service. 
* docker service ps nginx1 - After a few seconds, you should see that the swarm did its job and successfully started 4 more containers. Notice that the containers are scheduled across all three nodes of the cluster. The default placement strategy that is used to decide where new containers are to be run is the emptiest node, but that can be changed based on your needs.
* curl localhost:80 - run again this mulitple times to see how the nodes are being load balance between different nodes automatically. The --publish 80:80 parameter is still in effect for this service; that was not changed when you ran the docker service update command. However, now when you send requests on port 80, the routing mesh has multiple containers in which to route requests to. The routing mesh acts as a load balancer for these containers, alternating where it routes requests to. You should see which node is serving each request because of the useful --mount command you used earlier. Limits of the routing mesh: The routing mesh can publish only one service on port 80. If you want multiple services exposed on port 80, you can use an external application load balancer outside of the swarm to accomplish this.
* docker service logs nginx1 - Another easy way to see which nodes those requests were routed to is to check the aggregated logs. You can get aggregated logs for the service by using the command docker service logs [service name]. This aggregates the output from every running container, that is, the output from docker container logs [container name]. In addition to seeing whether the request was sent to node1, node2, or node3, you can also see which container on each node that it was sent to. For example, nginx1.5 means that request was sent to a container with that same name as indicated in the output of the command used earlier: docker service ps nginx1.
*  docker service update --image nginx:1.13 --detach=true nginx1 - This triggers a rolling update of the swarm. Quickly enter the command docker service ps nginx1 over and over to see the updates in real time. --update-parallelism: specifies the number of containers to update immediately (defaults to 1).
--update-delay: specifies the delay between finishing updating a set of containers before moving on to the next set. after a few seconds run the command docker service ps nginx1 to see all the images that have been updated to nginx:1.13.
* docker service create --detach=true --name nginx2 --replicas=5 --publish 81:80  --mount source=/etc/hostname,target=/usr/share/nginx/html/index.html,type=bind,ro nginx:1.12
MAIN-NODE-KEY - this create a new service by copying the following line. Change the name and the publish port to avoid conflicts with your existing service. Also, add the --replicas option to scale the service with five instances
* watch -n 1 docker service ps nginx2 - On node1, use the watch utility to watch the update from the output of the docker service ps command. Tip: watch is a Linux utility and might not be available on other operating systems.
* docker swarm leave - This is the typical way to leave the swarm, but you can also kill the node and the behavior will be the same. Click node1 to watch the reconciliation in action. You should see that the swarm attempts to get back to the declared state by rescheduling the containers that were running on node3 to node1 and node2 automatically. The manager node contains the necessary information to manage the cluster, but if this node goes down, the cluster will cease to function. For a production application, you should provision a cluster with multiple manager nodes to allow for manager node failures.  You should have at least three manager nodes but typically no more than seven. Manager nodes implement the raft consensus algorithm, which requires that more than 50% of the nodes agree on the state that is being stored for the cluster. If you don't achieve more than 50% agreement, the swarm will cease to operate correctly. For this reason, note the following guidance for node failure tolerance:  Three manager nodes tolerate one node failure. Five manager nodes tolerate two node failures. Seven manager nodes tolerate three node failures. It is possible to have an even number of manager nodes, but it adds no value in terms of the number of node failures. For example, four manager nodes will tolerate only one node failure, which is the same tolerance as a three-manager node cluster. However, the more manager nodes you have, the harder it is to achieve a consensus on the state of a cluster.  While you typically want to limit the number of manager nodes to no more than seven, you can scale the number of worker nodes much higher than that. Worker nodes can scale up into the thousands of nodes. Worker nodes communicate by using the gossip protocol, which is optimized to be perform well under a lot of traffic and a large number of nodes. NEED FURTHER CLAIRFACTION REGARLDING THE consensus OF NODES MANAGER regarding the state of a node. Docker Swarm schedules services by using a declarative language. You declare the state, and the swarm attempts to maintain and reconcile to make sure the actual state equals the desired state. Docker Swarm is composed of manager and worker nodes. Only managers can maintain the state of the swarm and accept commands to modify it. Workers have high scalability and are only used to run containers. By default, managers can also run containers. The routing mesh built into Docker Swarm means that any port that is published at the service level will be exposed on every node in the swarm. Requests to a published service port will be automatically routed to a container of the service that is running in the swarm. You can use other tools to help solve problems with orchestrated, containerized applications in production, including Docker Swarm and the IBM Cloud Kubernetes Service. Open-source container orchestration platforms, such as Docker Swarm and Kubernetes, provide what to their users? A platform to help solve problems of running distributed containerized applications in production, such as high availability, scaling, fault tolerance, and scheduling
* docker exec [OPTIONS] CONTAINER COMMAND [ARG...] - The docker exec command is used to run a command in a running Docker container. It allows you to interact with the container's shell or run specific commands within the container. OPTIONS: These are optional flags that you can use with the docker exec command. Some common options include -d (detached mode), -i (interactive mode), and -t (allocate a pseudo-TTY).
CONTAINER: This is the ID or name of the container where you want to run the command.
COMMAND: This is the command that you want to run inside the container.
ARG: These are the arguments that you want to pass to the command.
For example, if you want to get a bash shell inside a running container, you can use the following command: docker exec -it <container_id> /bin/bash
In this command, -it is a combination of two options: -i (interactive mode) and -t (allocate a pseudo-TTY). This allows you to interact with the bash shell inside the container. <container_id> is the ID or name of the container where you want to get the bash shell. /bin/bash is the command that starts the bash shell. Please note that the docker exec command can only be used with running containers. If the container is not running, you need to start it first using the docker start command.
* TIP: Scheduling containers across a distributed cluster: In a production environment, you often need to distribute your containers across multiple machines to ensure high availability and to handle increased load. This requires a container orchestration tool like Kubernetes or Docker Swarm. High availability: In a production environment, you need to ensure that your application is always available. This means that if a container fails, it should be automatically restarted on another machine. This can be achieved using the restart policies in Docker or the liveness and readiness probes in Kubernetes. Scaling: In a production environment, you need to be able to handle increased load by scaling up your application. This means starting more containers when the load increases. This can be achieved using the scaling features in Docker Swarm or Kubernetes.



















* Containers are composed of Linux namespaces and control groups that provide isolation from other containers and the host, which means you can avoid potential conflicts between containers with different system or runtime dependencies. For example, you  can deploy an app that uses Java 7 and another app that uses Java 8 on the same host. Or you can run multiple NGINX containers that all have port 80 as their default listening ports. (If you're exposing on the host by using the --publish flag, the ports selected for the host must be unique.) Isolation benefits are possible because of Linux namespaces. 
Because of the isolation properties of containers, you can schedule many containers on a single host without worrying about conflicting dependencies. This makes it easier to run multiple containers on a single host, using all resources allocated to that single host and ultimately saving a lot of servers costs. 
Remember: You didn't have to install anything on your host (other than Docker) to run these processes / programs as each container includes the dependencies that it needs within the container, so you don't need to install anything on your host directly.
you should avoid using unverified content from the Docker Store when developing your own images because these images might contain security vulnerabilities or possibly even malicious software.
An image is the blueprint for spinning up containers.
This is the starting point for your Dockerfile. Every Dockerfile typically starts with a FROM line that is the starting image to build your layers on top of. In this case, you are selecting the python:3.6.1-alpine base layer because it already has the version of Python and pip that you need to run your application. The alpine version means that it uses the alpine distribution, which is significantly smaller than an alternative flavor of Linux. A smaller image means it will download (deploy) much faster, and it is also more secure because it has a smaller attack surface.
Look at the available tags for the official Python image on the Docker Hub. It is best practice to use a specific tag when inheriting a parent image so that changes to the parent dependency are controlled. If no tag is specified, the latest tag takes effect, which acts as a dynamic pointer that points to the latest version of an image.
For security reasons, you must understand the layers that you build your docker image on top of. For that reason, it is highly recommended to only use official images found in the Docker Hub, or noncommunity images found in the Docker Store. 
For a more complex application, you might need to use a FROM image that is higher up the chain. For example, the parent Dockerfile for your Python application starts with FROM alpine, then specifies a series of CMD and RUN commands for the image. If you needed more control, you could start with FROM alpine (or a different distribution) and run those steps yourself. However, to start, it's recommended that you use an official image that closely matches your needs.
The Dockerfile is used to create reproducible builds for your application. A common workflow is to have your CI/CD automation run docker image build as part of its build process. After images are built, they will be sent to a central registry where they can be accessed by all environments (such as a test environment) that need to run instances of that application. In the next section, you will push your custom image to the public Docker registry, which is the Docker Hub, where it can be consumed by other developers and operators.
Docker images contain all the dependencies that they need to run an application within the image. This is useful because you no longer need to worry about environment drift (version differences) when you rely on dependencies that are installed on every environment you deploy to. You also don't need to follow more steps to provision these environments. Just one step: install docker, and that's it.
There is a caching mechanism in place for building images and pushing (to register such as docker hub), where it only pushes the one layer that has changed. When you change a layer, every layer built on top of that will have to be rebuilt. Each line in a Dockerfile builds a new layer that is built on the layer created from the lines before it. This is why the order of the lines in your Dockerfile is important. You optimized your Dockerfile so that the layer that is most likely to change (COPY app.py /app.py) is the last line of the Dockerfile. Generally for an application, your code changes at the most frequent rate. This optimization is particularly important for CI/CD processes where you want your automation to run as fast as possible.
Each layer of the image (in Dockerfile) is read-only except for the last layer, which is created for the container. The read/write container layer implements "copy-on-write," which means that files that are stored in lower image layers are pulled up to the read/write container layer only when edits are being made to those files. Those changes are then stored in the container/top/last layer. You can inspect which files have been pulled up to the container level with the docker diff command.
even if you create multiple containers using same base image and just change a few layers on top in the dockerfile it will use the cached copy that used to build previous containers with the same base, Because the containers use the same read-only layers, you can imagine that starting containers is very fast and has a very low footprint on the host.

 environmental drift