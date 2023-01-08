# Docker Commands

* [Images](#images)
* [Containers](#containers)
* [Containerizing an app](#containerizing-an-app)
* [Docker Compose](#docker-compose)
* [Docker Networking](#docker-networking)
* [Volumes](#volumes)

### Images
* **docker image pull** - is the command to download images. We pull images from repositories inside of remote registries. By default, images will be pulled from repositories on Docker Hub. This command will pull the image tagged as latest from the alpine repository on Docker Hub: **docker image pull alpine:latest**.
* **docker image ls** - lists all of the images stored in your Docker host’s local image cache. To see the SHA256 digests of images add the --digests flag.
* **docker image inspect** - is a thing of beauty! It gives you all of the glorious details of an image — layer data and metadata.
* **docker manifest inspect** - allows you to inspect the manifest list of any image stored on Docker Hub. This will show the manifest list for the redis image: **docker manifest inspect redis**.
* **docker buildx** - is a Docker CLI plugin that extends the Docker CLI to support multi-arch builds.
* **docker image rm** - is the command to delete images. This command shows how to delete the alpine:latest image — **docker image rm alpine:latest**. You cannot delete an image that is associated with a container in the running (Up) or stopped (Exited) states.

### Containers
* **docker container run** - is the command used to start new containers. In its simplest form, it accepts an image and a command as arguments. The image is used to create the container and the command is the application the container will run when it starts. This example will start an Ubuntu container in the foreground, and tell it to run the Bash shell: **docker container run -it ubuntu /bin/bash**.
* **Ctrl-PQ** - will detach your shell from the terminal of a container and leave the container running (UP) in the background.
* **docker container ls** - lists all containers in the running (UP) state. If you add the **-a flag** you will also see containers in the stopped (Exited) state.
* **docker container exec** - runs a new process inside of a running container. It’s useful for attaching the shell of your Docker host to a terminal inside of a running container. This command will start a new Bash shell inside of a running container and connect to it: **docker container exec -it <container-name or container-id> bash**. For this to work, the image used to create the container must include the Bash shell.
* **docker container stop** - will stop a running container and put it in the Exited (0) state. It does this by issuing a SIGTERM to the process with PID 1 inside of the container. If the process has not cleaned up and stopped within 10 seconds, a SIGKILL will be issued to forcibly stop the container. docker container stop accepts container IDs and container names as arguments.
* **docker container start** - will restart a stopped (Exited) container. You can give docker container start the name or ID of a container.
* **docker container rm** - will delete a stopped container. You can specify containers by name or ID. It is recommended that you stop a container with the docker container stop command before deleting it with docker container rm.
* **docker container inspect** - will show you detailed configuration and runtime information about a container. It accepts container names and container IDs as its main argument.

### Containerizing an app
* **docker image build** - is the command that reads a Dockerfile and containerizes an application. The **-t flag** tags the image, and the **-f flag** lets you specify the name and location of the Dockerfile. With the -f flag, it is possible to use a Dockerfile with an arbitrary name and in an arbitrary location. The build context is where your application files exist, and this can be a directory on your local Docker host or a remote Git repo.
* **The FROM instruction in a Dockerfile** - specifies the base image for the new image you will build. It is usually the first instruction in a Dockerfile and a best-practice is to use images from official repos on this line.
* **The RUN instruction in a Dockerfile** - allows you to run commands inside the image. Each RUN instruction creates a single new layer.
* **The COPY instruction in a Dockerfile** - adds files into the image as a new layer. It is common to use the COPY instruction to copy your application code into an image.
* **The EXPOSE instruction in a Dockerfile** - documents the network port that the application uses.
* **The ENTRYPOINT instruction in a Dockerfile** - sets the default application to run when the image is started as a container.
* Other Dockerfile instructions include **LABEL, ENV, ONBUILD, HEALTHCHECK, CMD** and more…

### Docker Compose
* **docker-compose up** - is the command to deploy a Compose app. It expects the Compose file to be called docker-compose.yml or docker-compose.yaml, but you can specify a custom filename with the -f flag. It’s common to start the app in the background with the **-d flag**.
* **docker-compose stop** - will stop all of the containers in a Compose app without deleting them from the system. The app can be easily restarted with **docker-compose restart**.
* **docker-compose rm** - will delete a stopped Compose app. It will delete containers and networks, but it will not delete volumes and images.
* **docker-compose restart** - will restart a Compose app that has been stopped with **docker-compose stop**. If you have made changes to your Compose app since stopping it, these changes will not appear in the restarted app. You will need to re-deploy the app to get the changes.
* **docker-compose ps** - will list each container in the Compose app. It shows current state, the command each one is running, and network ports.
* **docker-compose down** - will stop and delete a running Compose app. It deletes containers and networks, but not volumes and images.

### Docker Networking
* **docker network ls** - lists all networks on the local Docker host.
* **docker network create** - creates new Docker networks. By default, it creates them with the nat driver on Windows and the bridge driver on Linux. You can specify the driver (type of network) with the **-d flag**.**docker network create -d overlay overnet** will create a new overlay network called overnet with the native Docker overlay driver.
* **docker network inspect** - provides detailed configuration information about a Docker network.
* **docker network prune** - deletes all unused networks on a Docker host.
* **docker network rm** - deletes specific networks on a Docker host.

### Volumes
* **docker volume create** - is the command we use to create new volumes. By default, volumes are created with the local driver, but you can use the **-d flag** to specify a different driver.
* **docker volume ls** - will list all volumes on the local Docker host.
* **docker volume inspect** - shows detailed volume information. Use this command to see many interesting volume properties, including where a volume exists in the Docker host’s filesystem.
* **docker volume prune** - will delete all volumes that are not in use by a container or service replica. Use with caution!
* **docker volume rm** - deletes specific volumes that are not in use.
* **docker plugin install** - will install new volume plugins from Docker Hub.
* **docker plugin ls** - lists all plugins installed on a Docker host.