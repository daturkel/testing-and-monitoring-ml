# Docker and Docker Compose

## Docker recap

What is Docker? Docker packages software into **containers** that have all the dependencies and tools needed for the code to run.

Docker lets us automate the deployment of applications as portable, self-contained containers.

### Containers and images

**Images** contain files and metadata but are not actually running the processes.

**Containers** are created from images and are running the processes.

**Containers are the instantiations of images.**

### Docker vs virtual machines

Both seek to isolate an application into a self-contained unit.

A **virtual machine** emulates a full computer, including hardware. 

Containers share the host machine's kernel, making them more light-weight and fast.

[This article](https://www.backblaze.com/blog/vm-vs-containers/) contains a good comparison.

### The Dockerfile

An example Dockerfile for a Flask app:

```dockerfile
FROM python:3.7-alpine
WORKDIR /code

# Set env vars required by Flask
ENV FLASK_APP app.py
ENV FLASK_RUN_HOST 0.0.0.0

# Install gcc so certain Python packages install faster
RUN apk add --no-cache gcc musl-dev linux-headers

# copy local requirements.txt into container
COPY requirements.txt requirements.txt

# install requirements
RUN pip install -r requirements

# Copy the current directory . into the workdir . in the image
COPY . .

# Set the default command for the container to flask run
CMD ["flask", "run"]
```

We create the `docker build` command to create the image, and `docker run` to run a container based on that image.

### Docker Hub

Docker Hub is a repository for sharing and storing Docker images.

## Why use Docker?

Remember that the ML code is a very small portion of an ML system. There's a lot of supporting scaffolding.

Docker benefits on the software side:

**Standardization**: our environments are deterministic and repeatable and we can develop on an environment that matches production. **Build once, run anywhere.**

**Efficiency for continuous integration**: we can use the same image across every step of the deployment process, allowing us to parallelize aspects of CI. E.g. running tests in parallel and redeploying to a different environment.

**Maintainability**: code will run the same no matter where it is. Helps eliminate environment-related issues and eliminates the issue of "it works on my machine."

**Isolation and security**: each container has its own resources isolated from other containers. Applications in containers can't escape their containers.

**Compatibility and scalability**: Docker containers can be run on just about any cloud provider or hosting service, so we can scale horizontally (more instances) or vertically (more resources in an instance). We can also use container orchestration software like Kubernetes to dynamically scale containers.

Docker benefits from a business perspective:

**Cost reduction**: the aforementioned CI efficiency can reduce compute costs, and containerization means no cost of OS or VM licensing.

### Why for ML in particular?

**Reproducibility**: we should be able to reproduce a prediction created at any point in the past.

**Scalability**: training models can be CPU/GPU-intensive, so being able to scale up can save us a ton of time.

**Compatibility**: once a model is in production (or testing for prod), it's very useful to be able to pass around a production-like environment to debug locally.

## Docker Compose

**Docker Compose lets us define and run multi-container Docker applications.**

We configure our application's containerized services with a YAML file.

### How does Docker Compose work?

**Multiple isolated environments on a single host**: this is the basic premise of containerization.

**Preservation of [volume data](https://blog.container-solutions.com/understanding-volumes-docker) when containers are created**: we can persist data (between Compose runs) from a container as needed.

**Only recreate containers that have changed**: this lets us spin up a multi-container application quickly.

**Variables and moving composition between environments**: we can separate code and configuration.

### Why use Docker Compose?

If your application has more than one container (like a server and a database), then building and connecting the containers from separate Dockerfiles is a pain.

Docker Compose lets us define multi-container applications in a single YAML file and run the whole application with a single command.

### docker-compose.yml

Running compose has three steps

1. Define each service in its own Dockerfile
2. Define the services in relation to each other in the docker-compose.yml file
3. Run `docker-compose up`

An example docker-compose.yml file:

```yaml
version: '3' # version of compose
services:
  web:
    build: . # dockerfile is in current directory
    ports:
    - "5000:5000"
    volumes:
    - .:/code
    - logvolume01:/var/log
    links:
    - redis
  redis:
    image: redis # pull from dockerhub rather than build
volumes:
  logvolume01: {}
```

Now our application will run two containers: `web` and `redis`.

The key commands are `docker-compose build`, `docker-compose up`, and `docker-compose down`.