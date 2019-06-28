---
published: true
---

As you might already know it’s healthy to produce small docker images with only the needed dependences for your software to work properly.

Being small in size, means more efficency in your pipeline and deployments, It’s small to download, small to push. So keep that in mind when you are building a docker image for anything.

## Generic Guidelines:

There is some generic guidelines that you can keep in mind while building your own docker images, at the top of my mind:

    Make sure you don’t genereate caching files, or delete them after installations, example of tools that uses cache files: pip, apt, apk
    Some times there is some dependency packages that you need for build step only for your project, and then they are not needed anymore, example on that is gcc. Make sure to remove these dependencies from the image.
    Docker images consists of layers, each layer represents a command in your Dockerfile, we need to keep those layers small in size and only contain the files they need, so for example if your installing python requirements using pip, you only need the requirements file and not the whole source code, so we will make sure to only copy the requirements.txt file to Dockerimage before installing those requirements using pip, as we will see later in this article.
    Chain your commands in one layer, meaning: one RUN command.
    I always try to use Alpine as base images first because they are small in size, they work for most of the usecases

## Our Dockerfile for Python

We will write a simple Dockerfile for python and install some dependencies just for the fun of it. And here is the main points the we need to focus one:

    Our base image will be alpine
    Its always good to specifiy which python version we will work on
    Copy the requirements.txt file only, before executing pip install
    Installing system dependencies and pip dependencipes in the same image layer (same RUN command)
    Make sure system dependencies that are used for building or (pip install) step only shall be removed immediately after executing pip install, It’s crucial to remove them in the same layer as the installation, otherwise the trick won’t work and those dependencies will just be copied to the next layer.
    We will use apk to install those dependencies in a virtual namespace so that we can remove them with that name
    There is no need to have a virtual environment since docker container itself is an isolated environment. using virtualenv defies the purpose of docker containers.

**So, let’s dive in:**

```Dockerfile
# Alpine base image that contains python 3.7
FROM python:3.7-alpine

# define the directory to work in
WORKDIR /code

# copy the requirements.txt file to the work directory
COPY requirements.txt .

# Install some system deps in a virtual environment named .build-deps, you can name it what ever you want

# install pip dependencies in the same layer

RUN apk add --no-cache --virtual .build-deps \
    build-base openssl-dev pkgconfig libffi-dev \
    cups-dev jpeg-dev && \
    pip install --no-cache-dir -r requirements.txt && \
    apk del .build-deps # delete the .build-deps in the same layer

# Copy rest of the source code
COPY src/ src/

# EXPOSE the needed ports, for example 8080
EXPOSE 8080

# Running Command or Entry Point
CMD python src/app.py
```

As you can see above a simple Dockerfile that uses Python 3.7 alpine image, and applies the rules we defined above to make our image small.

**Also notice the syntax of apk, let’s break it down:**

 - `--no-cache` tells apk not to store cache files
 - `--virtual .build-deps` tells apk to create a virtual namespace called “.build-deps” and to install the next packages into that namespace.
 - after that the list of packages that you want to install

**What about the pip command, something special ?**

Yes, have you noticed that `--no-cache-dir` it asks pip not to store cache files at all.

**Finally, Removing the building dependencies**

In the same layer (same RUN command) we are also deleting those deps that we don’t need anymore, and they did their job building the packages during `pip install ...` process. The syntax is easy, we ask apk to delete the virtual namespace with all the packages that were installed to it.

That’s it your done ! maybe let’s check how much space have you saved after modifying your Dockerfile.

## Checking size of the images

Now it’s time to check the size of the new image, probably check first the old image before modifying Dockerfile, and after, Let us now much did you reduce !

Using docker command in the command line, get list of images:

```
$ docker images
```

more specifically you can grep for your image name, i will just grep python

```
$ docker images | grep python

  python  3.5-alpine  be8ce886a36a  12 days ago     74MB
  python  3.6-alpine  1837080c5e87  2 months ago    74.4MB
  python  3.7-alpine  aadc3feb2b19  3 months ago    78.2MB
```

in the last column, you can see the final size of the image, above there are 3 python alpine images, check the size of your old image, and then build the new image and check it’s size !

**That’s all folks.**