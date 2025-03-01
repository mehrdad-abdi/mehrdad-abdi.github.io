---
layout: post
title: "A Docker recap for testers - part 2"
date: 2025-01-25
categories: blog
tags: [docker, testing, recap]
author: "Mehrdad Abdi"
img: docker-recap.webp
---

This is the continue of [A Docker recap for testers - part 1](/recap-docker1).

# 6. Docker Compose

## What is Docker Compose?

Docker Compose is a tool that lets you define and manage multi-container Docker applications using a simple YAML file. Instead of starting containers manually with docker run, you define everything in a file called docker-compose.yml, and then run:
```
docker-compose up
```
And boom! All your containers are up and running with a single command.

Why it’s powerful:

- Simplifies deployment of complex applications.
- Easy scaling of services.
- Readable configuration in YAML format.

## Installing Docker Compose

Before we dive in, let’s make sure you have Docker Compose installed.
```
sudo apt-get update
sudo apt-get install docker-compose-plugin
docker-compose --version
```

## Basic Docker Compose Example

Let’s create our first Docker Compose file. Imagine we want to run a simple web app with NGINX and a Redis database.
1. Create a directory:
```
mkdir myapp && cd myapp
```
2. Create `docker-compose.yml`:
```
version: '3'
services:
  web:
    image: nginx
    ports:
      - "8080:80"
  redis:
    image: redis
```
Here’s what’s happening:
 - `version: '3'` defines the Compose file format.
 - `services` are the containers we want to run (`web` and `redis`).
 - `ports` maps port 80 inside the container to port 8080 on our machine.
3. Start the app:
```
docker-compose up
```
Now, visit http://localhost:8080 to see NGINX in action!

## Running Compose in Detached Mode

By default, docker-compose up runs in the foreground. To run in the background (detached mode), use:
```
docker-compose up -d
```
To stop the containers:
```
docker-compose down
```
This command gracefully shuts down all services and removes the network it created.

## Managing Services

Once your app is running, here are some handy commands:

- Check running services:
```
docker-compose ps
```
- View logs for all services:
```
docker-compose logs
```
- View logs for a specific service:
```
docker-compose logs web
```
- Restart a specific service:
```
docker-compose restart redis
```

## Adding a Database - MySQL Example

Let’s extend our Compose file to add a MySQL database.

Update docker-compose.yml

```
version: '3'
services:
  web:
    image: nginx
    ports:
      - "8080:80"
  redis:
    image: redis
  db:
    image: mysql
    environment:
      MYSQL_ROOT_PASSWORD: example
```

Here, we’ve added a `db` service with the MySQL image and set the root password using environment variables.

Restart the app:
```
docker-compose up -d
```
Now you have NGINX, Redis, and MySQL running together seamlessly!

## Volumes and Networks in Compose

Docker Compose also supports volumes and custom networks for advanced setups.

Adding a volume:
```
  db:
    image: mysql
    environment:
      MYSQL_ROOT_PASSWORD: example
    volumes:
      - db_data:/var/lib/mysql

volumes:
  db_data:
```
This ensures MySQL data persists even if the container stops.

Custom network:
```
networks:
  mynetwork:
```
Then assign it to services:
```
services:
  web:
    networks:
      - mynetwork
```

# 7. Scaling Services with Docker Compose

If you’ve ever wondered how to handle more traffic or improve the reliability of your app using multiple containers, this section is for you. 
We’ll cover how scaling works, real-world use cases, and live demos to show it in action. 
Let’s get started!

## What is Scaling in Docker Compose?

First, what does scaling mean in Docker Compose?

Simply put, scaling allows you to run multiple instances of a service to handle more load, improve availability, and distribute tasks efficiently.

For example:
- A web app serving thousands of users? Scale the frontend to multiple containers.
- A background worker processing tasks? Scale it to process jobs faster.
With Docker Compose, you can scale services with a single command—no complex configurations needed!

## Setting Up the Demo App 

Let’s create a simple web application to demonstrate scaling.

Create a project folder:
```
mkdir scaling-demo && cd scaling-demo
```

