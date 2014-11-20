Docker Client [![Circle CI](https://circleci.com/gh/spotify/docker-client.png?style=badge)](https://circleci.com/gh/spotify/docker-client)
=============

This is a simple [Docker](https://github.com/dotcloud/docker) client written in Java.

Usage
-----

```java
// Create a client based on DOCKER_HOST and DOCKER_CERT_PATH env vars
final DockerClient docker = DefaultDockerClient.fromEnv().build();

// Pull image
docker.pull("busybox");

// Create container
final ContainerConfig config = ContainerConfig.builder()
    .image("busybox")
    .cmd("sh", "-c", "while :; do sleep 1; done")
    .build();
final ContainerCreation creation = docker.createContainer(config);
final String id = creation.id();

// Inspect container
final ContainerInfo info = docker.inspectContainer(id);

// Start container
docker.startContainer(id);

// Kill container
docker.killContainer(id);

// Remove container
docker.removeContainer(id);
```

### Unix socket support

Unix socket support is available on Linux since v2.5.0:

```java
final DockerClient docker = new DefaultDockerClient("unix:///var/run/docker.sock");
```

### HTTPS support

We can connect to [HTTPS-secured Docker instances](https://docs.docker.com/articles/https/)
with client-server authentication. The semantics are similar to using [the `DOCKER_CERT_PATH`
environment variable](https://docs.docker.com/articles/https/#client-modes):

```java
final DockerClient docker = new DefaultDockerClient.builder()
    .uri(URI.create("https://boot2docker:2376"))
    .dockerCertificates(new DockerCertificates("/Users/rohan/.docker/boot2docker-vm/"))
    .build();
```

### Connection pooling

We use the [Apache HTTP client](https://hc.apache.org/) under the covers, with a shared connection
pool per instance of the Docker client. The default size of this pool is 100 connections, so each
instance of the Docker client can only have 100 concurrent requests in flight.

If you plan on waiting on more than 100 containers at a time (`DockerClient.waitContainer`), or
otherwise need a higher number of concurrent requests, you can modify the connection pool size:

```java
final DockerClient docker = DefaultDockerClient.fromEnv()
    .connectionPoolSize(SOME_LARGE_NUMBER)
    .build()
```

Note that the connect timeout is also applied to acquiring a connection from the pool. If the pool
is exhausted and it takes too long to acquire a new connection for a request, we throw a
`DockerTimeoutException` instead of just waiting forever on a connection becoming available.

Maven
-----

```xml
<dependency>
  <groupId>com.spotify</groupId>
  <artifactId>docker-client</artifactId>
  <version>2.7.1</version>
</dependency>
```

Testing
-------

If running the tests are slow, like on the order of tens of minutes instead of the expected minutes,
check how many stopped containers you have with `docker ps -a`. If you have a lot, remove them
with `docker rm $(docker ps -a -q)`. Your tests should run faster now.


Releasing
---------

```sh
mvn release:clean
mvn release:prepare
mvn release:perform
```
