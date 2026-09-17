# docker-testapp

# Setting up MongoDB + Mongo Express Containers

## 1. Create Docker Network

Create a custom network so the MongoDB and Mongo Express containers can communicate with each other.

```bash
docker network create mongo-network
```

Check the network:

```bash
docker network ls
```

![Docker Network](assets/step-1.png)


## 2. Run MongoDB Container

Run MongoDB inside a Docker container:

```bash
docker run -d -p 27017:27017 --name mongo --network mongo-network -e MONGO_INITDB_ROOT_USERNAME=admin -e MONGO_INITDB_ROOT_PASSWORD=<your-password> mongo
```

### What we used

- `-d` → run in background
- `-p 27017:27017` → map MongoDB port
- `--name mongo` → container name
- `--network mongo-network` → connect to our custom network
- `-e` → set environment variables

![MongoDB Container](assets/step-2.png)


## 3. Run Mongo Express

Run Mongo Express and connect it to the same network:

```bash
docker run -d -p 8081:8081 --name mongo-express --network mongo-network -e ME_CONFIG_MONGODB_ADMINUSERNAME=admin -e ME_CONFIG_MONGODB_ADMINPASSWORD=<your-password> -e ME_CONFIG_MONGODB_URL="mongodb://admin:<your-password>@mongo:27017/?authSource=admin" mongo-express
```

The important part is:

```text
mongodb://admin:<your-password>@mongo:27017/
```

Here, `mongo` is the name of the MongoDB container.

![Mongo Express Container](assets/step-3.png)


## 4. Open Mongo Express

Open this URL in the browser:

```text
http://localhost:8081
```

Mongo Express provides a web interface to view and manage the MongoDB databases.

![Mongo Express](assets/step-4.png)


## 5. View MongoDB Data

Using Mongo Express, we can view databases, collections, and documents stored in MongoDB.

![MongoDB Collection](assets/step-5.png)

---

Par itni mehanat kyu karni hai, use Docker Compose Bro!
# Docker Compose

Docker Compose is used to **define and run multiple Docker containers** using a YAML file.

Instead of writing multiple long `docker run` commands, we can define the setup once in:

```text
docker-compose.yml
```

## Why Docker Compose?

It makes multi-container applications:

- Easier to configure
- Faster to start
- Easier to reproduce

## Example

For our MongoDB + Mongo Express setup:

```yaml
services:
  mongo:
    image: mongo
    ports:
      - "27017:27017"
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: <your-password>

  mongo-express:
    image: mongo-express
    ports:
      - "8081:8081"
    environment:
      ME_CONFIG_MONGODB_ADMINUSERNAME: admin
      ME_CONFIG_MONGODB_ADMINPASSWORD: <your-password>
      ME_CONFIG_MONGODB_URL: mongodb://admin:<your-password>@mongo:27017/?authSource=admin
```

Here, `mongo` is the service name used by Mongo Express to connect to MongoDB.

## Important Commands

### Start

```bash
docker compose -f fileName.yaml up -d
```

### Stop

```bash
docker compose -f fileName.yaml down
```


## Key Takeaway

**Docker Compose = define multiple containers in YAML + manage them with simple commands.**

---

**NEXT IMPORTANT TOPIC**
## Dockerizing the Node.js Application

Dockerizing means packaging the Node.js application so it can run inside a Docker container.

### Dockerfile

Create a file named `Dockerfile` in the project root:

```dockerfile
FROM node

ENV MONGO_DB_USERNAME=admin \
    MONGO_DB_PWD=qwerty

RUN mkdir -p testapp

COPY . /testapp

CMD ["node", "/testapp/server.js"]
```

### Dockerfile Process

```text
FROM  → Get Node.js environment
ENV   → Set environment variables
RUN   → Create /testapp directory
COPY  → Copy project files into /testapp
CMD   → Start server.js
```

Important:

```text
RUN → executes during image build
CMD → executes when the container starts
```

### Build the Docker Image

```bash
docker build -t docker-testapp .
```

This creates the Docker image:

```text
docker-testapp
```

### Run the Docker Image
```bash
docker run -d -p 5050:5050
```
---

### BUT YOU CAN GET THE ERROR IN MONGODB CONNECTION
### MongoDB Connection

Before Dockerizing:

```javascript
const MONGO_URL = "mongodb://admin:qwerty@localhost:27017";
```

After Dockerizing, `localhost` refers to the Node.js container itself.

So we changed it to:

```javascript
const MONGO_URL = "mongodb://admin:qwerty@mongo:27017";
```

Here, `mongo` is the name of the MongoDB container.

### Run the MongoDB Container

MongoDB must be running before starting the Node.js container:

```bash
docker run -d -p 27017:27017 --name mongo --network mongo-network -e MONGO_INITDB_ROOT_USERNAME=admin -e MONGO_INITDB_ROOT_PASSWORD=qwerty mongo
```

### Create and Run the Node.js Container

```bash
docker run -d -p 5050:5050 --network mongo-network --name node-app docker-testapp
```

This creates and starts the Node.js container from the `docker-testapp` image.

### Test the Application

Open:

```text
http://localhost:5050
```

API:

```text
http://localhost:5050/getUsers
```

### Key Takeaway

```text
Dockerfile → Docker Image → Container → Node.js App
```

The Node.js container communicates with the MongoDB container through the shared `mongo-network`.


