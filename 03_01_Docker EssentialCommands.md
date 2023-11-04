# Understanding Docker Commands: A Comprehensive Guide

* `docker ps`: 
List All Running Containers. The `docker ps` command provides a snapshot of all your currently running containers. It offers a quick overview of the containers' names, IDs, statuses, and resource usage.

* `docker ps -a`: 
List All Containers (Stopped and Running). Adding the `-a` flag expands the scope to include all containers, both running and stopped. This command gives you a comprehensive view of your container ecosystem.

* `docker ps -a -q`: 
List Only Container IDs. For scripting and automation purposes, you can use `docker ps -a -q` to extract only the IDs of all containers. This minimalist output is handy when you need to pass container IDs as parameters to other commands.

* `docker stop container-id`: 
Stop a Running Container. To gracefully stop a running container, use `docker stop` followed by the container's ID. Docker sends a SIGTERM signal to the main process inside the container, allowing the process to shut down gracefully. The container is given a specific amount of time (default is 10 seconds) to stop and perform any necessary cleanup operations before it is forcefully terminated. Please note, if a container does not respond to the SIGTERM signal within the specified timeout period, Docker will forcefully terminate the container using a SIGKILL signal, which can result in data corruption or loss if the application is in the middle of critical operations.

* `docker kill container-id`: 
The `docker kill` command is used to forcefully stop a running container in a Docker environment. Unlike the `docker stop` command, which sends a SIGTERM signal to the container, allowing it to shut down gracefully, `docker kill` immediately terminates the container by sending a SIGKILL signal. This abrupt termination does not give the container any opportunity to clean up or execute shutdown procedures, making it a forceful and instantaneous way to stop a container.

* `docker start container-id`: Start a Stopped Container
When a container is in a stopped state, use `docker start` along with the container's ID to initiate it. This command reactivates the container, making it operational again.

* `docker restart container-id`: Restart a Running Container
To restart an already running container, employ `docker restart` followed by the container's ID. Docker will stop the container, then immediately start it again, ensuring a fresh state.

* `docker port container-id`: List Port Mappings of a Container
With `docker port`, you can inspect the port mappings of a specific container. It reveals the internal ports mapped to the host system, facilitating network configuration.

* `docker rm container-id` or `name`: Remove a Stopped Container
To clean up your system, use `docker rm` followed by the container's ID or name. This command removes a stopped container, freeing up resources.

* `docker rm -f container-id` or `name`: Remove a Running Container Forcefully
Forcibly remove a running container by adding the `-f` flag to `docker rm`. Be cautious when using this command, as it terminates the container immediately, potentially leading to data loss.

* `docker pull image-info`: Pull an Image from Docker Hub
To fetch an image from the Docker Hub repository, employ `docker pull` followed by the image's name. This command downloads the specified image, making it available for creating containers.

* `docker pull nirajp82/my_first_docker_image`: Pull a Specific Image from Docker Hub
If you want a particular image, specify the repository and image name in the `docker pull` command. This allows you to download customized or specific images for your projects.

* `docker logout`
`docker logout` command logs out the Docker registry associated with the Docker CLI. It is essential for security and managing different Docker registries. After logging out, you need to log in again to perform any registry-related operations.

* `docker login -u username -p password`
Use `docker login` to authenticate with a Docker registry. Replace `username` with your Docker Hub username and `password` with your Docker Hub password. This command allows you to pull and push images to your Docker Hub account.

* `docker stats`
The `docker stats` command provides a live stream of container resource usage statistics. It displays CPU, memory, network, and disk I/O usage for all running containers. This is helpful for monitoring the performance of your containers.

* `docker top container-id or name`
With `docker top`, you can view the running processes inside a specific container. Provide the `container-id` or `name` as an argument, and Docker will show you the active processes within that container.

* `docker version`
`docker version` provides detailed information about your Docker installation, including the Docker client and server version, build date, and other relevant details. It helps you verify the version you are running and check for updates.

* `docker container run -d image_name`
The `docker container run` command creates and runs a new container based on the specified `image_name`. The `-d` flag runs the container in detached mode, allowing it to run in the background. This is commonly used for running applications as services.

* `docker system prune`
`docker system prune` removes all stopped containers, unused networks, dangling images, and build cache. It helps clean up your system by removing unused resources, freeing up disk space, and improving system performance.

* `docker rmi image-id`
The `docker rmi` command removes one or more specified images from your local system. Replace `image-id` with the ID of the image you want to remove. This is useful for cleaning up unused or unwanted images to free up disk space.

* `docker logs <container id> --follow`
Adding the `--follow` flag to `docker logs` allows you to continuously stream the logs of a container, displaying new log messages as they are generated. This is useful for real-time monitoring of container activities.

* `docker logs <container id> --since`
The `--since` flag in `docker logs` allows you to filter log output to show only the logs generated since a specific timestamp. Specify the timestamp, and Docker will display logs generated after that time.

* `docker logs <container id> --tail`
`--tail` flag limits the number of lines displayed in the log output. For example, `docker logs <container id> --tail=50` shows only the last 50 lines of logs for the specified container.

* `docker logs <container id> --timestamps`
Adding `--timestamps` to `docker logs` displays timestamps along with the log entries. This helps you understand when each log entry was generated, providing context to the log output.

* `docker logs <container id> --details`
`docker logs` command shows the logs generated by a container. Use the `--details` flag to include extra details such as timestamps and container IDs in the log output.

* `docker logs <container id> --until`
The `--until` flag in `docker logs` filters log output to show only the logs generated before a specific timestamp. Specify the timestamp, and Docker will display logs generated before that time.

These Docker commands provide essential functionalities for managing containers, monitoring resources, and analyzing container logs effectively. Understanding their usage is crucial for efficient Docker container orchestration.

*For more detailed information, check out the official [Docker Documentation](https://docs.docker.com/).*
