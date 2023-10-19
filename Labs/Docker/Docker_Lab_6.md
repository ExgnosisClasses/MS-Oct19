# Docker Lab Six

## Building Registries

---

### Step 1: Lab objectives

This lab reviews the basics of building Docker registries using the Docker registry utility.

### Step 2: Setup

You should start this lab with no containers to make it easier to see what is happening. If you have any containers running, you should stop them, then run docker container prune to remove all the stopped containers

### Step 3: Run the registry container

Run the registry:2 image as shown. The image will be pulled automatically from docker hub.

Running this command creates a registry called “devteam” that can be accessed on the local host and uses an anonymous volume for storage located in the docker volume directory. Confirm that it is running using the docker ps.

```bash
D:\docker>docker run -d -p 5500:5000 --restart=always --name prod_qa registry:2
930b9a278a52adc677f04ada4f1b7e7f652ea06206bd1cf1a84ffe15dda6a119

D:\docker>docker ps -a
CONTAINER ID   IMAGE        COMMAND                  CREATED         STATUS         PORTS                    NAMES
930b9a278a52   registry:2   "/entrypoint.sh /etc…"   3 minutes ago   Up 3 minutes   0.0.0.0:5500->5000/tcp   devteam
```
P
### Step 4:  Add an image

First, get the latest alpine image from docker hub and tag it with the tag that indicates it is now going to be pushed into the devteam registry. You can use whatever image name and tag you want after the “/” in the tag.

```bash
D:\docker>docker pull alpine
Using default tag: latest
latest: Pulling from library/alpine
Digest: sha256:f271e74b17ced29b915d351685fd4644785c6d1559dd1f2d4189a5e851ef753a
Status: Image is up to date for alpine:latest
docker.io/library/alpine:latest

D:\docker>docker images
REPOSITORY                 TAG       IMAGE ID       CREATED         SIZE
alpine                     latest    042a816809aa   2 weeks ago     7.05MB
registry                   2         81c944c2288b   2 months ago   24.1MB

D:\docker>docker tag 042a816809aa localhost:5500/myalp:test

D:\docker>docker images
REPOSITORY                 TAG       IMAGE ID       CREATED         SIZE
alpine                     latest    042a816809aa   2 weeks ago     7.05MB
localhost:5500/myalp       test      042a816809aa   2 weeks ago     7.05MB
registry                   2         81c944c2288b   2 months ago   24.1MB

```
Note that there are two tags to the image **042a816809aa**.  The tag indicates what registry the image is located in. 

In this case, the `alpine:latest` tag tells us that the image is located in docker hub in the alpine repository. The other tag tells us that the image can also be found in our devteam registry (or at least it will be after we push it there) in the myalp repository.

Now push the image to our devteam registry and remove the alpine image and the image you just pushed to the devteam registry

```bash

D:\docker>docker push localhost:5500/myalp:test
The push refers to repository [localhost:5500/myalp]
8e012198eea1: Pushed
test: digest: sha256:93d5a28f....a0 size: 528     test      042a816809aa   2 weeks ago     7.05MB

D:\docker>docker rmi alpine
Untagged: alpine:latest
Untagged: alpine@sha256:f271e74b17ced29b915d351685fd4644785c6d1559dd1f2d4189a5e851ef753a

D:\docker>docker rmi localhost:5500/myalp:test
Untagged: localhost:5500/myalp:test
Untagged: localhost:5500/myalp@sha256:93d5a2d288d69b5997b8ba47396d2cbb62a72b5d87cd3351094b5d578a0
Deleted: sha256:042a816809aac8d0f7d7cacac7965782ee2ecac3f21bcf9f24b1de1a7387b769
Deleted: sha256:8e012198eea15b2554b07014081c85fec4967a1b9cc4b65bd9a4bce3ae1c0c88
```

### Step 5: Pull the image

Now you can pull the alpine image from the devteam registry by using its fully qualified name. When it says “pulled from myalp” that means that it is pulling the “test” image from the repository “myalp” in the registry “localhost:5500”.

```bash
D:\docker>docker images
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
registry     2         81c944c2288b   2 months ago   24.1MB

D:\docker>docker pull localhost:5500/myalp:test
test: Pulling from myalp
8921db27df28: Pull complete
Digest: sha256:93d5a28ff72d288d69b5997b8ba47396d2cbb62a72b5d87cd3351094b5d578a0
Status: Downloaded newer image for localhost:5500/myalp:test
localhost:5500/myalp:test

D:\docker>docker images
REPOSITORY             TAG       IMAGE ID       CREATED        SIZE
localhost:5500/myalp   test      042a816809aa   2 weeks ago    7.05MB
registry               2         81c944c2288b   2 months ago   24.1MB
```
### Part 6: Remove the Registry

To clean things up, stop the container that gives access to the registry. Then remove the container, but use the “-v” tag so that the associated anonymous volume where the image is actually located is also deleted.

```bash
D:\docker>docker stop devteam
devteam

D:\docker>docker container rm -v devteam
devteam
```
---

## End of Lab

