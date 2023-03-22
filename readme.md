
## Setup

# First we'll create our own network
1. docker network create rabbits

# We need Erlang cookie for communication between the nodes.
2. steps to get a cookie 

# will run a standalone instance  
docker run -d --rm --net rabbits --hostname rabbit-1 --name rabbit-1 rabbitmq:3.8
# how to grab existing erlang cookie
docker exec -it rabbit-1 cat /var/lib/rabbitmq/.erlang.cookie
# clean up
docker rm -f rabbit-1

# make sure to give a proper path of your config file.
3. Starting rabbit-1

    ```
    docker run -d --rm --net rabbits \  
    -v ${PWD}/config/rabbit-1/:/config/ \  
    -e RABBITMQ_CONFIG_FILE=/config/rabbitmq \
    -e RABBITMQ_ERLANG_COOKIE=<copied Erlang cookie from step1> \
    --hostname rabbit-1 \
    --name rabbit-1 \
    -p 8081:15672 \
    rabbitmq:3.8-management
    ```

2. Stating rabbit-2

    ```
    docker run -d --rm --net rabbits \  
    -v ${PWD}/config/rabbit-2/:/config/ \
    -e RABBITMQ_CONFIG_FILE=/config/rabbitmq \
    -e RABBITMQ_ERLANG_COOKIE=<copied Erlang cookie from step1> \
    --hostname rabbit-2 \
    --name rabbit-2 \
    -p 8082:15672 \
    rabbitmq:3.8-management
    ```

3. Stating rabbit-3

    ```
    docker run -d --rm --net rabbits \  
    -v ${PWD}/config/rabbit-3/:/config/ \
    -e RABBITMQ_CONFIG_FILE=/config/rabbitmq \
    -e RABBITMQ_ERLANG_COOKIE=<copied Erlang cookie from step1> \
    --hostname rabbit-3 \
    --name rabbit-3 \
    -p 8083:15672 \
    rabbitmq:3.8-management
    
    ```
# Now go to this URL http://localhost:8081 make sure it's working

6. Creating Publisher Image

    ```
    docker build ./applications/publisher -t nilesh/rabbitmq-publisher:v1.0.0
    ```
7. Starting Publisher
    ```
    docker run -it --rm --net rabbits -e RABBIT_HOST=rabbit-1 -e RABBIT_PORT=5672 -e RABBIT_USERNAME=guest -e RABBIT_PASSWORD=guest -p 80:80 nilesh/rabbitmq-publisher:v1.0.0
    ```

8. Creating Consumer Image on new terminal
    ```
    docker build ./applications/consumer -t nilesh/rabbitmq-consumer:v1.0.0
    ```
9. Starting Publisher
    ```
    docker run -it --rm --net rabbits -e RABBIT_HOST=rabbit-1 -e RABBIT_PORT=5672 -e RABBIT_USERNAME=guest -e RABBIT_PASSWORD=guest nilesh/rabbitmq-consumer:v1.0.0
    ```

10. Feeding messages to the queue on new terminal
    ```
    curl -X POST localhost:80/publish/hello
    ```


Management Dashboard can be accessed from browser by these endpoint `http://localhost:8081`, `http://localhost:8082`, `http://localhost:8083`.