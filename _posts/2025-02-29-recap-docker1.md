---
layout: post
title: "A Docker recap for testers - part 1"
date: 2025-03-01
categories: blog
tags: [docker, testing, recap]
author: "Mehrdad Abdi"
img: docker-recap.webp
---

In this post, we recap docker containers concepts and provide a small cheatsheet.
This content is created with the assistance of ChatGPT to enhance clarity, structure, and accuracy. Efforts have been made to ensure the information is correct.

# 1. Docker Fundamentals

## what is Docker?

Docker is an open-source platform that enables you to build, package, and run applications in isolated environments called containers. Think of a container as a lightweight, portable unit that includes your application and everything it needs to run: code, libraries, dependencies, and runtime.

Why is this powerful? Because it works the same way on yourfriend’s laptop, a cloud server, or even ‘it on a Raspberry Pi. No more works on my machine’ issues!

Docker helps with:

- Portability: Run applications anywhere.

- Consistency: Eliminate environment-related bugs.

- Efficiency: Lightweight compared to virtual machines.

## how is a container different from a virtual machine (VM)?

A VM runs a full-blown operating system along with your app. It’s resource-heavy. A container, on the other hand, shares the host OS kernel and only contains the app and its dependencies. This makes containers lightweight, fast, and more efficient.

In simple terms:

- VMs = Big, heavy trucks.

- Containers = Lightweight, zippy scooters.

Both are useful but for different scenarios.

## Docker architecture

Docker Daemon: Runs in the background, manages containers.

Docker Client: This is what you interact with using commands like docker run. It talks to the daemon.

Docker Images: Blueprint for containers. It defines what’s inside your container.

Docker Containers: The running instances created from images.

Docker Registry (like Docker Hub): A library where images are stored and shared.

You (the client) place an order (run a command), the kitchen (daemon) prepares it using a recipe (image), and serves it as a dish (container).

# 2. Working with Docker Containers

## Running Your First Container

verify the installation with:

```
docker --version
```

Open your terminal and type:

```
docker run hello-world
```

This command does three things:

1. Pulls the hello-world image from Docker Hub.

2. Creates a container from that image.

3. Runs the container, which outputs a friendly message confirming Docker works.


## Interactive vs. Detached Mode 

Containers can run in two modes:

1- Interactive Mode: Allows you to interact with the container directly.

```
docker run -it alpine /bin/sh
```

- `-it` stands for interactive terminal.

- You’re now inside the container, running a shell.

To exit, type exit.

2- Detached Mode: Runs the container in the background.

```
docker run -d nginx
```

- `-d` runs the container in the background.
- Use docker ps to see running containers.


## Basic Docker Commands

List running containers:
```
docker ps
```

List all containers (including stopped ones):
```
docker ps -a
```

Get only the container IDs:
```
docker ps -q
```

Stop a container:
```
docker stop <container_id>
```

Start a stopped container:
```
docker start <container_id>
```

Remove a container:
```
docker rm <container_id>
```

Remove an image:
```
docker rmi <image_id>
```

View logs:
```
docker logs <container_id>
```

Inspect container details:
```
docker inspect <container_id>
```

Access a running container:
```
docker exec -it <container_id> /bin/sh
```

Remove all stopped containers:
```
docker container prune
```

Remove all containers (be careful!):
```
docker rm $(docker ps -aq)
```

## Naming Containers

By default, Docker assigns random names to containers. But you can assign custom names:

```
docker run --name my_nginx -d nginx
```

Now, instead of using the container ID, you can manage it using its name:

```
docker stop my_nginx
```

This makes container management easier, especially in complex environments.


# 3. Docker Images

## What is a Docker Image?

A Docker image is like a snapshot or a blueprint that contains everything needed to run an application:

- The application code
- Runtime environment
- Libraries and dependencies
- Configuration files

When you run an image, it becomes a container. Think of an image as a recipe and the container as the final dish. You can create as many dishes (containers) as you like from the same recipe (image).

## Understanding Image Layers

Docker images are made up of layers. Each layer represents a change or modification to the image, like adding a file or installing a package.

