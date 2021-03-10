# Docker Mastery with Kubernetes and Swamp
## Lesson 18. Docker install and config
- `docker version`: displays the data of the docker client and server (engine)
- `docker info`: shows the configuration and setup values for the docker engine and client, like the number of containers, images, CPUs using, total memory used.
- Docker command format: `docker <management-command> <sub-command> (options)`

## Lesson 19. Containers
- images are the applications we want to run, while containers are running instances of said images. Multiple containers can run the same image.
- `docker container run`: starts a new container from an image
- `docker container ls`: list running containers
- `docker container start`: run an existing container
- `docker container stop`: stops the execution of a running container
- `docker container logs <container>`: shows the logs of the specified container
- `docker container top <container>`: displays the running processes of a container
- `docker container rm <containers>`: removes (deletes) one or more non-running containers

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

### Solution
```sh
# Create the containers
docker container run -d -p 80:80 --name nginx-c nginx
docker container run -d -p 8080:80 --name httpd-c httpd
docker container run -d -p 3306:3306 --name mysql-c -e MYSQL_RANDOM_ROOT_PASSWORD=yes mysql

# List the containers
docker container ls

# Display the logs of a container
docker container logs mysql-c

# Stop and remove a container
docker container stop <containers> && docker container rm <containers>
docker container ls
```

## Lesson 25: CLI Process Monitoring
- `docker container inspect <container>`: details of one container config
- `docker container stats`: live performance stats for all containers

## Lesson 26: Getting a shell inside a container
- `docker container run -it`: start a new container interactively
- `docker container exec -it`: run additional commands or processes in running container. Exiting or terminating the process doesn't stop the container.
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
- `docker network ls`: lists networks
- `docker network inspect <network>`: details of the config of a network, like the containers connected.
- `docker network create --driver <network>`: creates a new network with the possibility to using a different driver with the flag.
- `docker network connect <network> <container>`: Attach a network to a container
- `docker network disconnect <network> <container>`: Detach a network from container
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
3. Use the `--rm` flag when running your containers so that they are removed when stopped.
4. Ensure `curl` is installed on latest version for that distro
  - ubuntu: `apt-get update && apt-get install curl`
  - centos: `yum update curl`
5. Check `curl --version`

### Solution
```sh
# In one terminal window
docker container run --rm -it --name centos centos:7 bash
yum update curl
curl --version # curl 7.29.0

# In other terminal window
docker container run --rm -it --name ubuntu ubuntu:14.04 bash
apt-get update && apt-get install curl
curl --version # curl 7.35.0
```

## Lesson 34. Assignment. DNS Round Robin test
- DNS Round Robin refers to the technique of using multiple IP addresses and servers behind a single DNS name so that the service is always available. Basically, multiple hosts respond to the same DNS name.
- Since Docker Engine 1.11, we can assign an alias to a custom network so that multiple containers can respond to the same DNS address.

1. Create a new virtual network with the default bridge driver.
2. Create two containers from `elasticsearch:2` image. Remember to connect them to the custom network using `--network || --net`.
3. Research and use `--network-alias || --net-alias search` when creating the containers to give them an additional DNS name to respond to.
4. Use the `alpine` image to run `alpine nslookup search` to see the DNS addresses of the containers that respond to that DNS name. Connect the `alpine` container to the network.
5. Run `centos curl -s search:9200` with `--net` flag multiple times until you see both containers show. What this does is that it makes a request to the `search` DNS name and since the containers use `elasticsearch`, the default port is `9200`, and the output will be the configuration for it. In that output, you should see the container that took the request and responded. Keep in mind that `elasticsearch` gives random names when it responds.

