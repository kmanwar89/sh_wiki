# Information gathering

`docker ps -a` or `docker container list` \- List all containers
`docker ps -aq` \- List all container ID's

- This command is useful because it can be called as a variable \[ex: *$(docker ps -aq)*\] and fed into other commands (such as `docker stop` or `docker rm`) as shown in the command above.

`docker image list` \- List all docker images
`docker system df` \- Docker disk usage
`docker system info` \- System info

# Interacting with containers

> ### Container creation

`docker start` \- starts any stopped container
`docker run` \- creates & starts the newly-created container

> ### Stopping containers

`docker stop $(docker ps -aq)` \- stop all running containers by feeding in container ID's
`docker stop <container name | container ID>` \- stop a single container using SIGTERM

> #### Alternate syntax:

`docker container stop <container name | container ID>`
`docker kill <container name | container ID>` \- kill a single container using SIGKILL

# Building

Note: the file must be called *Dockerfile*, verbatim. It is not a file extension, but that is the default filename that the docker build command looks for.

`docker build .` \- build an image from a Dockerfile in the current directory
`docker build [-t username/repository:tag] .` \- build an image from a Dockerfile in the current directory, and optionally tag it.
`docker build -t [username/repository:tag] - < [name of dockerfile]` \- build an image using a Dockerfile read from STDIN
`docker build -f [Dockerfile] .` \- builds from a Dockerfile in an alternate location

`docker run --name <human readable name> <image name>` \- create a container from an image, and give it a custom name (instead of the system generated ones)

# Working with Docker Compose

Docker Compose is a way of building and interacting with containers using a YAML-formatted file. This has the unique advantage of not having to type out the commands each time, but also allowing for manipulation of networks, databases and inter-connections between containers in a very simple and seamless way.

`docker compose up -d` \- create and start containers from a *docker-compose.yml* file in a daemonized or "detached" fashion, i.e. the container will continue running in the background
`docker-compose up --force-recreate --build -d` \- quick one-liner to pull down changes to a container after modifying the *compose* file. Can optionally run *docker compose up -d* again and changes will be reflected

# Managing Images

`docker image rm <image name>` \- deletes an image from local image store
`docker rmi $(docker images -q)` \- removes all unused images
`docker tag <imagename:oldtag> <username/repository:newtag>` \- renames/re-tags an image

# Using Repositories/Docker Hub

`docker pull <image name>:[tag]` \- retrieve image from repository
`docker push <username/repository:tag>` \- pushes image to a repository.

- Currenly, Docker Hub limits to a single private repository in the format &lt;username/repo name&gt;. Tags can be used to manage multiple images within the repository.

# Container and System Management

> ## Deleting images & containers

`docker rm $(docker ps -aq)` \- remove all containers based on container ID. Will not remove containers that haven't been stopped

> ## General system cleanup

`docker system prune [--volumes]` \- removes all stopped containers, networks, dangling images/cache and optionally any unused volumes
`docker volume prune` \- removes any unused volumes

> ## Upgrading an image

This is useful when a new version is released; rather than deleting the image and re-downloading it, use the following condensed command to upgrade an image. [Source](https://stackoverflow.com/questions/49316462/how-to-update-existing-images-with-docker-compose)
`docker-compose up --force-recreate --build -d`
Can also use:
`docker-compose pull`

> ## Naming and interacting with containers

When working in a docker-compose file, there are two ways to refer to a container - the name that is present while defining the various components of the compose file, or the "container_name" directive. For example:

```
CONTAINER ID   IMAGE                              COMMAND                  CREATED             STATUS                          PORTS                                                                                  NAMES
a5130a954a0f   jc21/nginx-proxy-manager           "/init"                  3 seconds ago       Up 1 second                     0.0.0.0:80-81->80-81/tcp, :::80-81->80-81/tcp, 0.0.0.0:443->443/tcp, :::443->443/tcp   npm_prod
```

The name "npm\_prod" is shown above in the far-right column under "Names", and this was defined in the compose file under the "container\_name" option:

```
version: '3'
services:
  app:
    image: jc21/nginx-proxy-manager
    container_name: npm_prod <--------
```

If this option is removed, the container will inherit a default name in the form of &lt;image name\_service name\_numeral&gt;. For example:

```
CONTAINER ID   IMAGE                              COMMAND                  CREATED        STATUS                      PORTS                                                                                  NAMES
c00ecf065e9d   jc21/nginx-proxy-manager           "/init"                  1 second ago   Up 1 second                 0.0.0.0:80-81->80-81/tcp, :::80-81->80-81/tcp, 0.0.0.0:443->443/tcp, :::443->443/tcp   npm_app_1
```

The docker-compose file now looks like:

```
version: '3'
services:
  app:
    image: jc21/nginx-proxy-manager
```

This is a helpful option to help keep track of various containers, to help separate dev/prod or for any other reson where a unique identifier would be helpful, or necessary

# Interacting with a container

## Log management

`docker container logs --tail 100 <container name>` \- delete last 100 lines of a containers' log
`docker logs <container name> --follow` \- equivalent of *tail -f* functionality to view logs in real time. Useful for diagnosis

## Get a console into the container

`docker exec -it <container name> /bin/bash`

# Volumes

There are two main types of volumes - bind and mount, explained by the following graphic:

![b3d2eb112b402b9cbe1651ab5257b387.png](:/ff5177757f594e5b994dcfb155fd41c3)

The syntax looks like the following --

- For *docker-compose*:

```
services:
  service_name:
    image: image_repo/image_name:tag
    container_name: name_of_container
    volumes:
      - /path/on/host/machine/to/expose:/path/within/the/docker/container:<optional permissions>
```

- For *docker run*:

```
docker run -detached -port_to_expose ### \
    -v /path/on/host/machine/to/expose:/path/within/the/docker/container:<optional permissions>
```

# Docker Networking

## Create docker network

docker network create &lt;name&gt;

## Attach a container to that network

docker run --network &lt;name&gt; --name=*name of container* \[*image name*\]

## Exposing Host and Container ports

Depending on the service running, the container will require either a private, ephemeral port that is decided by the developers, or an exposed, well-known port. For a secure setup behind a reverse proxy (such as NPM), only 3 ports should be bound to the host machine (and thereby forwarded by the router/exposed to the Internet): 80, 81 and 443.

Other services, such as Pi-Hole, AdGuard or other services that operate on well-known port numbers may require those ports to be exposed (such as 53 for DNS). When creating a docker-compose file, ports are exposed with the following syntax:

```
ports:
  - <host port>:<container port>
```

For example, NPM requires 80, 81 and 443, so the port mapping should look like:

```
ports:
  - 80:80
  - 81:81
  - 443:443
```

On the router, these ports would be directly forwarded. However, since only 1 service can use a port at a time, and multiple services may require well-known ports such as 80 or 443, custom ports can be bound to those, and Docker's service handles the "routing" of the traffic into the container. If a service requires port 80, for instance, this limitation can be overcome as follows:

```
ports:
  - 8080:80
```