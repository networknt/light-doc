---
title: "Proxy Benchmark"
date: 2021-02-11T12:48:27-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

We know the light-proxy is based on the light-4j, which is designed for high throughput, low latency and small memory footprint; however, we still want to have the real performance numbers when using it with the backend APIs. 

There are several questions that we want to answer: 

What is the baseline performance for the backend API?
What is the performance when adding the proxy in front of the backend API?
What is the overhead of each middleware handler in the proxy?
What is the performance difference between HTTP1/1 vs HTTP/2 between the proxy and the backend API.
What is the proxy performance when connecting to the Spring Boot backend API?

### wrk

To generate enough load to test the backend service without middleware handlers, we will use the [wrk][]. With the pipeline.lua script, we can generate millions of request per second. We will use the wrk docker container because some users might not have a Linux desktop to run it. 

To multiply the number of request, we use a script pipeline.lua

pipeline.lua

```
init = function(args)
   request_uri = args[1]
   depth = tonumber(args[2]) or 1

   local r = {}
   for i=1,depth do
     r[i] = wrk.format(nil, request_uri)
   end
   req = table.concat(r)
end

request = function()
   wrk.headers["J-Tenant-Id"] = "1007"
   return req
end

```

To run the docker container in the folder with pipeline.lua file. Notice that we are using the host IP address to access the server instead of localhost. 

```
cd ~/networknt/light-example-4j/proxy
docker run --rm -v `pwd`:/data williamyeh/wrk -t4 -c128 -d30s http://192.168.1.144:8080 -s pipeline.lua --latency -- /v1/pets 16
```

If you are using Linux and have wrk installed natively, you can run the following command in a folder with pipeline.lua file.


```
cd ~/networknt/light-example-4j/proxy
wrk -t4 -c128 -d30s http://localhost:8080 -s pipeline.lua --latency -- /v1/pets 16
```

Note: As we are testing the JVM server, we need to allow the server to warm up before collecting the performance data. So all the wrk test will discard the first round and use the second around. 


### Light-4j Baseline

Before we test the light-proxy, we need to make sure that we have the backend API baseline. As the light-proxy will be used to address all the cross-cutting concerns, we will bypass the middleware handlers in the backend API. 

