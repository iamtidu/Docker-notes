# Docker Learning Guide

This guide covers Docker concepts from beginner to advanced levels, incorporating common commands and best practices.

---

## Beginner Topics

### What is Docker?

Docker is an open-platform tool designed to make it easier to create, deploy, and run applications by using **containers**. Containers allow a developer to package up an application with all of the parts it needs, such as libraries and other dependencies, and ship it all out as one package.

Think of it like a shipping container for software. Just like a physical shipping container can hold almost anything and be transported globally by ship, train, or truck, a Docker container can hold almost any software application and run consistently on any machine that has Docker installed (your laptop, a server in a data center, a cloud VM, etc.).

**Key Benefits:**
* **Consistency:** Eliminates the "it works on my machine" problem.
* **Efficiency:** Containers share the host OS kernel, making them much lighter and faster than Virtual Machines.
* **Portability:** Build once, run anywhere Docker is installed.
* **Isolation:** Containers run in isolated environments, preventing conflicts between applications or dependencies.

### Docker vs. Virtual Machines (VMs)

| Feature         | Docker Container             | Virtual Machine (VM)          |
| :-------------- | :--------------------------- | :---------------------------- |
| **Abstraction** | Operating System Level       | Hardware Level                |
| **OS** | Shares Host OS Kernel        | Includes a Full Guest OS      |
| **Resources** | Lightweight (MBs)            | Heavyweight (GBs)             |
| **Boot Time** | Seconds or less              | Minutes                       |
| **Isolation** | Process-level isolation      | Full hardware isolation       |
| **Use Case** | Packaging & running apps     | Running different OS, full isolation |

Essentially, VMs virtualize the hardware to run multiple operating systems, while Docker containers virtualize the operating system to run multiple applications *on the same* OS kernel.

### Docker Images

A **Docker image** is a lightweight, standalone, executable package that includes everything needed to run a piece of software, including the code, runtime, system tools, system libraries, and settings.

* **Blueprint:** Think of an image as a blueprint or a template for creating containers.
* **Read-Only:** Images are read-only. When you run an image, you create a writable layer on top of it – that's the container.
* **Layers:** Images are built in layers. Each instruction in a Dockerfile creates a new layer. This makes builds and distribution efficient, as layers can be cached and shared.

**Viewing Local Images:**

To see the Docker images you have downloaded or built on your machine:

```bash
docker image ls
# OR shorthand:
docker images
```
This command lists your images, showing their repository name, tag, image ID, creation date, and size.

### Docker Containers
A Docker container is a running instance of a Docker image. You can create, start, stop, move, or delete containers.

