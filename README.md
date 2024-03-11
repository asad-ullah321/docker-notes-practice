
  # Docker Notes and Guide  
  Welcome to my Docker journey! This README is where I jot down my notes and discoveries as I explore Docker. Here, I'll be sharing insights, commands, and tips to help me remember and learn. 

---

  ## Need and History ⏰  
  Before virtualization or containers, programs and software were usually built and run directly on physical machines, known as bare metal. This setup caused problems, especially when multiple programs were developed on the same machine. They had to share resources like libraries, which could lead to conflicts. Changes to one program's libraries could affect others, forcing developers to make compromises. To solve these issues, virtualization and then containers emerged.
 
 --- 

  ## Comparison 
### Bare Metal 💻
This is simple to understand and direct access to the hardware can be useful for specific configuration, but can lead to:
- Intense dependency conflicts
- Low utilization efficiency
- Large blast radius
- Slow start up & shut down speed (minutes)
- Very slow provisioning & decommissioning (hours to days)


### Virtual Machines 🖥️
Virtual machines use a system called a "hypervisor" that can carve up the host resources into multiple isolated virtual hardware configuration which you can then treat as their own systems (each with an OS, binaries/libraries, and applications).

This helps improve upon some of the challenges presented by bare metal:

- No dependency conflicts
- Better utilization efficiency
- Small blast radius
- Faster startup and shutdown (minutes)
- Faster provisioning & decommissioning (minutes)



### Containers 📦
Containers are similar to virtual machines in that they provide an isolated environment for installing and configuring binaries/libraries, but rather than virtualizing at the hardware layer containers use native linux features (cgroups + namespaces) to provide that isolation while still sharing the same kernel.

This approach results in containers being more "lightweight" than virtual machines, but not providing the save level of isolation:

- No dependency conflicts
- Even better utilization efficiency
- Small blast radius
- Even faster startup and shutdown (seconds)
- Even faster provisioning & decommissioning (seconds)
- Lightweight enough to use in development!
---

## Commands

### Using Third Party Containers 🛳️
```bash
# Create a container from the ubuntu image
docker run -it --rm ubuntu:22.04

# Create a container from the ubuntu image (with a name and WITHOUT the --rm flag)
docker run -it --name my-ubuntu-container ubuntu:22.04

```


### Volume Mount 📂
We can use volumes and mounts to safely persist the data.

```bash
docker volume create my-volume

docker run  -it --rm -v my-volume:/my-data ubuntu:22.04
```
### Bind Mount 🔗
we can mount a directory from the host system using a bind mount:
```bash
# Create a container that mounts a directory from the host filesystem into the container
docker run  -it --rm -v ${PWD}/my-data:/my-data ubuntu:22.04
```

#### Database build code with mounts (volume mount):

```bash
docker run -e POSTGRES_USER=postgres \ 
-e POSTGRES_PASSWORD=tempPass \ 
-v pgdata:/var/lib/postgresql/data \ 
-p 5432:5432 \
 postgres:15.1-alpineAA


# With custom postresql.conf file
docker run -d --rm \
  -v pgdata:/var/lib/postgresql/data \
  -v ${PWD}/postgres.conf:/etc/postgresql/postgresql.conf \
  -e POSTGRES_PASSWORD=foobarbaz \
  -p 5432:5432 \
  postgres:15.1-alpine -c 'config_file=/etc/postgresql/postgresql.conf'

```
#### Connection string to connect to postgresql db running in container
```bash
postgresql://username:password@127.0.0.1:5432/mydatabase
DATABASE_URL:=postgres://postgres:tempPass@127.0.0.1:5432/postgres
```


### Basic Node Container image 
```bash
# Pin specific version for stability
# Use slim for reduced image size
FROM node:19.6-bullseye-slim

# Set NODE_ENV
ENV NODE_ENV production

# Specify working directory other than /
WORKDIR /usr/src/app

# Copy only files required to install
# dependencies (better layer caching)
COPY package*.json ./

# Install only production dependencies
# Use cache mount to speed up install of existing dependencies
RUN --mount=type=cache,target=/usr/src/app/.npm \
  npm set cache /usr/src/app/.npm && \
  npm ci --only=production

# Use non-root user
# Use --chown on COPY commands to set file permissions
USER node

# Copy remaining source code AFTER installing dependencies. 
# Again, copy only the necessary files
COPY --chown=node:node ./src/ .

# Indicate expected port
EXPOSE 3000

CMD [ "node", "index.js" ] 
```

### Docker Registry
Docker registries serve as repositories for Docker images. You can push your Docker images to registries like Docker Hub or GitHub Container Registry (GHCR) for sharing with others or for deployment on various platforms.

Docker-hub (login required)

```bash
docker tag local-image:tagname new-repo:tagname
docker push new-repo:tagname
```

[Docker Hub](https://hub.docker.com/) is a popular public registry where you can store and share Docker images. Here's how you can push an image to Docker Hub:
``` bash
docker tag my-scratch-image asadullah321/my-scratch-image # defaults to latest
docker push asadullah321/my-scratch-image

docker tag my-scratch-image asadullah321/my-scratch-image:abc-123
docker push asadullah321/my-scratch-image:abc-123
```

GHCR (login using token)
GitHub Container Registry (GHCR) allows you to publish Docker images in your GitHub repositories. Here's how you can push an image to GHCR:
```bash

docker tag my-scratch-image ghcr.io/asad-ullah321/my-scratch-image # defaults to latest
docker push ghcr.io/asad-ullah321/my-scratch-image

docker tag my-scratch-image: ghcr.io/asad-ullah321/my-scratch-image:abc-123 
docker push ghcr.io/asad-ullah321/my-scratch-image:abc-123 
```
---

## Learning Source
These Docker notes and guides are based on the learnings from [devops-directive-docker-course](https://github.com/sidpalas/devops-directive-docker-course). 
