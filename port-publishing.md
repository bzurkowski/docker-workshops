# Port publishing

## Outline

In this chapter you will learn:

* Why do we need to publish container ports.
* How to publish container ports.

## Walkthrough

By default containers do not publish any ports to the outside world. Ports are opened only for internal container-to-container or host-to-container communication in a Docker network. In order to access container ports from the outside of the Docker network, e.g. via public host IP or host loopback address, ports must be published explicitly via `docker run` command.

### Task 1: Why do we need publish container ports

The objective of this task is to ilustrate the problem of publishing ports in containers.

Let's run a container with Nginx:

```bash
$ docker run --name noportpub -d --rm nginx
```

List the created container:

```bash
$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS               NAMES
dc8defc10363        nginx               "nginx -g 'daemon of…"   11 seconds ago       Up 9 seconds        80/tcp              noportpub
```

**Analyze the command output.**

Note that the container exposes port `80` for internal `HTTP` communication (`PORTS` column).

Now, fetch the IP of the container in the Docker network:

```bash
$ docker inspect --format '{{ .NetworkSettings.IPAddress }}' noportpub
172.18.0.2
```

Try to communicate with the Nginx server using obtained internal container IP:

```bash
$ curl http://172.18.0.2:80
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
...
```

The communication succeeds and Nginx server responds with a welcome message. This is because the communication happend in the scope of the internal Docker network.

Subsequently, let's try to communicate with the server via loopback address which is an address from the outside of the Docker network:

```bash
$ curl http://localhost:80
curl: (7) Failed to connect to localhost port 80: Connection refused
```

The connection has been refused, because default firewall rules prevent communication with port `80` in the container from the outside of the Docker network.

### Task 2: Publishing container ports

In order to publish a container port there is `--publish` flag for `docker run` command. It has the following syntax:

```bash
--publish HOST_PORT:CONTAINER_PORT
```

It defines a mapping between `HOST_PORT` and `CONTAINER_PORT`. Multiple instances of this option can be provided to define mapping for multiple container ports.

As an example, let's run a container with Nginx and publish port `80` to the outside:

```bash
$ docker run --name portpub -d --rm --publish 8080:80 nginx
```

We've mapped port `8080` of the Docker host to port `80` of the container. This means, that all communication to port `8080` on the host will be forwarded to port `80` in the container. Also, appropriate firewall rules are set to permit the communication.

Once again try to communicate with the Nginx server from the Docker host:

```
$ curl http://localhost:8080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
...
```

This time the communication succeed. You may also try to access Nginx via the web browser by entering the same URL.

## Exercises

1. Run a `Redis` container with port `6379` exposed to the outside - map port `6000` of the Docker host to port `6379` of the container.
2. Connect to containerized Redis instance via loopback address and try to set several keys:

  ```bash
  $ redis-cli -h localhost -p 6000
  ```