### Solution
```sh
# Create virtual network
docker network create elastic-network

# Create elasticsearch containers
docker container run --rm -d --network elastic-network --network-alias search --name elastic-one elasticsearch:2
docker container run --rm -d --network elastic-network --network-alias search --name elastic-two elasticsearch:2

# Create alpine container
docker container run --rm -it --network elastic-network --name alpine alpine:3.10 /bin/sh
nslookup search

# Output
# Name:      search
# Address 1: 172.18.0.3 elastic-two.elastic-network
# Address 2: 172.18.0.2 elastic-one.elastic-network

# Create centos container
docker container run --rm -it --network elastic-network --name centos centos:latest bash
curl -s search:9200

# Output of one container
# {
#   "name" : "Eliminator",
#   "cluster_name" : "elasticsearch",
#   "cluster_uuid" : "nP9P4sNpSNmtzyTaHq6jjg",
#   "version" : {
#     "number" : "2.4.6",
#     "build_hash" : "5376dca9f70f3abef96a77f4bb22720ace8240fd",
#     "build_timestamp" : "2017-07-18T12:17:44Z",
#     "build_snapshot" : false,
#     "lucene_version" : "5.5.4"
#   },
#   "tagline" : "You Know, for Search"
# }

# Output of second container
# {
#   "name" : "Rage",
#   "cluster_name" : "elasticsearch",
#   "cluster_uuid" : "zkgaGoAeRGihfck2S3Q2MQ",
#   "version" : {
#     "number" : "2.4.6",
#     "build_hash" : "5376dca9f70f3abef96a77f4bb22720ace8240fd",
#     "build_timestamp" : "2017-07-18T12:17:44Z",
#     "build_snapshot" : false,
#     "lucene_version" : "5.5.4"
#   },
#   "tagline" : "You Know, for Search"
# }

# Remove the elasticsearch containers
docker container rm -f elastic-one elastic-two
```

## Lesson 36. What's an Image
- An image can be defined as the binaries and dependencies of an app, as well as the metadata to run it.
- No OS, no Kernel. It's just the binaries, because the host already provides the OS and kernel.

## Lesson 37. Using Docker Hub
- Official images can be found in the **Docker Hub** and these are maintained by Docker Inc. working with the team of the software the image represents.
- By default, when pulling an image from the Docker Registry, you pull the image with the tag `latest`, which means the latest version of the image.
- When in production, pull specific versions of images. Don't just let the software be automatically updated. You want to have control over that because your app may break if the image gets updated with the `latest` version, and you need other version, like `1.11.9` for example.
- Keep in mind that a version of an image can have different **tags**. If you pull a version using its different tags, you won't actually pull the image several times. And you can notice that by checking the *Image Id* of those images. Different tags, but the same image id.

## Lesson 38. Images and their layers
- `docker image history <image>`: shows the different layers or history of how that image was created (the steps taken to create the image)
- The SHA given to each layer ensures that we never duplicate the image data when creating another image. For example, if we download Ubuntu for an image, and later we want to use Ubuntu for another image, we don't need to download it again, as we have it cached, and can be used as the base for the new image.
- Layes can be anything like adding `mysql` to the image, or copying some files, or exposing a port.
- `docker image inspect <image>`: shows all the metadata of the image like its tags, the container configs, the ENV variables, the command that is run by default at the start, etc.
- As a review:
  - Images are made up of file system changes and metadata.
  - Each layer is uniquely identified and only stored once on a host.
  - A container is just a single read/write layer on top of the image.

## Lesson 39. Tagging and pushing to Docker Hub
- `docker image tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]`: assigns one or more tags to an image
- The way to refer to an image (because of their lack of a name) is with `<user>/<repo>:<tag>`, where `user` is the user host that created the image, `repo` is the repository name of the image.
- Only official repositories are allowed to have their repo name as only `<repo>/<tag>`, with no need for username or organization.
- Tags can be thought as different ways to call or label an Image ID. Meaning an image ID can have one or more tags. But a tag can only have one image ID.
- You can add a new tag to an existing image and that will only create a new label matching an image ID.
- `docker image push SOURCE_IMAGE[:TAG]`: uploades changed layers to an image registry that defaults to Docker Hub.
- `docker login`: logs the user in so that he can push and pull images to the registry. This stores a session id in the local Docker config.
- `docker logout`: logs out the user.
- Remember, `latest` is the tag used and implied by default when not provided anytime you refer to an image (`<user>/<repo>`).
- So if you do `docker image tag my_user/nginx my_user/nginx:testing` or `docker image tag my_user/nginx my_user/nginx:1.11.1`, you are essentially just adding two more labels to the image ID of `my_user/nginx`.
- You could actually tag an image with a different username, but it would only work locally. You cannot push to a registry if the username is different from your registry username.
- Best practice: create the repository in the registry before pushing the image to it.

