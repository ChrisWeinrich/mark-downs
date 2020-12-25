# 9 Container Networking

## Network Types

### Bridge Networking (Default)

- Creepy
- Each Network is isolated

### Overlay Networking

- Single Overlay Network

```bash
docker network create -o encrypted
```

### MACVLAN/ Transparent Driver

Every Container has its own IP

   