The plan is to use the [petstore specification](https://github.com/networknt/model-config/blob/master/rest/openapi/petstore/1.0.0/openapi.yaml) to generate a project. At the moment, we don't have an option to generate the project with light-proxy to address the cross-cutting concerns. We are going to use the [petstore][] project with only the handler.yml change to bypass the middleware handlers. In the future, we will add an option to generate projects deployed behind the light-proxy. 

We also want to compare the server in standalone mode and Docker container and give user an idea on the overhead of Docker container.

##### Standalone

We are going to use the [petstore][] project with an external handler.yml and server.yml in light-example-4j/config folder. 

```
cd ~/networknt/light-example-4j/proxy/petstore/http
cp ~/networknt/light-example-4j/rest/openapi/petstore/src/main/resources/config/handler.yml .
cp ~/networknt/light-example-4j/rest/openapi/petstore/src/main/resources/config/server.yml .
```

handler.yml

```
paths:
  - path: '/v1/pets'
    method: 'GET'
    exec:
      # - default
      - com.networknt.petstore.handler.PetsGetHandler

```

We make the modification in the handler.yml to comment out the `default` chain from the path `/v1/pets` so that no middleware handlers will executed before the business handler is invoked. 

server.yml

```
enableHttp: ${server.enableHttp:true}

enableHttps: ${server.enableHttps:false}

enableHttp2: ${server.enableHttp2:false}

```

By default in the [petstore][] porject, we have https and http2 enabled. For this performance test, we are going to enable http only. That's why we make the above change in the server.yml file.

Now, let's build the [petstore][] project in the light-example-4j.

```
cd ~/networknt/light-example-4j/rest/openapi/petstore
mvn clean install -Prelease
```

After the build, a fat jar will be created in the target folder. We are going to use that jar file to start the server.

```
cd ~
java -jar -Dlight-4j-config-dir=networknt/light-example-4j/proxy/petstore/http networknt/light-example-4j/rest/openapi/petstore/target/petstore-3.0.1.jar

```

Run a curl test to make sure the endpoint is working.

```
curl http://localhost:8080/v1/pets
```

And the result:

```
[{"id":1,"name":"catten","tag":"cat"},{"id":2,"name":"doggy","tag":"dog"}]
```

Run the following wrk command from another terminal.

```
cd ~/networknt/light-example-4j/proxy
docker run --rm -v `pwd`:/data williamyeh/wrk -t4 -c128 -d30s http://192.168.1.144:8080 -s pipeline.lua --latency -- /v1/pets 16

```

And the result: 

```
steve@freedom:~/networknt/light-example-4j/proxy$ docker run --rm -v `pwd`:/data williamyeh/wrk -t4 -c128 -d30s http://192.168.1.144:8080 -s pipeline.lua --latency -- /v1/pets 16
Running 30s test @ http://192.168.1.144:8080
  4 threads and 128 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     1.63ms    2.77ms  58.78ms   89.87%
    Req/Sec   610.62k    53.96k  754.10k    71.00%
  Latency Distribution
     50%  564.00us
     75%    1.72ms
     90%    4.45ms
     99%   13.56ms
  72890448 requests in 30.09s, 13.10GB read
Requests/sec: 2422353.50
Transfer/sec:    445.86MB

```

As you can see the backend API can handle the 2.4 million requests per second with average latency of 1.63ms. 

##### Docker HTTP

In this step, we will dockerize the petstore application and then run wrk from the docker. In the [petstore][] project, there is a build.sh and a docker folder with Dockerfile.

I have built the docker image and pushed to the docker hub at https://hub.docker.com/r/networknt/com.networknt.petstore-3.0.1

To start the docker container with the externalized config folder. 

```
cd ~/networknt/light-example-4j/proxy/petstore/http
docker run -d -v `pwd`:/config -p 8080:8080 networknt/com.networknt.petstore-3.0.1
```

Now, let's try the curl command

```
curl http://localhost:8080/v1/pets
[{"id":1,"name":"catten","tag":"cat"},{"id":2,"name":"doggy","tag":"dog"}]
```

Run the wrk

```
cd ~/networknt/light-example-4j/proxy
docker run --rm -v `pwd`:/data williamyeh/wrk -t4 -c128 -d30s http://192.168.1.144:8080 -s pipeline.lua --latency -- /v1/pets 16

```

And the result.

```
steve@freedom:~/networknt/light-example-4j/proxy$ docker run --rm -v `pwd`:/data williamyeh/wrk -t4 -c128 -d30s http://192.168.1.144:8080 -s pipeline.lua --latency -- /v1/pets 16
Running 30s test @ http://192.168.1.144:8080
  4 threads and 128 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     0.99ms    1.08ms  42.69ms   91.42%
    Req/Sec   372.35k    32.59k  475.20k    75.08%
  Latency Distribution
     50%  738.00us
     75%    1.24ms
     90%    1.93ms
     99%    5.04ms
  44481200 requests in 30.09s, 8.00GB read
Requests/sec: 1478098.10
Transfer/sec:    272.06MB

```

As you can see that the docker container will impact the throughput a lot. The number of requests per seconds is dropped from 2.42 million to 1.47 million. 

##### Docker HTTP with Middleware Enabled

Although this benchmark is for the proxy, but it would be very useful to compare with the embedded solution to address cross-cutting concerns so that users can choose between sidecar and embedded patterns based on the performance data.

The generated [petstore][] project has all the middleware handlers defined and enabled but with HTTPS/2 enabled. So we need to create a new embedded folder under proxy/petstore and copy the server.yml from the http folder. After that, we start the server.


```
cd ~/networknt/light-example-4j/proxy/petstore/embedded
docker run -d -v `pwd`:/config -p 8080:8080 networknt/com.networknt.petstore-3.0.1

```

```
curl http://localhost:8080/v1/pets
[{"id":1,"name":"catten","tag":"cat"},{"id":2,"name":"doggy","tag":"dog"}]
```

Run the wrk

```
cd ~/networknt/light-example-4j/proxy
docker run --rm -v `pwd`:/data williamyeh/wrk -t4 -c128 -d30s http://192.168.1.144:8080 -s pipeline.lua --latency -- /v1/pets 16

```

And the result:

```
steve@freedom:~/networknt/light-example-4j/proxy$ docker run --rm -v `pwd`:/data williamyeh/wrk -t4 -c128 -d30s http://192.168.1.144:8080 -s pipeline.lua --latency -- /v1/pets 16
Running 30s test @ http://192.168.1.144:8080
  4 threads and 128 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    18.79ms   18.06ms 183.14ms   86.23%
    Req/Sec    35.29k     9.18k   70.46k    68.11%
  Latency Distribution
     50%   13.38ms
     75%   25.76ms
     90%   42.19ms
     99%   85.18ms
  4218592 requests in 30.07s, 776.47MB read
Requests/sec: 140275.23
Transfer/sec:     25.82MB

```


##### Standalone HTTPS

The above baseline test are using HTTP as most users are using HTTP with sidecar light-proxy in a Kubernetes cluster. For a baseline test, it would be a good idea to compare the HTTP connection to HTTPS/2 with a docker container. It will give users a guideline to choose the connection type between the light-proxy sidecar and the backend API. 

In the `~/networknt/light-example-4j/proxy/petstore` folder we have a http folder for the server.yml with HTTP enabled. Let's create a new folder called `https` and copy only the handler.yml from the `http` folder. The default server.yml in the Docker image have HTTPS/2 enabled. 

```
cd ~/networknt/light-example-4j/proxy/petstore
mkdir https
cd https
cp ../http/handler.yml .
```

With this new config folder, let's start the server again.


```
cd ~
java -jar -Dlight-4j-config-dir=networknt/light-example-4j/proxy/petstore/https networknt/light-example-4j/rest/openapi/petstore/target/petstore-3.0.1.jar

```

On the terminal, you can see that the port is 8443 now.


Run the wrk

```
cd ~/networknt/light-example-4j/proxy
docker run --rm -v `pwd`:/data williamyeh/wrk -t4 -c128 -d30s https://192.168.1.144:8443 -s pipeline.lua --latency -- /v1/pets 16

```

Result:

```
steve@freedom:~/networknt/light-example-4j/proxy$ docker run --rm -v `pwd`:/data williamyeh/wrk -t4 -c128 -d30s https://192.168.1.144:8443 -s pipeline.lua --latency -- /v1/pets 16
Running 30s test @ https://192.168.1.144:8443
  4 threads and 128 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     1.33ms    2.41ms  89.11ms   91.52%
    Req/Sec   494.28k    59.49k  611.55k    88.89%
  Latency Distribution
     50%  556.00us
     75%    1.18ms
     90%    3.23ms
     99%   12.09ms
  58887792 requests in 30.07s, 10.58GB read
Requests/sec: 1958059.64
Transfer/sec:    360.40MB

```

Compare with the HTTP petstore server without docker, the throughput dropped from 2.42 million to 1.96 million per second. And the latency dropped from 1.63ms to 1.33ms on average indicates that the CPU now is the bottleneck not the latency. 


##### Docker HTTPS

Whth the https folder that contains the handler.yml to bypass the middleware handlers, we can start the server with docker and do another test. 

```
cd ~/networknt/light-example-4j/proxy/petstore/https
docker run -d -v `pwd`:/config -p 8443:8443 networknt/com.networknt.petstore-3.0.1

```

And the result.

```
steve@freedom:~/networknt/light-example-4j/proxy$ docker run --rm -v `pwd`:/data williamyeh/wrk -t4 -c128 -d30s https://192.168.1.144:8443 -s pipeline.lua --latency -- /v1/pets 16
Running 30s test @ https://192.168.1.144:8443
  4 threads and 128 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     1.43ms    3.08ms 226.09ms   96.50%
    Req/Sec   285.56k    40.79k  377.54k    82.97%
  Latency Distribution
     50%    0.97ms
     75%    1.67ms
     90%    2.76ms
     99%    7.49ms
  34071200 requests in 30.05s, 6.12GB read
Requests/sec: 1133762.98
Transfer/sec:    208.68MB

```

You can see that the throughput dropped from 1.95 million to 1.13 million when using Docker with HTTPS/2. 


### Spring Boot Baseline

We got four set of data with the above baseline tests with petstore. Now, let's generate a project based on the same specification from the Swagger Editor. To make the test simple, we just test with the HTTP standalone and with docker. 

For performance test, we need to modify the generated project a little bit. Because the Spring Boot project is only used for the proxy testing, we will put it into the light-example-4j/proxy/petstore folder.

In the generated project, we will make the change to the PetsApiController.java

```
    public ResponseEntity<List<Pet>> listPets(@Parameter(in = ParameterIn.QUERY, description = "How many items to return at one time (max 100)" ,schema=@Schema()) @Valid @RequestParam(value = "limit", required = false) Integer limit) {
        String accept = request.getHeader("Accept");
        try {
            return new ResponseEntity<List<Pet>>(objectMapper.readValue("[{\"id\":1,\"name\":\"catten\",\"tag\":\"cat\"},{\"id\":2,\"name\":\"doggy\",\"tag\":\"dog\"}]", List.class), HttpStatus.OK);
        } catch (IOException e) {
            log.error("Couldn't serialize response for content type application/json", e);
            return new ResponseEntity<List<Pet>>(HttpStatus.INTERNAL_SERVER_ERROR);
        }
    }

```

We change the response JSON the same as the Light-4j implementation. Also we change the status to HttpStatus.OK. 

##### Standalone

To run it as a standalone app, you need to have JDK8 installed. I am using SDKMAN to switch the JVM on my desktop. 


```
cd ~/networknt/light-example-4j/proxy/petstore/spring-server-generated
mvn clean install
java -jar target/swagger-spring-1.0.0.jar
```

To confirm that the server is working, issue the following curl command. 


```
curl http://localhost:8080/v1/pets
[{"id":1,"name":"catten","tag":"cat"},{"id":2,"name":"doggy","tag":"dog"}]
```

Run the wrk from the Docker.

```
cd ~/networknt/light-example-4j/proxy
docker run --rm -v `pwd`:/data williamyeh/wrk -t4 -c128 -d30s http://192.168.1.144:8080 -s pipeline.lua --latency -- /v1/pets 16

```

And the result:

```
steve@freedom:~/networknt/light-example-4j/proxy$ docker run --rm -v `pwd`:/data williamyeh/wrk -t4 -c128 -d30s http://192.168.1.144:8080 -s pipeline.lua --latency -- /v1/pets 16
Running 30s test @ http://192.168.1.144:8080
  4 threads and 128 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    34.45ms   30.99ms 407.58ms   78.26%
    Req/Sec    23.57k     2.91k   55.33k    72.79%
  Latency Distribution
     50%   27.39ms
     75%   48.82ms
     90%   71.10ms
     99%  142.43ms
  2812140 requests in 30.10s, 571.76MB read
Requests/sec:  93439.50
Transfer/sec:     19.00MB

```

##### Docker

The generated spring project doesn't have a Dockerfile, so we add one. 

Dockerfile

```
FROM openjdk:8-jre-alpine
ADD /target/swagger-spring-1.0.0.jar server.jar
CMD ["/bin/sh","-c","java -XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap -XX:MaxRAMFraction=1 -jar /server.jar"]
```

Also, add a build.sh to build the docker image and push to docker hub.

build.sh

```


#!/bin/bash

set -ex

VERSION=$1
IMAGE_NAME="networknt/springboot-petstore"

showHelp() {
    echo " "
    echo "Error: $1"
    echo " "
    echo "    build.sh [VERSION]"
    echo " "
    echo "    where [VERSION] version of the docker image that you want to publish (example: 0.0.1)"
    echo " "
    echo "    example: ./build.sh 0.0.1"
    echo " "
}

build() {
    echo "Building ..."
    mvn clean install
    echo "Successfully built!"
}

cleanup() {
    if [[ "$(docker images -q $IMAGE_NAME 2> /dev/null)" != "" ]]; then
        echo "Removing old $IMAGE_NAME images"
        docker images | grep $IMAGE_NAME | awk '{print $3}' | xargs docker rmi -f
        echo "Cleanup completed!"
    fi
}

publish() {
    echo "Building Docker image with version $VERSION"
    docker build -t $IMAGE_NAME:$VERSION -t $IMAGE_NAME:latest -f Dockerfile . --no-cache=true
    echo "Images built with version $VERSION"
    echo "Pushing image to DockerHub"
    docker push $IMAGE_NAME
    echo "Image successfully published!"
}

if [ -z $VERSION ]; then
    showHelp "[VERSION] parameter is missing"
    exit
fi

build;
cleanup;
publish;

```

With the docker image pushed to docker hub. Run the following command to start the Spring Boot Petstore.

```
docker run -d -p 8080:8080 networknt/springboot-petstore
```

Run the wrk

```
cd ~/networknt/light-example-4j/proxy
docker run --rm -v `pwd`:/data williamyeh/wrk -t4 -c128 -d30s http://192.168.1.144:8080 -s pipeline.lua --latency -- /v1/pets 16

```

And the result.

```
steve@freedom:~/networknt/light-example-4j/proxy$ docker run --rm -v `pwd`:/data williamyeh/wrk -t4 -c128 -d30s http://192.168.1.144:8080 -s pipeline.lua --latency -- /v1/pets 16
Running 30s test @ http://192.168.1.144:8080
  4 threads and 128 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    44.55ms   43.48ms 530.06ms   85.89%
    Req/Sec    20.68k     2.84k   35.22k    68.86%
  Latency Distribution
     50%   32.96ms
     75%   66.43ms
     90%   99.83ms
     99%  194.75ms
  2469276 requests in 30.10s, 502.05MB read
Requests/sec:  82037.22
Transfer/sec:     16.68MB

```

The throughput dropped from 93K to 82K as expected. 


### Baseline Summary

| Test Case                 | Max Throughput | Avg Latency | Transfer | 
| ------------------------: | -------------: | ----------: | -------: |
| Light-4j Standalone       | 2,422,353      | 1.63ms      | 445.86MB |
| Spring Boot Standalone    | 93,439         | 34.45ms     | 19.00MB  |
| Light-4j Docker           | 1,478,098      | 0.99ms      | 272.06MB |
| Light-4j Docker with CCC  | 140,275        | 18.79ms     | 25.82MB  |
| Spring Boot Docker        | 82,037         | 44.55ms     | 16.68MB  |
| Light-4j HTTPS Standalone | 1,958,059      | 1.33ms      | 360.40MB |
| Light-4j HTTPS Docker     | 1,133,762      | 1.43ms      | 208.68MB |


### Light-proxy Config

The light-proxy is published to the docker hub for each release, so it is readily available. We need to provide the externalized config files though.

```
cd ~/networknt/light-example-4j/proxy/config
mkdir http
```

We create a folder named `http` and later we will create another one called `https` to test with HTTPS/2 between the proxy and backend API.

As there is no default config files in the light-proxy docker image required by our customers. We need to copy from the test resources from the light-proxy repository. After that we need to make some modifications.

We need to copy the petstore openapi.yml for security and validation.



### Light-4j with Proxy

With both Light-4j and Spring Boot implementations are Dockerized, we can start use the light-proxy docker image with externalized config folder in front of the backend API. 

```
docker-compose -f docker-compose-light-4j.yml up
```

run wrk on port 8000

```
cd ~/networknt/light-example-4j/proxy
docker run --rm -v `pwd`:/data williamyeh/wrk -t4 -c128 -d30s http://192.168.1.144:8000 -s pipeline.lua --latency -- /v1/pets 16

```

Result:

```
steve@freedom:~/networknt/light-example-4j/proxy$ docker run --rm -v `pwd`:/data williamyeh/wrk -t4 -c128 -d30s http://192.168.1.144:8000 -s pipeline.lua --latency -- /v1/pets 16
Running 30s test @ http://192.168.1.144:8000
  4 threads and 128 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    11.51ms    7.36ms 116.40ms   68.34%
    Req/Sec    27.07k     2.98k   41.92k    84.14%
  Latency Distribution
     50%   12.43ms
     75%   19.25ms
     90%    0.00us
     99%    0.00us
  3228224 requests in 30.05s, 594.18MB read
Requests/sec: 107411.98
Transfer/sec:     19.77MB

```

### Spring Boot with Proxy

Start the docker-compose with Spring Boot backend.

```
docker-compose -f docker-compose-spring-boot.yml up
```

Run wrk

```
cd ~/networknt/light-example-4j/proxy
docker run --rm -v `pwd`:/data williamyeh/wrk -t4 -c128 -d30s http://192.168.1.144:8000 -s pipeline.lua --latency -- /v1/pets 16

```

And the result.

```
steve@freedom:~/networknt/light-example-4j/proxy$ docker run --rm -v `pwd`:/data williamyeh/wrk -t4 -c128 -d30s http://192.168.1.144:8000 -s pipeline.lua --latency -- /v1/pets 16
Running 30s test @ http://192.168.1.144:8000
  4 threads and 128 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    28.96ms   14.42ms 130.54ms   55.10%
    Req/Sec     9.61k     1.17k   13.46k    72.45%
  Latency Distribution
     50%   38.16ms
     75%   54.65ms
     90%    0.00us
     99%    0.00us
  1147034 requests in 30.10s, 247.43MB read
Requests/sec:  38109.76
Transfer/sec:      8.22MB

```


### Light-4j with Proxy HTTPS

This is a similar test with Light-4j with Proxy but enable HTTPS/2 between the proxy and the backend API. We want to see if the HTTPS/2 can impact the performance or not.

copy the config files from config/http to config/https and make the modification for the proxy.yml file.

```
# If HTTP 2.0 protocol will be used to connect to target servers
http2Enabled: true

# If TLS is enabled when connecting to the target servers
httpsEnabled: true

# Target URIs
hosts: https://192.168.1.144:8443

```

As you can see, we set the proxy to connnect to the backend API with httpsEnabled and http2Enabled to true. And update the url to port 8443. 

Create a docker-compose file

docker-compose-l4j-https.yml

```
version: "3"

services:
  petstore:
    image: networknt/com.networknt.petstore-3.0.1
    volumes:
    - ./petstore/https:/config
    network_mode: host
  proxy:
    image: networknt/light-proxy
    volumes:
    - ./config/https:/config
    network_mode: host

```

We use the petstore/https config folder to start the backend API with port 8443. For the proxy, we use the config/https for the config.

Start the docker-compose

```
docker-compose -f docker-compose-l4j-https.yml up
```

Run the wrk and the result:

```
steve@freedom:~/networknt/light-example-4j/proxy$ docker run --rm -v `pwd`:/data williamyeh/wrk -t4 -c128 -d30s http://192.168.1.144:8000 -s pipeline.lua --latency -- /v1/pets 16
Running 30s test @ http://192.168.1.144:8000
  4 threads and 128 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    14.01ms    9.34ms 199.19ms   71.79%
    Req/Sec    22.34k     2.52k   29.28k    86.56%
  Latency Distribution
     50%   13.34ms
     75%   20.02ms
     90%   28.01ms
     99%    0.00us
  2665184 requests in 30.05s, 490.55MB read
Requests/sec:  88681.46
Transfer/sec:     16.32MB

```

The result is a slower than the HTTP connection between proxy and backend.

### Proxy Summary

| Test Case                 | Max Throughput | Avg Latency | Transfer | 
| ------------------------: | -------------: | ----------: | -------: |
| Light-4j Standalone       | 2,422,353      | 1.63ms      | 445.86MB |
| Spring Boot Standalone    | 93,439         | 34.45ms     | 19.00MB  |
| Light-4j Docker           | 1,478,098      | 0.99ms      | 272.06MB |
| Light-4j Docker with CCC  | 140,275        | 18.79ms     | 25.82MB  |
| Spring Boot Docker        | 82,037         | 44.55ms     | 16.68MB  |
| Light-4j HTTPS Standalone | 1,958,059      | 1.33ms      | 360.40MB |
| Light-4j HTTPS Docker     | 1,133,762      | 1.43ms      | 208.68MB |
| Light-4j with Proxy       | 107,411        | 11.51ms     | 19.77MB  |   
| Light-4j with Proxy HTTPS | 88,681         | 14.01ms     | 16.32MB  |
| Spring Boot with Proxy    | 38,109         | 28.96ms     | 8.22MB   |
  


### Conclusion

| Test Case                 | Max Throughput | Avg Latency | Transfer | 
| ------------------------: | -------------: | ----------: | -------: |
| Light-4j Standalone       | 2,422,353      | 1.63ms      | 445.86MB |
| Spring Boot Standalone    | 93,439         | 34.45ms     | 19.00MB  |
| Times                     | 26             | 21          | 23       |

Compare the standalone mode between Light-4j and Spring Boot petstore implementations on Linux System, we can see  
the thoughput and transfer rate per second is over 20 times different. The light-4j latency is also more than 20 times lower. This test doesn't have any cross-cutting concerns addressed and it is not reflect the normal production scenario; however, it gives us some ideas on the overhead of each framework and its potential. 


| Test Case                 | Max Throughput | Avg Latency | Transfer | 
| ------------------------: | -------------: | ----------: | -------: |
| Light-4j Standalone       | 2,422,353      | 1.63ms      | 445.86MB |
| Spring Boot Standalone    | 93,439         | 34.45ms     | 19.00MB  |
| Times                     | 26             | 21          | 23       |


| Test Case                 | Max Throughput | Avg Latency | Transfer | 
| ------------------------: | -------------: | ----------: | -------: |
| Light-4j Docker           | 1,478,098      | 0.99ms      | 272.06MB |
| Spring Boot Docker        | 82,037         | 44.55ms     | 16.68MB  |
| Times                     | 18             | 45          | 16       |


Compare the Dockerized petstore implementations between Light-4j and Spring Boot, you can see the diffence is getting smaller. This is due to the overhead Docker added to the applications. Although Docker offers a lot of important features, it does add overhead than running the application standalone. Compare with the standalone version, you can see that Light-4j throughput is dropped from 2.4 million to 1.5 million which is very significant. For Spring Boot, it is less impact compare beteen 9.3k to 8.2k. 

A lot of users might ask why the latency for light-4j is reduced from 1.63ms to 0.99ms. It is due to the throughput is dropped from 2.4m to 1.4m so the latency is reduce. The Light-4j separates the IO threads and Work threads so it can use CPU up to 100%. When the througput is reduced, the latency will be smaller. Spring Boot is using Servlet engine and one thread per request so the CPU is waiting for the IO most of the time. 

| Test Case                 | Max Throughput | Avg Latency | Transfer | 
| ------------------------: | -------------: | ----------: | -------: |
| Light-4j Docker with CCC  | 140,275        | 18.79ms     | 25.82MB  |
| Light-4j with Proxy       | 107,411        | 11.51ms     | 19.77MB  |   
| Spring Boot with Proxy    | 38,109         | 28.96ms     | 8.22MB   |
  
When using the light-proxy to address the cross-cutting concerns, the throughput difference is further reduced to 107k vs 38k and the both latencies are reduced to 11.51 vs 28.96 due to the reduced throughput.

Also, when compare with the embedded ligth-4j from proxy, we can see that the throughput is 140k vs 107k. This is not as significant as I was expected. I thought the embedded approach would perform much better. This actually give us the reason to use the light-proxy sidecar in the cloud. 


| Test Case                 | Max Throughput | Avg Latency | Transfer | 
| ------------------------: | -------------: | ----------: | -------: |
| Light-4j Standalone       | 2,422,353      | 1.63ms      | 445.86MB |
| Light-4j HTTPS Standalone | 1,958,059      | 1.33ms      | 360.40MB |
| Light-4j HTTPS Docker     | 1,133,762      | 1.43ms      | 208.68MB |
| Light-4j with Proxy       | 107,411        | 11.51ms     | 19.77MB  |   
| Light-4j with Proxy HTTPS | 88,681         | 14.01ms     | 16.32MB  |

A lot of people ask if HTTPS will impact the performance and the above data confirms it. It redueces the throughput but also reduce the latency if it is standalone. When using the proxy, the throughput is reduced about 20% and the latency is increased. 


### Limitations

The tests were performed on my desktop with the following configurations. 

CPU: Ryzen 2700 8 Core
Memory: 32GB
OS: Ubuntu 20.04

As I have dockerized the backend APIs, light-proxy and wrk, everyone can test in their network environment. It would be very interested in seeing the test done in a Kubernetes cluster. 

If anyone has some ideas for more test scenarios, please let me know.  

[petstore]: https://github.com/networknt/light-example-4j/tree/release/rest/openapi/petstore
[wrk]: /tool/wrk-perf/