## Lesson 40. Dockerfile Basics
- A Dockerfile is a file that contains the layers by which the image will be created. The layers are read from **top to bottom** and something done in a previous layer can be reflected in a latter layer.
- By default, no ports are exposed when creating an image with Dockerfile, so they need to be specified using `EXPOSE`. This does not mean that the host's ports will be connected to these ports. It just means that the containers created from the image of this Dockerfile will have these ports open.
- `CMD` is the final layer that must be in every Dockerfile, because it tells the image what command to run once the container is created.

## Lesson 41. Dockerfile Builds
- `docker image build -t REPO_NAME[:TAG] PATH_TO_DOCKERFILE`: Builds an image based on a Dockerfile. 
  - If you use `-f` you can specify the path to another Dockerfile instead of using the default file.
  - If you use `-t` you can give the image a repository name (highly recommended) with an optional tag (defaults to `latest`).
- Once a layer (step) changes in the Dockerfile and we are building an image, every step after the change needs to be re-run again and not rely on cache. **BE AWARE OF THE ORDER OF YOUR LAYERS (STEPS)**.
- **Best practice**: keep the steps that change the **least at the top of the file** and the steps that change the **most at the bottom**.

## Lesson 42. Extending official images
- You should aim to always use official images as the base of your custom images.
- You can skip using `EXPOSE` or `CMD` commands in the Dockerfile when using a base official image, because the image is "inheriting" those layers from it.
- Images that depend on images get all the steps and config of the parent image.
- You can change directories in your containers using the `WORKDIR` command in the Dockerfile. Technically, you can achieve the same with a `RUN cd [PATH]`, but the best practice is to use `WORKDIR`.

## Lesson 43. Assignment. Build your own Dockerfile and run containers from it
1. Dockerize an existing Node.js app found in the `dockerfile-assignment-1` directory.
2. Make a `Dockerfile`. Build it, test it.
3. Push it to the registry.
4. Remove it locally.
5. Run it again.

- Expect it to be iterative. It is rare that one gets it right the first time.

### Solution
1. Write the Dockerfile.
```Dockerfile
# Dockerfile
# Set up base image
FROM node:6-alpine

# Expose port
EXPOSE 3000

# Install tini and create app directory
RUN apk add --update tini && mkdir -p /usr/src/app

# Set directory to created app directory
WORKDIR /usr/src/app

# Copy package.json file into app directory as package.json
COPY package.json package.json

# Install dependencies
RUN npm install && npm cache clean --force

# Copy all from current dir on host to current dir in container (image)
COPY . .

# Run container command
CMD ["/sbin/tini", "--", "node", "./bin/www"]
```
2. Build the image from the Dockerfile.

```sh
# Build image with tag and path to Dockerfile
docker image build -t byomead/node-app:latest .

# Run container
docker container run --rm -d -p 80:3000 --name node-app byomead/node-app:latest

# Login to Docker Hub
docker login
# You will be prompted to enter user and password

# Push image to Docker Hub
# For best practices, first create the Docker repo in the registry and then push to it
docker image push byomead/node-app:latest

# Remove image from local cache
# First remove containers using the image
docker container rm -f [...CONTAINERS]

# Then remove the image
docker image rmi byomead/node-app:latest

# Run a new container from the image in the registry
docker container run --rm -d -p 80:3000 --name node-app byomead/node-app:latest
# This should download the image and run the container
```

## Lesson 45. Using `prune`
- You can use `prune` command to clean up images, volumes, build cache and containers.
- `docker image prune`: clean up "dangling" images. Add `-a` to remove all images not being used.
- `docker system prune`: clean up everything.
- `docker volume prune`: clean up unused volumes.

## Lesson 46. Container lifetime & persisten data
- Containers are meant to be **immutable** and **ephemeral**. Buzzwords for saying containers are not designed to change once deployed, they are temporary and disposable.
- If some of the configuration or versioning or anything needs to be changed for a container, re-deploy it, and get rid of the previous one.
- Since containers are disposable... what about when we use containers for databases, or when the app writes data into a file, or we need to keep unique data like key-value pairs?
- There are two solutions: **Volumes** and **Bind Mounts**
- **Volumes**: creates special location outside of container's UFS (*Union Filesystem*) to store unique data that is preserved across container removals and can be attached to any container created. Containers only see it as a local file path.
- **Bind Mounts**: directly mount a host's directory or file into a container. The container will see it as a local path, and won't know it's from the host.

