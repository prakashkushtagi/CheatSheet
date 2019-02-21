# Docker Cheatsheet

## Tutorial series

Get started with Docker: [https://docs.docker.com/engine/getstarted/](https://docs.docker.com/engine/getstarted/)

## Installation

### Linux

Install script provided by Docker:

```
curl -sSL https://get.docker.com/ | sh
```

Or see [Installation](https://docs.docker.com/engine/installation/linux/) instructions for your Linux distribution.


### Mac OS X

Download and install [Docker For Mac](https://docs.docker.com/docker-for-mac/)

### Create Docker VM with Docker Machine

You can use [Docker Machine](https://docs.docker.com/machine/) to:
- Install and run Docker on Mac or Windows
- Provision and manage multiple remote Docker hosts
- Provision Swarm clusters

A simple example to create a local Docker VM with VirtualBox: 

```
docker-machine create --driver=virtualbox default
docker-machine ls
eval "$(docker-machine env default)"
```

Then start up a container:

```
docker run alpine echo "hello-world"
```

That's it, you have a running Docker container.


## Container lifecycle

* Create a container: [`docker create imageName`](https://docs.docker.com/engine/reference/commandline/create).  
* Create and start a container in one operation: [`docker run imageName`](https://docs.docker.com/engine/reference/commandline/run)
  -  Remove the container after it stops `--rm`: `docker run  --rm alpine ls /usr/lib`
  -  Attach the container stdin/stdout to the current terminal use `-it`: `docker run -it ubuntu bash`
  -  To mount a directory on the host to a container add `-v` option, e.g. `docker run -v $HOSTDIR:$DOCKERDIR imageName` 
  -  To map a container port use `-p $HOSTPORT:$CONTAINERPORT`: `docker run -p 8080:80 nginx`
  -  To run the container in background use `-d` switch: `docker run -d -p 8080:80 nginx`
  -  Set container name: `docker run --name myContainerName imageName` 
  -  Example to run nginx web server to serve files from html directory on port 8080: 

```
    docker run -d -v $(pwd)/html:/usr/share/nginx/html -p 8080:80 --name myNginx nginx
    # access the webserver
    curl http://localhost:8080
``` 
* In some cases containers need [extended privileges](https://docs.docker.com/engine/reference/run/#runtime-privilege-and-linux-capabilities). Add privileges with the `--cap_add` switch, e.g. `docker run --cap-add SYS_ADMIN imageName`. The flag `--cap_drop` is used to remove privileges. 
* Rename a container: [`docker rename name newName`](https://docs.docker.com/engine/reference/commandline/rename/) 
* Delete a container: [`docker rm containerID`](https://docs.docker.com/engine/reference/commandline/rm)
  - Delete all unused containers: `docker ps -q -a | xargs docker rm`
  - Remove the volumes associated with a container: `docker rm -v containerName`
 
* Update container resource limits:
[`docker update --cpu-shares 512 -m 300M`](https://docs.docker.com/engine/reference/commandline/update/)  
* List running containers: [`docker ps`](https://docs.docker.com/engine/reference/commandline/ps/)
* List all containers: [`docker ps -a`](https://docs.docker.com/engine/reference/commandline/ps/)
* List all container IDs: [`docker ps -q -a`](https://docs.docker.com/engine/reference/commandline/ps/)

## Starting and stopping containers

* Start a container:
[`docker start containerName`](https://docs.docker.com/engine/reference/commandline/start) * [`docker stop`](https://docs.docker.com/engine/reference/commandline/stop) 
* Restart a container: [`docker restart containerName`](https://docs.docker.com/engine/reference/commandline/restart)
* Pause a container ("freeze"): [`docker pause containerName`](https://docs.docker.com/engine/reference/commandline/pause/)
* Unpause a container: [`docker unpause containerName `](https://docs.docker.com/engine/reference/commandline/unpause/) 
* Stop and wait for termination: [`docker wait containerName`](https://docs.docker.com/engine/reference/commandline/wait) 
* Kill a container (sends SIGKILL): [`docker kill containerName`](https://docs.docker.com/engine/reference/commandline/kill) 

## Executing commands in containers and apply changes

* Execute a command in a container: [`docker exec -it containerName command`](https://docs.docker.com/engine/reference/commandline/exec) 
* Copy files or folders between a container and the local filesystem: [`docker cp containerName:path localFile`](https://docs.docker.com/engine/reference/commandline/cp) or `docker cp localPath containerName:path`
* Apply changes in container file systems: [`docker commit containerName`](https://docs.docker.com/engine/reference/commandline/commit)


## Logging and monitoring

Logging on Docker could be challenging - check [Top 10 Docker Logging Gotchas](https://sematext.com/blog/top-10-docker-logging-gotchas/).

* Show container logs: [`docker logs containerName`](https://docs.docker.com/engine/reference/commandline/logs)
* Show only new logs: [`docker logs -f containerName`](https://docs.docker.com/engine/reference/commandline/logs)
* Show CPU and memory usage: [`docker stats`](https://docs.docker.com/engine/reference/commandline/stats)
* Show CPU and memory usage for specific containers: [`docker stats containerName1 containerName2`](https://docs.docker.com/engine/reference/commandline/stats)
* Show running processes in a container: [`docker top containerName`](https://docs.docker.com/engine/reference/commandline/top) 
* Show Docker events: [`docker events`](https://docs.docker.com/engine/reference/commandline/events) 
* Show storage usage: [`docker system df`](https://docs.docker.com/engine/reference/commandline/system_df) 
* To run a container with a custom [logging driver](https://docs.docker.com/engine/admin/logging/overview/) (i.e., to syslog), use: 
```
docker run -–log-driver syslog –-log-opt syslog-address=udp://syslog-server:514 \
alpine echo hello world
```
* Use [Sematext Docker Agent](https://sematext.com/docker/) for log collection. Create Docker Monitoring App & Logs App in [Sematext Cloud UI](https://sematext.com/cloud) to get required tokens. Start collecting all container metrics, host metrics, container logs and Docker events:

```
docker run -d --name sematext-agent-docker \
-e SPM_TOKEN=YOUR_SPM_TOKEN 
-e LOGSENE_TOKEN=YOUR_LOGSENE_TOKEN \
-v /var/run/docker.sock:/var/run/docker.sock \
sematext/sematext-agent-docker
```

The command above will collect all container metrics, host metrics, container logs and Docker events. 


## Exploring Docker information

* Show Docker info: [`docker info`](https://docs.docker.com/engine/reference/commandline/info)
* List all containers: [`docker ps -a`](https://docs.docker.com/engine/reference/commandline/ps) 
* List all images: [`docker image ls`](https://docs.docker.com/engine/reference/commandline/image_ls) 
* Show all container details: [`docker inspect containerName`](https://docs.docker.com/engine/reference/commandline/inspect) 
* Show changes in the container's files:[`docker diff containerName`](https://docs.docker.com/engine/reference/commandline/diff) 


## Manage Docker images

* List all images: [`docker images`](https://docs.docker.com/engine/reference/commandline/images) 
* Search in registry for an image:
[`docker search searchTerm`](https://docs.docker.com/engine/reference/commandline/search) 
* Pull image from a registry: [`docker pull imageName`](https://docs.docker.com/engine/reference/commandline/pull) pulls an image from registry to local machine
* Create image from Dockerfile: [`docker build`](https://docs.docker.com/engine/reference/commandline/build)
* Remove image: [`docker rmi imageName`](https://docs.docker.com/engine/reference/commandline/rmi) 
* Export container into tgz file: [`docker export myContainerName -o myContainerName `](https://docs.docker.com/engine/reference/commandline/export) 
* Create an image from a tgz file:[`docker import file`](https://docs.docker.com/engine/reference/commandline/import) 

## Docker networks

* List existing networks: [`docker network ls`](https://docs.docker.com/engine/reference/commandline/network_ls/)
* Create a network: [`docker network create netName`](https://docs.docker.com/engine/reference/commandline/network_create/)
* Remove network: [`docker network rm netName`](https://docs.docker.com/engine/reference/commandline/network_rm/)
* Show network details: [`docker network inspect`](https://docs.docker.com/engine/reference/commandline/network_inspect/)
* Connect container to a network: [`docker network connect networkName containerName`](https://docs.docker.com/engine/reference/commandline/network_connect/)
* Disconnect network from container: [`docker network disconnect networkName containerName`](https://docs.docker.com/engine/reference/commandline/network_disconnect/)

## Data cleanup 

Data management commands:

* Remove unused resources: [`docker system prune`](https://docs.docker.com/engine/reference/commandline/system_prune/)
* Remove unused volumes: [`docker volume prune`](https://docs.docker.com/engine/reference/commandline/volume_prune/)
* Remove unused networks: [`docker network prune`](https://docs.docker.com/engine/reference/commandline/network_prune/)
* Remove unused containers: [`docker container prune`](https://docs.docker.com/engine/reference/commandline/network_prune/)
* Remove unused images: [`docker image prune`](https://docs.docker.com/engine/reference/commandline/network_prune/)

====================================================================================================================================================================

****  http://www.openkb.info/2018/10/docker-cheat-sheet.html  ****

Docker Chet Sheet from OPENKB WebSite

Docker cheat sheet
This article records the command commands for Docker from https://docs.docker.com.

Orientation
1. List Docker CLI commands

docker
docker container --help

2. Display Docker version and info

docker --version
docker version
docker info

3. Execute Docker image

docker run hello-world

4. List Docker images

docker image ls

5. List Docker containers (running, all, all in quiet mode)

docker container ls
docker container ls --all
docker container ls -aq

Containers

1. Create image using this directory's Dockerfile

docker build -t friendlyhello .

2. Run "friendlyname" mapping port 4000 to 80

docker run -p 4000:80 friendlyhello
3. Run "friendlyname" mapping port 4000 to 80 in detach mode

docker run -d -p 4000:80 friendlyhello

4. Manage containers

# List all running containers
docker container ls
# List all containers, even those not running                                
docker container ls -a    
# Gracefully stop the specified container         
docker container stop <hash>  
# Force shutdown of the specified container         
docker container kill <hash>         
# Remove specified container from this machine
docker container rm <hash>        
# Remove all containers
docker container rm $(docker container ls -a -q)

5. Manage images

# List all images on this machine
docker image ls -a        
# Remove specified image from this machine                     
docker image rm <image id>   
# Remove all images from this machine         
docker image rm $(docker image ls -a -q)

6. Docker hub related

# Log in this CLI session using your Docker credentials
docker login          
# Tag <image> for upload to registry   
docker tag <image> username/repository:tag
# Upload tagged image to registry  
docker push username/repository:tag 
# Run image from a registry           
docker run username/repository:tag

Services

1. Sample docker-compose.yml that defines how Docker containers should behave in production.

version: "3"
services:
  web:
    # replace username/repo:tag with your name and image details
    image: username/repo:tag
    deploy:
      replicas: 5
      resources:
        limits:
          cpus: "0.1"
          memory: 50M
      restart_policy:
        condition: on-failure
    ports:
      - "4000:80"
    networks:
      - webnet
networks:
  webnet:

2. Enable swarm mode and make your current machine a swarm manager

docker swarm init

3. Mange Stack

# Run the specified Compose file                                        
docker stack deploy -c <composefile> <appname>
docker stack deploy -c docker-compose.yml getstartedlab

# List stacks or apps
docker stack ls

4. Manage Service

# List running services associated with an app
docker service ls               
# List tasks associated with an app 
docker service ps <service>     
docker service ps getstartedlab_web

5. Inspect task or container

docker inspect <task or container>

6. Take down an application

docker stack rm <appname>
docker stack rm getstartedlab

7. Take down a single node swarm from the manager

docker swarm leave --force

Swarms
1. Create a VM

docker-machine create --driver virtualbox myvm1
docker-machine create --driver virtualbox myvm2

2. List VMs

docker-machine ls

3. Instruct myvm1 to become swarm manager

docker-machine ssh myvm1 "docker swarm init --advertise-addr <myvm1 ip>"
docker-machine ssh myvm1 "docker swarm init --advertise-addr 192.168.99.100"

4. Instruct myvm2 to become swarm worker

docker-machine ssh myvm2 "docker swarm join --token <token> <ip>:2377"
docker-machine ssh myvm2 "docker swarm join --token SWMTKN-1-437hle524hh1hulxorovrlbfgfx645plt3sba8af3tewsb5q8d-7ez64tn8ggv7twildcg19j30c 192.168.99.100:2377"

5. List the nodes in the swarm

docker-machine ssh myvm1 "docker node ls"

6. Configure your shell to talk to myvm1

# Show shell variable for myvm1

docker-machine env myvm1 
# Set docker-machine shell variable

eval $(docker-machine env myvm1)
# Unset docker-machine shell variable

eval $(docker-machine env -u)
# Verify which is the active machine,indicated by asterisk

docker-machine ls

7. View join token from swarm manager

docker-machine ssh myvm1 "docker swarm join-token -q worker"

8. Open ssh session with the VM; type "exit" to end
docker-machine ssh myvm1

9. View nodes in swarm (while logged on to manager)

docker node ls

10. Leave swarm

# Make the worker leave the swarm

docker-machine ssh myvm2 "docker swarm leave"
# Make master leave, kill swarm

docker-machine ssh myvm1 "docker swarm leave -f"

11. Status/Stop/Start a VM

docker-machine status myvm1
docker-machine stop myvm1
docker-machine start myvm1

12. Stop/Start all running VMs

docker-machine stop $(docker-machine ls -q)
docker-machine start $(docker-machine ls -q)

13. Delete all VMs and their disk images

docker-machine rm $(docker-machine ls -q)

14. Copy files to VM's home directory

docker-machine scp docker-compose.yml myvm1:~

Stacks
1. Sample docker-compose-stack.yml which include "visualizer" and "redis" services

version: "3"
services:
  web:
    # replace username/repo:tag with your name and image details
    image: username/repo:tag
    deploy:
      replicas: 5
      restart_policy:
        condition: on-failure
      resources:
        limits:
          cpus: "0.1"
          memory: 50M
    ports:
      - "80:80"
    networks:
      - webnet
  visualizer:
    image: dockersamples/visualizer:stable
    ports:
      - "8080:8080"
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
    deploy:
      placement:
        constraints: [node.role == manager]
    networks:
      - webnet
  redis:
    image: redis
    ports:
      - "6379:6379"
    volumes:
      - "/home/docker/data:/data"
    deploy:
      placement:
        constraints: [node.role == manager]
    command: redis-server --appendonly yes
    networks:
      - webnet
networks:
  webnet:

2. Create "./data" directory on myvm1 -- swarm manager

docker-machine ssh myvm1 "mkdir ./data"

3. Deploy the stack

eval $(docker-machine env myvm1)
docker stack deploy -c docker-compose-stack.yml getstartedlab

4. Check visualizer on myvm1

http://192.168.99.100:8080
Or:
docker stack ps getstartedlab

5. List all services

docker service ls
