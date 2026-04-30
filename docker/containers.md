# Docker — Containers

> Running, managing, and inspecting containers.

---

## docker run flags

```bash
docker run <image>                      # run a container (foreground)
docker run -d <image>                   # detached (background)
docker run -it <image> bash             # interactive with terminal
docker run --rm <image>                 # remove container after it exits
docker run --name <name> <image>        # assign a name
docker run -p <host>:<container> <image>  # map a port (e.g. -p 8080:80)
docker run -e KEY=value <image>         # set an environment variable
docker run -v <volume>:<path> <image>   # mount a volume or bind mount
docker run --network <network> <image>  # connect to a specific network
```

---

## Lifecycle

```bash
docker ps                               # list running containers
docker ps -a                            # list all containers (including stopped)
docker stop <container>                 # graceful stop (sends SIGTERM, then SIGKILL)
docker kill <container>                 # immediate stop (sends SIGKILL)
docker start <container>                # start a stopped container
docker restart <container>              # stop + start
docker rm <container>                   # remove a stopped container
docker rm -f <container>                # force remove a running container
docker container prune                  # remove all stopped containers
```

---

## Inspect & debug

```bash
docker logs <container>                 # show container output
docker logs -f <container>              # follow logs in real time
docker logs --tail 50 <container>       # last 50 lines
docker exec -it <container> bash        # open a shell in a running container
docker exec <container> <command>       # run a command without interactive shell
docker inspect <container>              # full JSON metadata
docker stats                            # live resource usage (CPU, memory, I/O)
docker top <container>                  # processes running inside the container
docker cp <container>:<path> <dest>     # copy a file out of a container
```

---

## Common pitfalls

- Using `docker kill` instead of `docker stop` — SIGKILL skips graceful shutdown, can corrupt state
- Forgetting `-it` for interactive containers — the shell exits immediately without a TTY
- `docker exec` only works on **running** containers — use `docker start` first if needed
- Logs disappear when container is removed — use a logging driver or volume for persistence
