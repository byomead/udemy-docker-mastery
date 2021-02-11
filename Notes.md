# Docker Mastery with Kubernetes and Swamp
## Lesson 18. Docker install and config
- docker version: displays the data of the docker client and server (engine)
- docker info: shows the configuration and setup values for the docker engine and client, like the number of containers, images, CPUs using, total memory used.
- Docker command format: `docker <management-command> <sub-command> (options)`

## Lesson 19. Containers
- images are the applications we want to run, while containers are running instances of said images. Multiple containers can run the same image.
- docker container run: starts a new container from an image
- docker container ls: list running containers
- docker container start: run an existing container
- docker container stop: stops the execution of a running container
- docker container logs <container>: shows the logs of the specified container
- docker container top <container>: displays the running processes of a container
- docker container rm <containers>: removes (deletes) one or more non-running containers

## Lessson 20. What happens when we run a container
1. Looks for the image locally in the cache
2. If not found, looks for the image in the remote image repo (defaults to Docker Hub)
3. Downloads the latest version by default
4. Creates a new container based on the image
5. Gives it a virtual IP in the private network inside docker engine
6. Starts the container using the command (CMD) in the image's Dockerfile

## Lesson 21: Containers vs VMs
- Containers are just processes run in the host's OS. You can actually see the process listed if you run `ps aux` (shows all running processes in the OS).

## Lesson 23: Assignment. Manage multiple containers
1. Run an nginx, mysql, adn httpd (apache) server
2. Run them in detach mode and name them
3. nginx should listen to 80:80, httpd to 8080:80 and mysql to 3306:3306
4. Pass an env variable to the mysql container using --env (MYSQL_RANDOM_ROOT_PASSWORD=yes)
5. Find the password created using the logs of the container
6. Stop and remove the containers
7. List the containers (they should be gone)

```shell
# Solution
docker container run -d -p 80:80 --name nginx-c nginx
docker container run -d -p 8080:80 --name httpd-c httpd
docker container run -d -p 3306:3306 --name mysql-c -e MYSQL_RANDOM_ROOT_PASSWORD=yes mysql
docker container ls
docker container logs mysql-c
docker container stop <containers> && docker container rm <containers>
docker container ls
```

## Lesson 25: CLI Process Monitoring
- docker container inspect <container>: details of one container config
- docker container stats: live performance stats for all containers

## Lesson 26: Getting a shell inside a container
- docker container run -it: start a new container interactively
- docker container exec -it: run additional commands or processes in running container. Exiting or terminating the process doesn't stop the container.
- The -it flag is actually two flags:
  - i: interactive, keeps the session open to receive terminal input
  - t: pseudo-tty, simulates a terminal prompt
- Containers only run as long as the command at the start up runs. If you reset the startup command of a container to `bash` for example, once you exit it, the container stops.
- docker container start -ai: runs a stopped container interactively
- The -ai flag is similar to the -it flag.
- docker pull <image>: pulls an image from the remote image repo. 
 
## Lesson 27: Docker Networks
- Each container is by default connected to a private network called "bridge".
- All containers on a network can talk to each other without publishing their ports.
- Best practice is to create a new network for each app.
- Network defaults work well, but are easily configurable.
- Containers can be in 0 to several networks.
- Only one container can listen to a host port at a time.
- Any traffic from the outside of the host that goes to the port mapped by the container will arrive to the container.

## Lesson 28: Docker Networks CLI Management
- docker network ls: lists networks
- docker network inspect <network>: details of the config of a network, like the containers connected.
- docker network create --driver <network>: creates a new network with the possibility to using a different driver with the flag.
- docker network connect <network> <container>: Attach a network to a container
- docker network disconnect <network> <container>: Detach a network from container
- By default, there are 3 networks: bridge, host and none:
  - bridge is the default network to which the containers are connected to automatically
  - host is the network the container can use to connect directly to the host network, sacrificing security.
- Using the --network when starting a container attaches it to the specified network.

## Lesson 29: Docker Networks DNS
- By default, the bridge network doesn't enable DNS, so if you wanted to connect between your containers in that network without relying on IP addresses, you would need to use the --link when starting the container.
- It is a better practice to simply create a custom network and connect your containers there, because all custom networks have DNS enabled by default.

## Lesson 31. Assignment. CLI App Testing
1. Use different Linux distro containers to check `curl` cli tool version.
2. Uso two different terminal windows to start `bash` in `centos:7` and `ubuntu:14.04` interactively.
3. Use the `--rm` flag when running your containers so they are removed when stopped.
4. Ensure `curl` is installed on latest version for that distro
  - ubuntu: `apt-get update && apt-get install curl`
  - centos: `yum update curl`
5. Check `curl --version`

```shell
# Solution
# In one terminal window
docker container run --rm -it --name centos centos:7 bash
yum update curl
curl --version # curl 7.29.0

# In other terminal window
docker container run --rm -it --name ubuntu ubuntu:14.04 bash
apt-get update && apt-get install curl
curl --version # curl 7.35.0
```