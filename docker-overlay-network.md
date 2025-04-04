# Docker Overlay Network Example

## Use Case: A Scalable Web Application with Multiple Services  

In this example, we deploy a web application with:  
1. An **Nginx service** (proxy/load balancer).  
2. A **PHP-FPM backend**.  
3. A **MySQL database**.  
4. An **overlay network** to allow services to communicate, even if they run on different nodes.  

---

## Step 1: Initialize Docker Swarm  
First, we need to initialize a Swarm cluster (on a single server for testing or multiple machines if needed).  

```bash
docker swarm init
```

To add more nodes, get the token with:  

```bash
docker swarm join-token worker
```

Then, run the provided command on the worker nodes.  

---

## Step 2: Create an Overlay Network  

```bash
docker network create --driver overlay my_overlay_network
```

---

## Step 3: Deploy the Services  

Now, we create a stack using **Docker Compose**.  

### File: `docker-compose.yml`  

```yaml
version: "3.8"

services:
  nginx:
    image: nginx:latest
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    networks:
      - my_overlay_network
    deploy:
      replicas: 2
      restart_policy:
        condition: on-failure

  php:
    image: php:fpm
    volumes:
      - ./app:/var/www/html
    networks:
      - my_overlay_network
    deploy:
      replicas: 2
      restart_policy:
        condition: on-failure

  db:
    image: mysql:8
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: mydb
      MYSQL_USER: user
      MYSQL_PASSWORD: password
    volumes:
      - db_data:/var/lib/mysql
    networks:
      - my_overlay_network
    deploy:
      replicas: 1
      restart_policy:
        condition: on-failure

networks:
  my_overlay_network:
    external: true

volumes:
  db_data:
```

---

## Step 4: Deploy the Application to the Cluster  

```bash
docker stack deploy -c docker-compose.yml my_app
```

---

## Overlay Network Explanation  
- All services use the `my_overlay_network`, allowing them to communicate **without exposing** ports outside the cluster.  
- **Nginx** can route requests to `php:9000` using the **service name**.  
- **PHP** can connect to the database via `db:3306`.  
- The **overlay network** ensures that services can communicate **even if they run on different nodes**.

