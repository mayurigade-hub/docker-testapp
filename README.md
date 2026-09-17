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