- Runtime Instance: It's the live, running version of your application packaged in the image.
- Isolated: Each container runs in its own isolated environment, with its own filesystem, networking, and process space, based on the image it was created from.
- Ephemeral (by default): If you stop and remove a container, any changes made inside its filesystem (that aren't saved to a volume) are lost.

### Running a Container:

To create and start a container from an image, use the docker run command, referencing the image by its ID or name:tag.

Bash

```bash
# Using the Image ID (find the ID using 'docker image ls')
docker run <IMAGE ID>

# Using the Image Name and Tag (more common)
docker run imagename:tag
```
This command typically starts the container in the foreground, attaching your terminal to its standard input, output, and error streams.

### Dockerfile Basics
A Dockerfile is a text document that contains all the commands a user could call on the command line to assemble an image.  docker `build uses` this file to automate the image creation process.   

### Common Dockerfile Instructions:

- `FROM:` Specifies the base image to start from (e.g., FROM python:3.9-slim). Every Dockerfile must start with FROM.
- WORKDIR: Sets the working directory for subsequent instructions like RUN, CMD, ENTRYPOINT, COPY, ADD.
- `COPY:` Copies files or directories from your local machine (the build context) into the image's filesystem (e.g., COPY . /app).
- `RUN:` Executes commands in a new layer on top of the current image, like installing packages (e.g., RUN pip install -r requirements.txt).
- `EXPOSE:` Informs Docker that the container listens on the specified network ports at runtime (e.g., EXPOSE 3000). This is documentation; it doesn't actually publish the port.
- `CMD:` Provides defaults for an executing container. There can only be one CMD. If you provide a command when running docker run, it overrides the CMD. (e.g., CMD ["python", "app.py"]).
- `ENTRYPOINT:` Configures a container that will run as an executable. Similar to CMD, but not easily overridden when running docker run.

#### Example Dockerfile (Simple Python App):

```bash
Dockerfile

# Use an official Python runtime as a parent image
FROM python:3.9-slim

# Set the working directory in the container
WORKDIR /app

# Copy the current directory contents into the container at /app
COPY . /app

# Install any needed packages specified in requirements.txt
RUN pip install --no-cache-dir -r requirements.txt

# Make port 80 available to the world outside this container
EXPOSE 80

# Define environment variable
ENV NAME World

# Run app.py when the container launches
CMD ["python", "app.py"]
```

### Basic Docker Commands (Your Examples Explained)
Here's a breakdown of the commands you listed:

1. Build an Image:

```Bash

# Build an image from the Dockerfile in the current directory (.)
docker build .
```
- Docker looks for a file named Dockerfile in the current directory (.).
- It executes the instructions in the Dockerfile, creating layers and ultimately a final image.
- The . also defines the "build context" - the set of files that can be referenced (e.g., with COPY) during the build.

2. Tag an Image During Build:

```Bash

# Build the image and tag it with a name and version
docker build -t myimage:01 .
```

- The `-t` flag stands for "tag". It allows you to assign a human-readable `name:version` (or `repository:tag`) to the image.
- `myimage` is the repository name (often your username/imagename for Docker Hub).
- `01` is the tag (often a version number like `1.0, latest`, or a git commit hash). Tags are pointers or labels for specific image revisions, making them easy to reference. `latest` is a common default tag, but it's best practice to use specific 
version tags.

3. List Images:

```Bash

docker image ls
# OR
docker images
```

- Displays all images stored locally on your Docker host.
4. Run a Container (Foreground):

```Bash

# Run a container from an image ID
docker run <IMAGE ID>
# Run a container from an image name and tag
docker run myimage:01
```

- Creates and starts a container. By default, it runs in the foreground, attached to your terminal. Press `Ctrl+C` to stop it (usually).

5. List Running Containers:

```Bash

docker ps
```
- Shows only the containers that are currently running. Displays container ID, image, command, creation time, status, ports, and names. Docker assigns random names if you don't provide one (e.g., goofy_mayer).

6. List All Containers (Running and Stopped):

```Bash

docker ps -a
```
- Shows all containers, including those that have exited. Useful for finding stopped containers to remove or restart.

7. Stop a Container:

```Bash

# Stop one or more containers by their name(s) or ID(s)
docker stop <NAMES or CONTAINER_ID> [<NAMES or CONTAINER_ID>...]
```

- Sends a SIGTERM signal (graceful shutdown), followed by a SIGKILL if it doesn't stop within a timeout. You get the <NAMES> from the docker ps or docker ps -a output.

8. Run Container with Port Mapping:

```Bash

# Map host port 3000 to container port 3000
docker run -p 3000:3000 <IMAGE ID or name:tag>
```
- The `-p` flag publishes a container's port(s) to the host.
Format: `-p <host_port>:<container_port>`.
Traffic sent to port 3000 on your Docker host machine will be forwarded to port 3000 inside the container.

9. Run Container in Detached Mode:

```Bash

# Run container in the background and print container ID
docker run -d -p 3000:3000 <IMAGE ID or name:tag>
```

- The -d flag runs the container in "detached" mode (in the background).
- Your terminal is immediately available again. Use docker ps to see it running.

10. Run Multiple Containers from the Same Image:

```Bash

# Run container 1, mapping host 3000 to container 3000
docker run -d -p 3000:3000 <IMAGE ID or name:tag>
# Run container 2, mapping host 3001 to container 3000
docker run -d -p 3001:3000 <IMAGE ID or name:tag>
# Run container 3, mapping host 3002 to container 3000
docker run -d -p 3002:3000 <IMAGE ID or name:tag>
```

- Demonstrates running multiple instances of the same application. Each needs a unique host port mapping if they expose the same container port.

11. Remove Stopped Containers:

```Bash

# Remove one or more stopped containers by name or ID
docker rm <NAMES or CONTAINER_ID> [<NAMES or CONTAINER_ID>...]
```

- Removes stopped containers. You cannot remove a running container unless you use the -f (force) flag (not generally recommended).

12. Run Container with Auto-Removal:

```Bash

# Run detached, automatically remove the container when it exits
docker run -d --rm -p 3000:3000 <IMAGE ID or name:tag>
```
- The `--rm` flag is useful for temporary tasks. Docker automatically cleans up the container (removes its filesystem) when the container exits.
