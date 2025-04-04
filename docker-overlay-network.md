# Docker Overlay Network Example

## Introduction
Overlay networks in Docker allow containers running on different Docker hosts to communicate securely. This is particularly useful in Docker Swarm mode for creating multi-host container networks.

## Prerequisites
- Docker Engine installed on multiple hosts
- Docker Swarm initialized

## Step 1: Initialize Docker Swarm
On the first host (manager node):
```bash
docker swarm init --advertise-addr <MANAGER-IP>
```

## Step 2: Join Worker Nodes
Run the join command on worker nodes:
```bash
docker swarm join --token <TOKEN> <MANAGER-IP>:2377
```

## Step 3: Create an Overlay Network
```bash
docker network create --driver overlay --attachable my-overlay-network
```

## Step 4: Deploy Services Using the Overlay Network
Create a web service:
```bash
docker service create --name web-service \
  --network my-overlay-network \
  --replicas 3 \
  -p 80:80 \
  nginx
```

Create a backend service:
```bash
docker service create --name api-service \
  --network my-overlay-network \
  --replicas 2 \
  my-api-image
```

## Step 5: Test Communication
Services on the same overlay network can communicate using service names:
```bash
# From inside a container in the web-service
curl http://api-service:8080/
```

## Benefits
- Containers can communicate across hosts
- Built-in service discovery
- Secure communication with encryption options
- Load balancing for service instances
