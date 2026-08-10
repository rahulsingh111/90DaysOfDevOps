- **Container commands** — run, ps, stop, rm, exec, logs
`docker run` - Create and start a new container from an image
`docker ps` - List running containers
`docker stop` - Stop a running container
`docker rm` - Remove a stopped container
`docker exec` - Execute a command in a running container
`docker logs` - View the logs of a container


- **Image commands** — build, pull, push, tag, ls, rm
`docker build` - Build an image from a Dockerfile
`docker pull` - Download an image from a registry
`docker push` - Upload an image to a registry
`docker tag` - Create a new tag for an image
`docker image ls` - List images on the local machine
`docker image rm` - Remove an image from the local machine


- **Volume commands** — create, ls, inspect, rm
`docker volume create` - Create a new volume
`docker volume ls` - List volumes on the local machine
`docker volume inspect` - Display detailed information about a volume
`docker volume rm` - Remove a volume from the local machine


- **Network commands** — create, ls, inspect, connect
`docker network create` - Create a new network
`docker network ls` - List networks on the local machine
`docker network inspect` - Display detailed information about a network
`docker network connect` - Connect a container to a network


- **Compose commands** — up, down, ps, logs, build
`docker-compose up` - Create and start containers defined in a docker-compose.yml file
`docker-compose down` - Stop and remove containers, networks, and volumes defined in a docker-compose.yml
`docker-compose ps` - List containers defined in a docker-compose.yml file
`docker-compose logs` - View the logs of containers defined in a docker-compose.yml file
`docker-compose build` - Build images defined in a docker-compose.yml file


- **Cleanup commands** — prune, system df
`docker system prune` - Remove unused containers, networks, images, and optionally volumes
`docker system df` - Show Docker disk usage


- **Dockerfile instructions** — FROM, RUN, COPY, WORKDIR, EXPOSE, CMD, ENTRYPOINT
`FROM` - Specify the base image for the Dockerfile
`RUN` - Execute a command during the build process
`COPY` - Copy files or directories from the host to the image
`WORKDIR` - Set the working directory for subsequent instructions
`EXPOSE` - Inform Docker that the container listens on the specified network ports at runtime
`CMD` - Provide default arguments for the container's main process
`ENTRYPOINT` - Configure a container to run as an executable with specified arguments