- Base layer: This is the starting point, like an OS (e.g., Ubuntu or Alpine).
- Intermediate layers: These include installed software, dependencies, and configurations.
- Top layer: This is your application code.

The beauty of layers is caching. If nothing changes in a layer, Docker reuses it, which makes builds faster!

## Pulling Docker Images

Want to use an existing image? Docker Hub is the place to go—it’s like the App Store for Docker images.

To pull an image from Docker Hub, use:
```
docker pull nginx
```

To see the images you’ve downloaded:
```
docker images
```

This lists all your local images, showing the repository name, tag, image ID, and size.

## Building Your Own Docker Image

Now, let’s create our own Docker image using a Dockerfile. A Dockerfile is a simple text file with instructions to build an image.

```
# Use a base image
FROM python:3.9  

# Set the working directory
WORKDIR /app  

# Copy files into the container
COPY app.py .  

# Install dependencies
RUN pip install flask  

# Run the app
CMD [ "python", "app.py" ]
```

Here are the key instructions:

- FROM: Defines the base image.
- COPY: Copies files from your system into the image.
- RUN: Executes commands during the build process.
- CMD: Specifies the default command to run when the container starts.


To build this image:
```
docker build -t my-flask-app .
```

- `-t my-flask-app` gives the image a name (or tag).
- The `.` means “build from the current directory.

After building, check it with:

```
docker images
```

And to run it:

```
docker run -p 5000:5000 my-flask-app
```

## Tagging and Versioning Images

Tagging helps manage different versions of an image.

When you build an image:
```
docker build -t my-flask-app:v1 .
```
This creates my-flask-app with version v1.

To tag an existing image:

```
docker tag my-flask-app my-flask-app:latest
```

You can run specific versions:

```
docker run my-flask-app:v1
```

## Optimizing Docker Images

Let’s talk about optimization. Smaller images are faster and more secure. Here’s how to keep them lean:

- Use lightweight base images like `alpine` instead of `ubuntu`.
- Clean up unnecessary files after installing dependencies:

```
RUN apt-get update && apt-get install -y python3 && rm -rf /var/lib/apt/lists/*
```

- Multi-stage builds: Compile your code in one stage, then copy the result into a minimal image.


# 4. Docker Volumes & Persistent Data

## Why Persistent Data Matters
By default, when a Docker container stops or is removed, all the data inside it is lost. This is fine for temporary tasks, but what if you’re running a database, a web app, or anything that needs to store data persistently?

That’s where Docker Volumes come in. They allow you to store data outside the container, ensuring it’s safe even if the container is gone.

## What Are Docker Volumes?
A Docker Volume is a special directory on your host system that’s managed by Docker. It’s designed for storing data persistently.

There are three main ways to persist data in Docker:

1. Volumes (Managed by Docker - the most common and recommended)
2. Bind Mounts (Directly map host directories)
3. tmpfs Mounts (Temporary, in-memory storage)

For most use cases, volumes are the best choice because they’re portable, secure, and optimized for Docker.

## Creating and Using Volumes

Let’s create and use a Docker volume!

### Create a volume:

```
docker volume create mydata
```

### Run a container with the volume:

```
docker run -d --name my-container -v mydata:/app/data busybox
```

- `-v mydata:/app/data` means:
  - `mydata` = the volume on your host
  - `/app/data` = where it’s mounted inside the container

### Verify the volume:

```
docker volume ls
docker inspect mydata
```

### Write data to the volume:

```
docker exec -it my-container sh
echo "Hello, Docker!" > /app/data/hello.txt
exit
```

Now, even if we remove the container:

```
docker rm -f my-container
```

The data is still there! Run another container with the same volume, and you’ll see the file.

## Bind Mounts

Next, let’s talk about Bind Mounts. Unlike volumes, bind mounts map a specific folder from your host machine into the container.

Example:

```
docker run -d -v /path/on/host:/app/data busybox
```

- Changes you make on your host directly reflect inside the container, and vice versa.

Bind mounts are useful for local development when you want real-time changes. But for production, volumes are usually safer and more manageable.

