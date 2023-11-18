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


what's a dockerfile
A Dockerfile is basically a text document that contains all the commands a user could call on the command line to assemble an image.

explain tags in docker image

What's the difference between "Shared" and "Simple" tags in docker images?

docker tag python-hello-world [dockerhub username]/python-hello-world

docker login

docker push [image name]

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