Create docker-compose.yml:
```
version: '3'
services:
  web:
    image: nginx
    ports:
      - "8080:80"
```
This basic setup runs a single NGINX container. Let’s start it:
```
docker-compose up -d
```
Now visit `http://localhost:8080`—you’ll see the default NGINX page.


## Scaling Services with Docker Compose

To scale the web service to 3 instances, simply run:
```
docker-compose up -d --scale web=3
```

Docker Compose will:

- Launch 3 identical NGINX containers.
- Connect them under the same network.
- Map port 8080 to the first available container (since you can’t bind the same port to multiple containers).

Verify the running containers:

```
docker-compose ps
```
You’ll see `web_1`, `web_2`, and `web_3` all running!

By default, Docker Compose doesn’t handle load balancing across containers for you. If you refresh `localhost:8080`, you’re only hitting one container.

To distribute traffic evenly, you’d typically:

- Use an external load balancer like HAProxy, NGINX in reverse proxy mode, or a cloud load balancer.
- Or, use Docker Swarm for advanced orchestration with built-in load balancing.

## Scaling Down & Managing Instances

To scale down the service:

```
docker-compose up -d --scale web=1
```
This stops and removes the extra containers, keeping just one running.

You can also stop all services:

```
docker-compose down
```
This removes all containers, networks, and resources created by Compose.

## Web + Worker Pattern

Let’s see a real-world example: scaling a web app with background workers.

Update docker-compose.yml:
```
version: '3'
services:
  web:
    image: nginx
    ports:
      - "8080:80"
  worker:
    image: busybox
    command: sh -c "while true; do echo Working...; sleep 5; done"
```
Now, scale the worker service to 4 instances:

```
docker-compose up -d --scale worker=4
```
Check the logs:
```
docker-compose logs worker
```
You’ll see all four workers running tasks in parallel—perfect for background job processing!

# 8. Docker in CI/CD Pipelines

## What is CI/CD?
- Continuous Integration (CI): Developers frequently merge their code into a shared repository, triggering automated builds and tests.
- Continuous Deployment/Delivery (CD): Once code passes tests, it is automatically deployed to a staging or production environment.

The goal? Fast, reliable, and automated software delivery—and Docker plays a huge role in making this process smooth!"

## Why Use Docker in CI/CD?
Docker is a game-changer for CI/CD because it provides:

- Consistency – Works the same on dev, test, and production environments.
- Fast, reproducible builds – No more 'works on my machine' issues!
- Isolation – Each build/test runs in its own container, preventing conflicts.

Let’s now see how Docker fits into each CI/CD stage.

## Using Docker in Build Pipelines
The first step in CI/CD is building your application. With Docker, we can:

- Package our application using a Dockerfile
- Build a lightweight, portable image
- Push the image to a container registry like Docker Hub or AWS ECR

Example: Building a Node.js app using Docker
1. Create a simple Dockerfile
2. Build and tag the image
```
docker build -t my-app:latest .
```
3. Push it to Docker Hub
```
docker tag my-app:latest mydockerhubusername/my-app:latest
docker push mydockerhubusername/my-app:latest
```
This image is now ready for deployment in the CI/CD pipeline!

## Testing Applications in Docker
CI/CD isn’t just about deploying—it’s also about automated testing!

We can run unit tests, integration tests, and even end-to-end tests inside Docker containers to ensure our app is bug-free before deployment.

If our app has Jest tests, we can modify our `Dockerfile`:
```
RUN npm install && npm test
```

Or, we can use a separate test stage in our pipeline:
```
docker run --rm my-app npm test
```
This ensures every change is tested in an isolated environment before deployment.

## Deploying Applications with Docker

Once our application passes testing, it’s time to deploy it!

With Docker, deployment is simple:

- Pull the latest Docker image
- Start the container

Example: Deploying with Docker Compose

```
version: '3'
services:
  app:
    image: mydockerhubusername/my-app:latest
    ports:
      - \"80:3000\"
```

Run it with:

```
docker-compose up -d
```
And just like that, your app is deployed!

