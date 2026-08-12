## Self-Assessment Checklist
Mark yourself honestly — **can do**, **shaky**, or **haven't done**:

- [ yes] Run a container from Docker Hub (interactive + detached)
- [ yes] List, stop, remove containers and images
- [ yes] Explain image layers and how caching works
- [ yes] Write a Dockerfile from scratch with FROM, RUN, COPY, WORKDIR, CMD
- [ yes] Explain CMD vs ENTRYPOINT
- [ yes] Build and tag a custom image
- [ yes] Create and use named volumes
- [ yes] Use bind mounts
- [ yes] Create custom networks and connect containers
- [ yes] Write a docker-compose.yml for a multi-container app
- [ yes] Use environment variables and .env files in Compose
- [ yes] Write a multi-stage Dockerfile
- [ yes] Push an image to Docker Hub
- [ yes] Use healthchecks and depends_on

## Quick-Fire Questions
Answer from memory, then verify:
1. What is the difference between an image and a container?
   An image is a snapshot of the file and configuration needed to run an application, while a container is a running instance of that image. Containers are isolated and can be started, stopped, and removed without affecting the underlying image.

2. What happens to data inside a container when you remove it?
    When you remove a container, any data stored inside the container's writable layer is lost. To persist data, you should use volumes or bind mounts.
3. How do two containers on the same custom network communicate?
    Two containers on the same custom network can communicate by using their service names as hostnames.

4. What does `docker compose down -v` do differently from `docker compose down`?
    `docker compose down -v` removes the containers, networks, and also any named volumes associated with the services, while `docker compose down` only removes the containers and networks, leaving volumes intact.

5. Why are multi-stage builds useful?
    Multi-stage builds are useful because they allow you to use multiple FROM statements in a single Dockerfile, enabling you to create smaller, more efficient images by copying only the necessary artifacts from one stage to another. This helps reduce the final image size and improves security by excluding unnecessary build tools and dependencies.

6. What is the difference between `COPY` and `ADD`?
    The `COPY` instruction is used to copy files and directories from the host machine into the Docker image, while `ADD` can do the same but also has additional features like automatically extracting tar files and supporting remote URLs. However, `COPY` is generally preferred for its simplicity and predictability.

7. What does `-p 8080:80` mean?
    The `-p 8080:80` option maps port 8080 on the host machine to port 80 in the container. This allows you to access the application running inside the container on port 80 by connecting to port 8080 on the host.

8. How do you check how much disk space Docker is using?
    You can check how much disk space Docker is using by running the command `docker system df`. This command provides a summary of the disk usage for images, containers, volumes, and build cache.

