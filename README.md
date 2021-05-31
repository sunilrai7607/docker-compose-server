# docker-compose-server
docker-compose for all mandatory services like mongodb, redis, kafka, rabbitMQ

docker-compose having following server configuration. 

* mongoDB
* redis
* kafka
* rabbitMQ

Run below commands to start all server in docker container under same network. Connect your springboot service as requirement. 

```commandline
## Build and start the container
 docker-compose up --build
## up specific service
docker-compose up --build $(<services.txt)

services1 services2, etc
 docker-compose up --build zookeeper kafka



## Stop and cleanup
Command+c in MAC machine
 docker-compose down
``` 

```commandline
#Mongo cli
docker exec -it <container-id> bash
root@localhost:/# mongo
root@localhost:/# db.auth('<user>','<pass>')
```

```yaml
# Redis
  redis-server:
    image: redis
    command: [ "redis-server", "--protected-mode", "no" ]
    hostname: localhost
    ports:
      - "6379:6379"
    networks:
      - spring-boot-profile-network
```

```yaml
# Mongodb
  mongodb-server:
    image: mongo
    restart: always
    hostname: localhost
    environment:
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: example
    ports:
      - "27017:27017"
    networks:
      - spring-boot-profile-network
```

```yaml
# RabbitMQ
  rabbitmq-server:
    image: rabbitmq:3-management
    hostname: localhost
    volumes:
      - ./.docker/rabbitmq/etc/:/etc/rabbitmq/
      - ./.docker/rabbitmq/data/:/var/lib/rabbitmq/
      - ./.docker/rabbitmq/logs/:/var/log/rabbitmq/
    environment:
      RABBITMQ_ERLANG_COOKIE: "SWQOKODSQALRPCLNMEQG"
      RABBITMQ_DEFAULT_USER: "rabbitmq"
      RABBITMQ_DEFAULT_PASS: "rabbitmq"
    ports:
      - "15672:15672"
      - "5672:5672"
    networks:
      - spring-boot-profile-network
```

```yaml
# Zookeeper and Kafka
  zookeeper:
    image: wurstmeister/zookeeper
    container_name: zookeeper
    ports:
      - "2181:2181"
    networks:
      - spring-boot-profile-network
  kafka:
    image: wurstmeister/kafka
    ports:
      - "9092:9092"
    environment:
      KAFKA_ADVERTISED_HOST_NAME: localhost
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
    depends_on:
      - zookeeper
    networks:
      - spring-boot-profile-network
```
## Open Source Tools for Distributed Tracing

* _**OpenTracing**_ is one such tool that helps standardize tracing, thus making the process of instrumenting an application for distributed tracing generic. Since the implementation and integration are quite simple, teams should be able to set up and start generating value from OpenTracing on day one. But to do this, organizations also need distributed traces that are OpenTracing-compatible

* _**Zipkin**_ was one of the first tracing systems developed according to Google’s Dapper, and its architecture is quite simple.
  Every operation performed in the application that is being traced is usually generated on the client-side and starts
  with an outgoing http request. Several headers are added to this request in order to create unique IDs that are traceable along
  with the application. The Reporter is the component that is implemented inside each host of the tracked application
  and is in charge of sending the data to Zipkin. Reporters then send all traces to Zipkin collectors, which persist the data to databases.
You can use either Cassandra or Elasticsearch for a scalable storage backend, as Zipkin supports them both.
This storage is queried by the API used by the Zipkin UI.

```commandline
docker run -d -p 9411:9411 openzipkin/zipkin
```

* **_jaeger_**: was developed by Uber and was shared as an open-source project once it was production-ready.
  Both implement a trace collector, and the different APIs send events/traces to this trace collector.
  The collector then records the data and the relation between the different traces.
  Both tools also provide a user interface for inspecting and following traces. Jaeger is similar to Zipkin but has a different implementation. Supported by the Cloud Native Computing Foundation (CNCF) as an incubating project,
  Jaeger implements the OpenTracing specification to the last API, and its preferred deployment method is actually Kubernetes.
  Jaeger is built with components that resemble other tracing systems, with collectors, a query service, and a UI.
  Jaeger also deploys an agent on every host that locally aggregates the data and sends it to the collector.
  From there, the data passes through the same flow–to be stored in Elasticsearch or Cassandra and even ScyllaDB–and is then queried
  and delivered to the UI. <br> As on-the-ground microservice practitioners are quickly realizing, the majority of operational problems that arise when moving to a
distributed architecture are ultimately grounded in two areas: networking and observability. It is simply an orders of
magnitude larger problem to network and debug a set of intertwined distributed services versus a single monolithic application.
  
```yaml
version: '3'
services:
  zipkin:
    image: openzipkin/zipkin
    container_name: zipkin
    ports:
    - 9411:9411
    networks:
      - trace
networks:
  trace:
    driver: bridge
```
Run below command to start zipkin
```commandline
docker compose up zipkin
```
```commandline
Attaching to zipkin
zipkin    |                   oo
zipkin    |                  oooo
zipkin    |                 oooooo
zipkin    |                oooooooo
zipkin    |               oooooooooo
zipkin    |              oooooooooooo
zipkin    |            ooooooo  ooooooo
zipkin    |           oooooo     ooooooo
zipkin    |          oooooo       ooooooo
zipkin    |         oooooo   o  o   oooooo
zipkin    |        oooooo   oo  oo   oooooo
zipkin    |      ooooooo  oooo  oooo  ooooooo
zipkin    |     oooooo   ooooo  ooooo  ooooooo
zipkin    |    oooooo   oooooo  oooooo  ooooooo
zipkin    |   oooooooo      oo  oo      oooooooo
zipkin    |   ooooooooooooo oo  oo ooooooooooooo
zipkin    |       oooooooooooo  oooooooooooo
zipkin    |           oooooooo  oooooooo
zipkin    |               oooo  oooo
zipkin    |      ________ ____  _  _____ _   _
zipkin    |     |__  /_ _|  _ \| |/ /_ _| \ | |
zipkin    |       / / | || |_) | ' / | ||  \| |
zipkin    |      / /_ | ||  __/| . \ | || |\  |
zipkin    |     |____|___|_|   |_|\_\___|_| \_|
zipkin    | :: version 2.23.2 :: commit 7bf3aab ::
zipkin    | 2021-05-31 11:31:19.174  INFO [/] 1 --- [oss-http-*:9411] c.l.a.s.Server                           : Serving HTTP at /0.0.0.0:9411 - http://<ipaddress>:9411/
```