## Integrating Docker with CI/CD Tools: GitHub Actions
Example: GitHub Actions Pipeline with Docker
```
name: CI Pipeline
on: [push]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
      - name: Build Docker image
        run: docker build -t my-app:latest .
  test:
    runs-on: ubuntu-latest
    needs: build  # Runs after build stage completes
    steps:
      - name: Run tests in container
        run: docker run --rm my-app npm test
```
Every time we push code, GitHub Actions will build, test, and validate the app inside a Docker container!

# 9. Docker Security Basics

## Running Containers with Least Privilege
By default, many containers run as the `root` user inside the container. 
This is a big security risk! If an attacker gains access to your container, they can exploit root privileges to break out into the host system.

🛑 Bad Practice: Running as Root (Avoid this!)
```
docker run -it my-insecure-container
```
✅ Best Practice: Run as a Non-Root User
Modify the Dockerfile to create a non-root user:
```
# Use official image
FROM node:18-alpine  

# Create a non-root user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup  
USER appuser  

# Set working directory
WORKDIR /app  
COPY . .  
CMD ["node", "server.js"]
```

## Understanding Image Vulnerabilities & Scanning for Them

Using vulnerable images can expose your application to exploits. But how do you know if your image has security issues?
Scan Docker Images with `trivy`:
```
docker pull nginx:latest  
trivy image nginx:latest
```
Other tools you can use:
- Docker Scout (built into Docker)
- Anchore
- Clair


## Managing Secrets in Docker

Never store passwords, API keys, or database credentials inside your Docker images! Let’s look at secure ways to handle secrets.

🛑 Bad Practice: Hardcoding Secrets in Dockerfile

```
ENV DB_PASSWORD=mysecretpassword
```

✅ Best Practice: Use Docker Secrets (for Swarm mode)

```
echo "mysecretpassword" | docker secret create db_password -
```

✅ Use `.env` Files (for Docker Compose)

```
DB_PASSWORD=mysecretpassword
```
Then reference it in docker-compose.yml:
```
services:
  app:
    environment:
      - DB_PASSWORD=${DB_PASSWORD}
```

## Best Practices for Securing Docker

✅ Use official base images (Minimize attack surface by using trusted sources.)

✅ Enable Docker Content Trust (Prevents using unverified images.)

✅ Use seccomp and AppArmor profiles (Restricts container permissions.)

✅ Limit container capabilities (Prevent containers from gaining unnecessary privileges.)

```
docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE my-secure-container
```

✅ Keep Docker updated (Security patches fix vulnerabilities.)

# 10. Docker Swarm - Introduction & Setup

## What is Docker Swarm?
Docker Swarm is Docker’s built-in orchestration tool for managing multiple containers across multiple machines. It turns a group of Docker hosts into a single virtual cluster.

Key Features of Docker Swarm:
- High Availability – If a container crashes, Swarm replaces it.
- Scalability – Easily scale containers up or down.
- Rolling Updates – Deploy new versions without downtime.
- Built-in Load Balancing – Distributes traffic between containers.

## Setting Up a Docker Swarm Cluster

Let’s create a Swarm cluster with one manager node and two worker nodes.

Step 1: Initialize the Swarm
Run this command on the manager node:

```
docker swarm init --advertise-addr <MANAGER-IP>
```

Step 2: Add Worker Nodes
On worker machines, join the cluster using the token from the manager:

```
docker swarm join --token <TOKEN> <MANAGER-IP>:2377
```

To verify all nodes:

```
docker node ls
```

## Deploying Services in Swarm

Now, let’s deploy a service in Swarm. Unlike regular docker run, Swarm uses docker service create.

Deploy a Web App Service
```
docker service create --name webapp -p 80:80 nginx
```

List Running Services
```
docker service ls
```

## Scaling Services in Swarm

One of Swarm’s biggest advantages is scaling. Let’s scale our web app to 3 replicas.
```
docker service scale webapp=3
```
To check replica distribution:
```
docker service ps webapp
```

## Updating Services with Zero Downtime

Swarm allows you to update services without downtime using rolling updates.

```
docker service update --image nginx:latest webapp
```
