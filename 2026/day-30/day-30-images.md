Docker layers are read-only filesystem snapshots that together make up a Docker image. Each instruction in a Dockerfile creates a new layer. Layers that modify the filesystem (such as COPY, ADD, or RUN) have a size, while metadata instructions (such as ENV, ENTRYPOINT, CMD, and STOPSIGNAL) show 0B because they only change the image configuration. Docker uses layers to reuse cached content, reduce storage by sharing common layers between images, and speed up image builds and downloads.

### Task 3: Container Lifecycle
`docker create --name mycontainer ubuntu sleep infinity`	Created
`docker start mycontainer`	Up (Running)
`docker pause mycontainer`	Up (Paused)
`docker unpause mycontainer`	Up (Running)
`docker stop mycontainer`	Exited
`docker restart mycontainer`	Up (Running)
`docker kill mycontainer`	Exited
`docker rm mycontainer`	Removed


### Task 4: Working with Running Containers
run an nginx container in detached mode:
`docker run -d --name mynginx nginx`
view its logs: `docker logs mynginx`
view real-time logs (follow mode): `docker logs -f mynginx`
exec into the container and look around the filesystem: `docker exec -it mynginx /bin`
run a single command inside the container without entering it: `docker exec mynginx ls /`
inspect the container to find its IP address, port mappings, and mounts: `docker inspect mynginx`


### Task 5: Cleanup
stop all running containers in one command: `docker stop $(docker ps -q)`
remove all stopped containers in one command: `docker rm $(docker ps -a -q)`
remove unused images: `docker image prune -a`
check how much disk space Docker is using: `docker system df`