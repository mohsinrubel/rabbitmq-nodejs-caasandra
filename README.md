# rabbitmq-nodejs-caasandra
## Setup Project

**Step 1: Setup**

Before you start, make sure you have Node.js and npm (Node Package Manager) installed on your system. You should also have Nest CLI installed. If you don't have it, you can install it globally using npm:

```bash
npm install -g @nestjs/cli
```

**Step 2: Create a New NestJS Project**

Create a new NestJS project by running the following command:

```bash
nest new api
```

This will create a new NestJS project in a directory called `api` and got to this directory.

**Step 3: Create a Microservice APP**

In NestJS, create microservice app using  the following command:

```bash
nest generate app auth
```

This will generate a new app in the `apps` directory  and there have existing `api` project.

**Step 4: Create Library**

Create  a library  within the microservice Project, The help us to configure common entity, method ect and used this multiple apss:

```bash
nest g library shared
```

This will create a controller file in the `src/microservice` directory.

**Step 5: Install Dependencies**

Install the necessary NestJS packages and RabbitMQ library, Cassandra Driver:

```bash
npm install --save @nestjs/microservices amqplib nestjs-rabbitmq
```


```bash
npm install cassandra-driver
```

## Setup Docker
**Step 1: Create Dockerfile**

Create Dockerfile in every your microservice application

```bash
FROM node

WORKDIR /usr/src/app

COPY package*.json .

RUN npm install

COPY . .

```

**Step 2: Create docker-compose**

Create  docker compos file in root application thats help to run all application

```bash
version: '3'
services:
  rabbitmq:
    image: rabbitmq:3-management
    container_name: rabbitmq
    hostname: rabbitmq
    volumes:
      - /var/lib/rabbitmq
    ports:
      - '5672:5672'
      - '15672:15672'
    networks:
      - server-network
    env_file:
      - .env

  auth:
    build:
      context: ./
      dockerfile: ./apps/auth/Dockerfile
    env_file:
      - .env
    depends_on:
      - rabbitmq
    volumes:
      - .:/usr/src/app # any change to base folder should be reflected
      - /usr/src/app/node_modules
    networks:
      - server-network
    command: npm run start:dev auth # overrides CMD from dockerfile

  api:
    build:
      context: ./
      dockerfile: ./apps/api/Dockerfile
    ports:
      - '4000:5000'
    env_file:
      - .env
    depends_on:
      - rabbitmq
      - auth
    volumes:
      - .:/usr/src/app # any change to base folder should be reflected
      - /usr/src/app/node_modules
    networks:
      - server-network
    command: npm run start:dev api

 
#Docker Networks
networks:
  server-network:
    driver: bridge
    external: true
#Volumes
volumes:
  dbdata:
    driver: local


```

## Setup Env file

Configure ``.env`` file in your root project


```bash
RABBITMQ_DEFAULT_USER=shihab < your rabitmq default user>
RABBITMQ_DEFAULT_PASS= < your rabitmq default password>
RABBITMQ_USER=shihab < your rabitmq user>
RABBITMQ_PASS= < your rabitmq password >
RABBITMQ_HOST=rabbitmq:5672

RABBITMQ_AUTH_QUEUE=auth_queue < your rabitmq que name for auth app>

SCYLLADB_HOST= <scylladb host>
SCYLLADB_USER=<scylladb user>
SCYLLADB_PASSWORD=<scylladb password>
SCYLLADB_KEYSPACE=<scylladb key space name>
SCYLLADB_DATACENTER=<scylladb data center>

```
## Run Application
Run this application using comand
```
docker compose up
```
## Database
### cassandra database setup using docker compose

```
version: '3'
services:
  cassandra:
    image: cassandra:latest
    restart: always
    ports:
      - 9042:9042
      - 7000:7000
    environment:
      - CASSANDRA_BROADCAST_ADDRESS=192.168.1.190
      - CASSANDRA_CLUSTER_NAME=cassandra-cluster
      - CASSANDRA_DC=datacenter1
      - CASSANDRA_ENDPOINT_SNITCH=GossipingPropertyFileSnitch
     # - CASSANDRA_SEEDS=<seed_node_ip1>,<seed_node_ip2>
      - CASSANDRA_USER=my_username
      - CASSANDRA_PASSWORD=my_password
    volumes:
      - ./data:/var/lib/cassandra
    networks:
       - app-network

  #Docker Networks
#docker network create app-network
networks:
  app-network:
    driver: bridge
    external: true
#Volumes
volumes:
  dbdata:
    driver: local
```
