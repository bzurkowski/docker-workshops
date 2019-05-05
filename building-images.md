# Preparing Dockerfiles

## Outline

In this chapter you will learn:

* How to prepare a Dockerfile for an application.
* How to build an image from a Dockerfile.
* How to tag images with new labels.

## Walkthrough

### Task 1: Sample application overview

Start with cloning the repository with a sample Python web application based on [Flask](http://flask.pocoo.org/) framework:

```bash
$ git clone https://github.com/bzurkowski/docker-workshops.git
```

If `git` command is not available, install it:

```bash
$ apt install -y git-core
```

Application is located in `sample-app` subdirectory:

```bash
$ cd ./docker-workshops/sample-app
$ ls -l
-rw-r--r--  1 bzurkowski  staff    6 May  5 12:59 requirements.txt
-rw-r--r--  1 bzurkowski  staff  189 May  5 13:00 sample_app.py
```

The application is very simple and consists of two files: `sample_app.py` and `requirements.txt`.

File `sample_app.py` contains the code of the application - Flask `Hello world!` example. It starts a server listening on port `5000`, and exposes a single `HTTP` endpoint under `/` which just prints `Hello World!`:

```python
from flask import Flask

app = Flask(__name__)

@app.route("/")
def hello():
    return "Hello World!"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=int("5000"), debug=True)
```

File `requirements.txt` contains list of application dependencies. At the moment it's only the `flask` library:

```
flask
```

### Task 2: Preparing a Dockerfile

In order to package our sample application into a Docker image, we need to write a `Dockerfile`.

Dockerfile is a text file containing all instructions that the Docker engine needs to know in order to prepare the application environment:

* a base Docker image to run from,
* copying application code,
* installing application dependencies,
* application start-up commands.

It is a convenient way to automate the image creation process. A ready Dockerfile may be passed as an argument for `docker build` command, which will process the instructions and provide a ready-to-run image object.

Here is the format of a Dockerfile:

```dockerfile
# Comment
DIRECTIVE arguments
```

Docker runs the directives in order. A Dockerfile **must** start with a `FROM` directive, which specifies a base image from which a new image should be built (e.g. `ubuntu`).

Here are the most common directives that can be used in a Dockerfile:

* `FROM <image>[:<tag>]`

  Sets a base Image for subsequent instructions.

  Examples:

  ```dockerfile
  FROM ruby:2.3.1
  FROM ubuntu
  ```

* `WORKDIR <path>`

  Sets the working directory for any `RUN`, `CMD`, `ENTRYPOINT`, `COPY` and `ADD` instructions that follow it in the Dockerfile.

  Examples:

  ```dockerfile
  WORKDIR /my/custom/path
  RUN pwd
  ```

  The output of the final `pwd` command in the above example would be `/my/custom/path`

* `COPY <src>... <dest>`

  Copies new files or directories from `<src>` and adds them to the filesystem of the image at the path `<dest>`.

  Examples:

  ```dockerfile
  COPY test relativeDir/
  COPY hom* /mydir/
  ```

* `RUN <command>`

  Executes any commands in a new layer on top of the current image and commits the results.

  Examples:

  ```dockerfile
  RUN apt-get update && \
      apt-get install redis && |
      ...
  ```

* `ENV <key> <value>`

  Sets the environment variable `<key>` to the value `<value>`.

  Examples:

  ```dockerfile
  ENV REDIS_VERSION 5.0.4
  ```

* `EXPOSE <port> [<port>/<protocol>...]`

  Informs Docker that the container listens on the specified network ports at runtime. Acts as a kind of port mapping documentation.

  Examples:

  ```dockerfile
  EXPOSE 6379
  ```

*  `CMD ["executable","param1","param2"]`

  Set a default command, which will be executed when container is run without specifying a command. If container runs with a command, the default command will be ignored.

  Examples:

  ```dockerfile
  CMD ["redis-server"]
  ```

Create a `Dockerfile` file in the application directory with the following content:

```dockerfile
FROM python:alpine3.7
COPY . /app
WORKDIR /app
RUN pip install -r requirements.txt
EXPOSE 5000
CMD python ./index.py
```

### Task 3: Building an image

In order to build an image from a Dockerfile there is `docker build` command. It has the following syntax:

```bash
$ docker build [OPTIONS] PATH
```

`PATH` constitutes a build context - Docker engine will search for a Dockerfile in the path. Besides, there is `--tag` option that enables naming a built image in the `name:tag` format.

Let's build an image from the Dockerfile prepared in the previous task:

```bash
$ docker build --tag sample_app:v1.0 .
Sending build context to Docker daemon  4.096kB
Step 1/6 : FROM python:alpine3.7
 ...
Step 2/6 : COPY . /app
 ...
 ---> b2900adc0cb4
Step 3/6 : WORKDIR /app
 ...
 ---> 26af9cda64ee
Step 4/6 : RUN pip install -r requirements.txt
 ...
 ---> f03bae55dbc0
Step 5/6 : EXPOSE 5000
 ...
 ---> 4ca17265a3da
Step 6/6 : CMD python ./sample_app.py
 ...
 ---> f4922ef0cb8e
Successfully built f4922ef0cb8e
Successfully tagged sample_app:v1.0
```

**Analyze the command output.**

Note the subsequent steps processed by the Docker engine. Each processed directive commits a new layer on top of the base image.

List images to confirm the successful image build:

```bash
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
sample_app          v1.0                f4922ef0cb8e        About an hour ago   91.8MB
```

Note tag `v1.0` for our new image.

### Task 4: Tagging an image

So far, we've built an image for sample application and tagged it with `v1.0` label. We may want to mark `v1.0` image as the `latest` one.

In order to retag an image there is `docker tag` command with the following syntax:

```bash
$ docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]
```

Let's retag our image as `latest`:

```bash
$ docker tag sample_app:v1.0 sample_app:latest
```

List images to check the new label:

```bash
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
sample_app          latest              f4922ef0cb8e        About an hour ago   91.8MB
sample_app          v1.0                f4922ef0cb8e        About an hour ago   91.8MB
```

Now, we can see two instances of `sample_app` image. One is tagged with `v1.0`. Second is tagged with `latest`. Even though the `SIZE` column informs that each instance consumes `91.8MB` of disk space (`183.6MB` in total), in fact, the two images still consume `91.8MB`, because image tag is just a pointer to the source image.

## Exercises

1. Run a container from the built image and publish the application port. Access the application in a browser. The page should display caption `Hello World!`.

2. Modify the sample application to print something different than `Hello World!`.

3. Rebuild the application image and tag it with label `v1.1`.

4. Mark version `v1.1` of the image as `latest`.

5. Run another container from the newer version of the image and access the application in a browser.