## Lesson 47. Data Volumes
- One way to give a **Volume** to a container is by using the `VOLUME` command in the Dockerfile.
- `VOLUME [PATH]`. This command in Dockerfile will create a new volume location **on the host** and assign a path to it. Any data or files inside that path will *outlive* the container and be available until **manually** deleted.
- The container will think that the path is the one you gave it in the command, but actually Docker creates a new path in the host and maps it to the path provided in the Dockerfile.
- Volumes need to be manually deleted as an ensurance that the data will remain even after the containers that used it were stopped/removed.
- You can also assign a volume when creating a container using the `-v [PATH]` flag.
- By default, any volume created will be given a random id as name. This makes it hard to know which volume is for which container.
- To solve that problem, named volumes come in. You cannot *name* a volume when assigning it with the Dockerfile, but you can when assigning the volume at the moment of creation for the container. You need to use the `-v [NAME]:[PATH]` flag. `NAME` being the name of the volume and `PATH`, well... the path. If the named volume already exists, it will use that one, if not, it will create a new one.
- When creating a container from an image that specifies its `VOLUME` path in its `Dockerfile` and you want to name it, you have to name the path specified in the Dockerfile, since that's the place the container will be looking for the volume.
- `docker volume create [NAME]`: create named volumes separately from a container. The only use case you might want to use this is to give special configuration to the volume, like a different driver.

## Lesson 49. Bind Mounting
- **Bind Mounting** maps a *host* file or directory to a *container* file or directory.
- Basically, it's just two locations pointing to the same file(s).
- Since bind mounts need the files to actually exist in the hard drive of the host, and are host-specific, they cannot be used in `Dockerfile`, and must be declared when creating a container.
- `... run -v /Users/user/stuff:/path/container` (mac/linux)
- `... run -v //c/Users/user/stuff:/path/container` (windows)
- This looks similar to when we make named volumes, but instead of a name, we provide a real path in the host.
- They way Docker knows the difference between a named volume and a bind mount is that the bind mount always starts with `/`.
- *Pro tip*: Use the `$(pwd)` command to specify the host path. This command returns the current working directory, so make sure to run the command inside the directory you want to map into the container.
- Unless you specify the container to have **read-only** permissions over the host directory, the container can actually overwrite and delete files and directories in it, so be careful!

## Lesson 50. Assignment: Named Volumes
1. Create a `postgres` container with named volume `psql-data` using version `9.6.1`.
2. Use Docker Hub to learn the `VOLUME` path of the image and the versions needed to run it.
3. Check logs and check the named volume was created: `docker volume ls`.
4. Stop the container.
5. Create a new `postgres` container with the same named volume using `9.6.2`.
6. Check logs to validate.

### Solution
```sh
# Create postgres container with named volume (there is no longer version 9.6.1 in Docker Hub)
# Look for VOLUME path in the Dockerfile
docker container run -d --name postgres1 -v psql-data:/var/lib/postgresql/data postgres:9.6

# Check volumes created
docker volume ls

# Stop container
docker container rm -f postgres1

# Create new postgres container with same named volume (there is no version 9.6.2 in Docker Hub)
docker container run -d --name postgres2 -v psql-data:/var/lib/postgresql/data postgres:10.16

# Check logs to validate
docker container logs postgres2
```

## Lesson 52. Assignment. Edit code running in containers with bind mounts
1. Use Jekyll (static site generator) to start a local web server.
2. Source code is in `bindmount-sample-1`.
3. Edit files on host.
4. Container detects changes in host files and updates web server.
5. `docker container run -p 80:4000 -v $(pwd):/site bretfisher/jekyll-serve`
6. Refresh to see changes
7. Change file in `_posts\` and refresh browser to see changes.

### Solution
```sh
# Move to the bindmount-sample-1 directory
cd bindmount-sample-1

# Create Jekyll container
docker container run -p 80:4000 -v $(pwd):/site bretfisher/jekyll-serve
```