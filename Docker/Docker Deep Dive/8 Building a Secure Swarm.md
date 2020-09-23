# 8 Building a Secure Swarm

## The Big Picture

=> Secure Swarm Cluster

Cluster of Docker Nodes

## Swarm Clustering Deep Dive

=> Swarm Kit part of Docker

```bash
docker swarm init 
```
Manager and Leader

```bash
docker swarm join ...
```

(1) 3 5 7 Managers

Workers should be in reliable network

## Building a Secure Swarm

```bash
docker swarm init
docker swarm join --token ...
```

there are token for worker and manager nodes

**Autolock**

## Orchestration










