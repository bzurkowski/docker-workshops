
# Managing images

## Outline

In this chapter you will learn:

* What is a Docker image.
* How to list Docker images downloaded into your system.
* How to search for a specific image in a registry.
* How to download an image from a registry.
* How to remove downloaded images.

## Walkthrough

Docker image is a lightweight and standalone **object containing packed application** code, along with its dependencies, configuration, environment variables, and a command starting the application process.

It is type of a **filesystem** that has a **layered structure**. Each increment of files in the image creates a new layer arranged on top of the previous ones. For example, the application code and installation of a library can be two separate layers. This improves the image building process and reduces disk space usage. Most layers are cached and can be reused by other images.

![](/assets/image-layers.png)

When a container is started, the filesystem stored in the image is mounted to the root directory of the container process (under '/' path). Layers of the original image are always read-only. New containers create a thin writable layer (*container layer*) on top of the image layers and use *Copy on Write* technique to add/modify image files:

![](/assets/image-multi-containers.png)

### Task 1: Listing images

Start with listing all Docker images downloaded into your system:

```bash
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
hello-world         latest              fce289e99eb9        4 months ago        1.84kB
```

**Analyze the command output.**

The `hello-world` image has been downloaded when we verified Docker installation using `docker run` command. `docker run` downloads an image automatically before running a container if the image is not available in the system.

### Task 2: Removing images

`hello-world` image is great for testing Docker installation, but not very useful afterwards. Let's remove it to save disk space and ensure an empty slate for further exercises:

```bash
$ docker rmi hello-world
Untagged: hello-world:latest
Untagged: hello-world@sha256:92695bc579f31df7a63da6922075d0666e565ceccad16b59c3374d2cf4e8e50e
Deleted: sha256:fce289e99eb9bca977dae136fbe2a82b6b7d4c372474c9235adc1741675f587e
Deleted: sha256:af0b15c8625bb1938f1d7b17081031f649fd14e6b233688eea3c5483994a66a3
```

Confirm successful removal by listing images again:

```bash
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
```

### Task 3: Searching images

Now, let's say that we are developing an application consiting of three components: a Ruby on Rails website, a Nginx web-server running in front, and a Redis database for storing user data.

We may want to ensure that Docker images required for running those components are available in the public registry - [Docker Hub](https://hub.docker.com/).

Check if there is an image for Redis:

```bash
$ docker search redis
NAME                             DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
redis                            Redis is an open source key-value store that…   6839                [OK]
bitnami/redis                    Bitnami Redis Docker Image                      112                                     [OK]
sameersbn/redis                                                                  76                                      [OK]
grokzen/redis-cluster            Redis cluster 3.0, 3.2, 4.0 & 5.0               45
```

**Analyze the command output.**

Note the columns titled `STARS` and `OFFICIAL`. The former indicates image popularity - the more stars the more users tested and appreciated the image. The latter informs whether an image is maintained by an open source/commercial project or third-party contributors.

Always strive to choose official images. If one does not exist, treat number of stars as the secondary criterion.

Docker images can be also found in the [Docker Hub web browser](https://hub.docker.com/). Try to find images for all application components using the website.

### Task 4: Pulling images

For downloading images found in the registry there is the `docker pull` command. It has the following syntax:

```bash
$ docker pull NAME[:TAG]
```

It takes a reference to an image as its argument. Image `NAME` specifies the application/service that we want to use (e.g. `redis`, `nginx`, `mysql`), whereas optional `TAG` indicates a variant of the application, e.g. version: `v1.2.1`.

You can find available image tags in image documentation on Docker Hub. Take a look at the documentation of [Redis](https://hub.docker.com/_/redis).

First, let's pull an image for Redis without specifying a tag:

```bash
$ docker pull redis
Using default tag: latest
latest: Pulling from library/redis
27833a3ba0a5: Pull complete
cde8019a4b43: Pull complete
97a473b37fb2: Pull complete
c6fe0dfbb7e3: Pull complete
39c8f5ba1240: Pull complete
cfbdd870cf75: Pull complete
Digest: sha256:000339fb57e0ddf2d48d72f3341e47a8ca3b1beae9bdcb25a96323095b72a79b
Status: Downloaded newer image for redis:latest
```

**Analyze the command output.**

Note that by default Docker attempts to download an image tagged as `latest`. This is a global convention. Tag `latest` usually points to the most recent version of an image (and the application installed in that image).

Docker downloads layers of the image one by one. Hashes `27833a3ba0a5`, `cde8019a4b43` etc. constitute unique layer identifiers.

List the images to confirm the successful image pull:

```bash
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
redis               latest              a55fbf438dfd        5 weeks ago         95MB
```

Pulling the latest images without specifying a tag is useful for development and testing purposes, but might be risky in production. Applications are often bound to an interface of a particular servcie version, e.g. our Rails application may use specific Redis interface that is available only till version `4.0.14` and was dropped in newer versions.

Therefore, let's pull the specific version of the Redis image:

```bash
$ docker pull redis:4.0.14
4.0.14: Pulling from library/redis
27833a3ba0a5: Already exists
cde8019a4b43: Already exists
97a473b37fb2: Already exists
c5b25739b664: Pull complete
304d8b43da6c: Pull complete
2d5b26f886bd: Pull complete
Digest: sha256:400c9f87bbe140d594d201ba0aabada8e1c0f6aaf5b54fa889f5d311377b2546
Status: Downloaded newer image for redis:4.0.14
```

Note that this time Docker attempts to pull version `4.0.14` instead of the `latest` one. Also, several image layers have been reused: `27833a3ba0a5`, `cde8019a4b43` and `97a473b37fb2` - the exact same layers have been already downloaded when we pulled the `latest` Redis. In this way Docker facilitates the image download process, as well as saves the disk space.

List the images to confirm the successful image pull:

```bash
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
redis               4.0.14              14433f4e77ab        5 weeks ago         83.4MB
redis               latest              a55fbf438dfd        5 weeks ago         95MB
```

## Exercises

1. Search for images related to at least three technologies that you use on a daily basis (web frameworks, programming languages, databases, etc.):

  ```bash
  $ docker search <your-favourite-tech>
  ```

2. Pull images for the remaining application components: `Nginx` version `1.16.0` and the `latest` version of `Rails` base image.
4. Fetch the list of all downloaded images.
5. Delete image for `Rails`.
