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
