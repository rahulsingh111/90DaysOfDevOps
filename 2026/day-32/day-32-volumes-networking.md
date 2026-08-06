### Task 1: The Problem

the data was removed because containers are ephemeral by nature. When a container is stopped and removed, all the data stored inside it is lost unless it is persisted using volumes or bind mounts. In this case, since the data was created inside the container without any persistence mechanism, it was deleted when the container was removed.

### Task 2: Named Volumes

the data is still there because named volumes provide a way to persist data outside of the container's lifecycle. When the container was removed, the named volume retained the data, allowing it to be accessed by a new container that was attached to the same volume.

### Task 3: Bind Mounts

A named volume is managed by Docker and is stored in Docker's storage area, while a bind mount is a direct mapping of a directory on the host machine to a directory in the container. With a bind mount, changes made to the files on the host are immediately reflected in the container, and vice versa. In contrast, named volumes are isolated from the host filesystem and are managed by Docker, making them more suitable for data that needs to persist across container restarts without being directly tied to the host's file structure.

### Task 4: Docker Networking Basics

was unable to ping by name but the ip ping from inside the containers worked. This is because the default bridge network does not provide automatic DNS resolution for container names. Containers on the default bridge can communicate with each other using their IP addresses, but they cannot resolve each other's names without additional configuration.

### Task 5: Custom Networks
yes they can ping each other by name. This is because custom bridge networks in Docker provide built-in DNS resolution, allowing containers to communicate with each other using their names. When containers are connected to the same custom network, they can easily resolve each other's names and establish communication without needing to rely on IP addresses.

### Task 6: Put It Together

the app container was able to connect to the database container using the database container's name. This demonstrates that when both containers are on the same custom network, they can communicate with each other using their names, allowing for seamless interaction between services in a multi-container application setup.