## Anonymous & Named Volumes

There are two types of volumes you’ll encounter:

1. Named Volumes: You explicitly give them a name, like mydata.
```
docker volume create mydata
docker run -v mydata:/data busybox
```
2. Anonymous Volumes: You don’t give a name; Docker generates one automatically.
```
docker run -v /data busybox
```

The downside? They’re harder to track and manage because you don’t control the name.

## Managing Volumes

Here’s how to manage Docker volumes:

List all volumes:
```
docker volume ls
```

Inspect volume details:
```
docker volume inspect mydata
```

Remove a volume:
```
docker volume rm mydata
```

Remove all unused volumes:
```
docker volume prune
```

Careful—this deletes volumes not used by any container.

## Best Practices for Persistent Data

Here are some best practices for managing persistent data with Docker:

- Use named volumes for clarity and management.
- Avoid storing sensitive data directly in bind mounts unless secure.
- Back up important volumes regularly. You can back up a volume like this:

```
docker run --rm -v mydata:/data -v $(pwd):/backup busybox tar cvf /backup/backup.tar /data
```

- For databases, always mount data directories to ensure persistence.

# 5. Docker Networking

## What is Docker Networking?

Docker Networking is the system that enables containers to communicate:

- With each other (container-to-container communication)
- With the host machine
- With external networks like the internet

By default, Docker isolates containers from the host and other containers unless you explicitly connect them. This improves security and control.

## Types of Docker Networks

Docker provides several network drivers, each designed for specific use cases:

1. Bridge Network (default): Great for containers on the same host. Containers can communicate using IP addresses or container names.

2. Host Network: Removes network isolation between the container and the host. The container shares the host’s IP and network stack.

3. None: Completely isolates the container. No network access at all.

4. Overlay Network: Used in Docker Swarm for multi-host networking.

5. Macvlan: Assigns a MAC address to containers, making them appear as physical devices on the network.

## Inspecting Docker Networks

To view existing Docker networks, run:
```
docker network ls
```

You’ll see default networks like:

- bridge (the default network)
- host
- none

To inspect the details of a network:

```
docker network inspect bridge
```
This shows information like connected containers, IP ranges, and more.

## Connecting Containers in the Same Network

Let’s see how containers communicate within the same network.

1. Create a custom network:
```
docker network create mynetwork
```
2. Run two containers on this network:
```
docker run -dit --name container1 --network mynetwork busybox
docker run -dit --name container2 --network mynetwork busybox
```
3. Test connectivity:
```
docker exec -it container1 ping container2
```
Since they’re on the same network, they can ping each other by name!
4. Run an app (like NGINX) and access it from another container:
```
docker run -d --name webserver --network mynetwork nginx
docker run -it --network mynetwork busybox wget -O- webserver
```

You’ll see the NGINX welcome page content in the terminal.

## Exposing Containers to the Outside World
To make a container accessible from your host or the internet, you publish ports using the `-p` flag:

```
docker run -d -p 8080:80 nginx
```

This maps port 80 inside the container to port 8080 on your host.

Now, open your browser and go to http://localhost:8080—you’ll see the NGINX page!

## Isolating Containers with Separate Networks

Sometimes, you want containers to be isolated from each other. Here’s how:

1. Create two separate networks:
```
docker network create frontend
docker network create backend
```
2. Run containers on different networks:
```
docker run -dit --name frontend-app --network frontend busybox
docker run -dit --name backend-db --network backend busybox
```
3. Try to ping between them:
```
docker exec -it frontend-app ping backend-db
```

Result? It won’t work! They’re isolated unless you connect them manually.

## Advanced Networking - Connecting Containers to Multiple Networks

You can even connect a container to multiple networks. This is great for apps that need to communicate with both public and private services.

1. Create networks:
```
docker network create public-net
docker network create private-net
```
2. Run a container on one network:
```
docker run -dit --name multi-net-app --network public-net busybox
```
3. Attach it to another network:
```
docker network connect private-net multi-net-app
```
Now, the container is part of both public-net and private-net!
