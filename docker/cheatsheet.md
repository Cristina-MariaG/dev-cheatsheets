# Docker — Cheatsheet

> Most used commands, grouped by situation.

---

## Check state

```bash
docker ps                               # running containers
docker ps -a                            # all containers (including stopped)
docker images                           # local images
docker stats                            # live CPU / memory usage
docker system df                        # disk usage (images, containers, volumes)
```

---

## Run a container

```bash
docker run -d --name <name> -p <host>:<container> <image>   # standard detached run
docker run -it --rm <image> bash                             # interactive, auto-remove on exit
docker run -e KEY=value -v <vol>:<path> <image>              # with env var and volume
```

---

## Build & push

```bash
docker build -t <name>:<tag> .          # build image from current directory
docker tag <image> <registry>/<name>:<tag>  # tag for a registry
docker push <registry>/<name>:<tag>     # push to registry
docker pull <image>:<tag>               # pull from registry
```

---

## Manage containers

```bash
docker stop <container>                 # graceful stop
docker start <container>                # start a stopped container
docker rm <container>                   # remove stopped container
docker rm -f <container>                # force remove running container
docker container prune                  # remove all stopped containers
```

---

## Debug

```bash
docker logs -f <container>              # follow logs in real time
docker exec -it <container> bash        # open a shell inside running container
docker inspect <container>              # full JSON metadata
docker cp <container>:<path> .          # copy a file out of a container
```

---

## Compose

```bash
docker compose up -d                    # start all services in background
docker compose up -d --build            # rebuild and start
docker compose down                     # stop and remove containers
docker compose down -v                  # also remove volumes
docker compose logs -f <service>        # follow a service's logs
docker compose exec <service> bash      # shell into a running service
docker compose ps                       # status of all services
```

---

## Clean up

```bash
docker image prune -a                   # remove all unused images
docker container prune                  # remove all stopped containers
docker volume prune                     # remove all unused volumes
docker network prune                    # remove all unused networks
docker system prune -a                  # remove everything unused (images, containers, networks)
docker system prune -a --volumes        # same + volumes (destructive — data loss)
```
