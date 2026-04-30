# Docker — Networking

> How containers communicate with each other and the outside world.

---

## Network drivers

| Driver | Description | Use case |
|---|---|---|
| `bridge` | Default — isolated virtual network on the host | Most single-host setups |
| `host` | Container shares the host's network stack directly | Performance-sensitive apps (Linux only — not available on Docker Desktop for Mac/Windows) |
| `none` | No network access | Fully isolated containers |
| `overlay` | Spans multiple Docker hosts | Docker Swarm / multi-host |

> Containers on the same custom bridge network can reach each other by **container name** (built-in DNS). The default bridge network does not have this — use custom networks.

---

## Commands

```bash
docker network ls                                    # list all networks
docker network create <name>                         # create a custom bridge network (default driver)
docker network create --driver bridge <name>         # same, explicit driver
docker network inspect <name>                        # full details (containers, subnet, config)
docker network rm <name>                             # remove a network
docker network connect <network> <container>         # attach a running container to a network
docker network disconnect <network> <container>      # detach a running container from a network
```

---

## Port mapping

```bash
docker run -p 8080:80 nginx             # host port 8080 → container port 80
docker run -p 127.0.0.1:8080:80 nginx  # bind to localhost only (not exposed publicly)
docker run -P nginx                     # map all EXPOSE'd ports to random host ports
docker port <container>                 # show current port mappings
```

> `-p` publishes a port. `EXPOSE` in a Dockerfile only documents intent — it does nothing without `-p` at runtime.

---

## Container-to-container communication

```bash
# Create a shared network
docker network create mynet

# Attach both containers to it
docker run -d --name db --network mynet postgres
docker run -d --name app --network mynet myapp

# Inside app, reach db by name:
# psql -h db -U user mydb
```

---

## Common pitfalls

- Using the default bridge network and expecting DNS — container name resolution only works on custom bridge networks
- Exposing ports on `0.0.0.0` (default) when only localhost access is needed — use `127.0.0.1:<port>` to avoid exposing the port to the outside
- Forgetting to create a shared network in Compose — services in the same Compose file share a default network automatically, no need to declare one unless you want isolation between services
- Using `--network host` on Mac/Windows — it connects to the Docker VM's network, not the actual host machine's network. It only behaves as expected on Linux
