# **Commonly used Docker commands**, along with examples for each. These examples are practical and easy to follow.

---

## **1. Check Docker Version**
```bash
docker --version
```
Example:
```bash
$ docker --version
Docker version 20.10.12, build e91ed57
```

---

## **2. Run a Container**
Run a container from an image (e.g., `nginx`) and expose port 80.
```bash
docker run -d -p 8080:80 nginx
```
Example:
- `-d`: Run in detached mode (in the background).
- `-p 8080:80`: Map port 8080 on the host to port 80 in the container.

---

## **3. List Running Containers**
```bash
docker ps
```
Example:
```bash
$ docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED       STATUS       PORTS                   NAMES
c1a2b3d4e5f6   nginx     "/docker-entrypoint.…"   5 minutes ago Up 5 minutes 0.0.0.0:8080->80/tcp    nginx-container
```

---

## **4. List All Containers**
List all containers, including stopped ones.
```bash
docker ps -a
```

---

## **5. Stop a Running Container**
Stop a container using its name or ID.
```bash
docker stop CONTAINER_ID
```
Example:
```bash
docker stop c1a2b3d4e5f6
```

---

## **6. Remove a Container**
Remove a stopped container.
```bash
docker rm CONTAINER_ID
```
Example:
```bash
docker rm c1a2b3d4e5f6
```

---

## **7. Pull an Image**
Download an image from Docker Hub.
```bash
docker pull IMAGE_NAME
```
Example:
```bash
docker pull ubuntu
```

---

## **8. List Images**
View all downloaded images.
```bash
docker images
```
Example:
```bash
$ docker images
REPOSITORY      TAG       IMAGE ID       CREATED        SIZE
nginx           latest    abc123def456   3 days ago     133MB
ubuntu          latest    xyz789ghi012   5 days ago     29MB
```

---

## **9. Remove an Image**
Delete an unused image.
```bash
docker rmi IMAGE_ID
```
Example:
```bash
docker rmi abc123def456
```

---

## **10. Build an Image**
Build an image from a `Dockerfile`.
```bash
docker build -t IMAGE_NAME .
```
Example:
```bash
docker build -t my-app .
```
- `-t my-app`: Tags the image as `my-app`.
- `.`: Current directory contains the `Dockerfile`.

---

## **11. View Container Logs**
Check logs of a running container.
```bash
docker logs CONTAINER_ID
```
Example:
```bash
docker logs c1a2b3d4e5f6
```

---

## **12. Execute Commands Inside a Container**
Run a command inside a running container.
```bash
docker exec -it CONTAINER_ID COMMAND
```
Example:
```bash
docker exec -it c1a2b3d4e5f6 bash
```
- `-it`: Interactive mode with a terminal.
- `bash`: Opens a shell inside the container.

---

## **13. Stop All Running Containers**
Stop all running containers at once.
```bash
docker stop $(docker ps -q)
```
Example:
```bash
$ docker stop $(docker ps -q)
c1a2b3d4e5f6
d7e8f9g0h1i2
```

---

## **14. Remove All Stopped Containers**
Remove all containers that are not running.
```bash
docker rm $(docker ps -aq)
```

---

## **15. Create a Volume**
Create a persistent storage volume.
```bash
docker volume create VOLUME_NAME
```
Example:
```bash
docker volume create my-volume
```

---

## **16. Attach a Volume to a Container**
Mount a volume inside a container.
```bash
docker run -d -v VOLUME_NAME:/path/in/container IMAGE_NAME
```
Example:
```bash
docker run -d -v my-volume:/data nginx
```

---

## **17. View Docker Networks**
List all networks.
```bash
docker network ls
```

---

## **18. Create a Network**
Create a custom network.
```bash
docker network create NETWORK_NAME
```
Example:
```bash
docker network create my-network
```

---

## **19. Connect a Container to a Network**
Attach a container to a specific network.
```bash
docker network connect NETWORK_NAME CONTAINER_ID
```
Example:
```bash
docker network connect my-network c1a2b3d4e5f6
```

---

## **20. Start Services with Docker Compose**
Run services defined in a `docker-compose.yml` file.
```bash
docker-compose up
```

---

## **21. Stop Services with Docker Compose**
Stop services and remove containers, networks, and volumes created by `docker-compose`.
```bash
docker-compose down
```

---

## **22. View Docker System Information**
Check overall system usage (disk, containers, images, etc.).
```bash
docker system df
```

---

## **23. Remove Unused Resources**
Clean up unused images, containers, volumes, and networks.
```bash
docker system prune
```
Example:
```bash
docker system prune -a
```
- `-a`: Removes all unused images, not just dangling ones.

---

This cheat sheet provides clear examples of the most commonly used Docker commands. Let me know if you need this in a downloadable PDF format!
