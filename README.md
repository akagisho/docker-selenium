# THIS REPO HAS MOVED

See: https://github.com/SeleniumHQ/docker-selenium

Commits in this repo are only meant for people who are not yet aware that the repo moved to [SeleniumHQ/docker-selenium](https://github.com/SeleniumHQ/docker-selenium) and in order to avoid CI failures trusting this docker image.

## Docker build to spawn selenium standalone servers with Chrome and Firefox
[![Gitter](https://badges.gitter.im/Join Chat.svg)](https://gitter.im/elgalu/docker-selenium?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

* selenium-server-standalone
* google-chrome-stable
* firefox (stable)
* VNC access (useful for debugging the container)
* fluxbox (lightweight window manager for X)

### One-liner Install & Usage

    sudo docker run --privileged -p 4444:4444 -p 5900:5900 elgalu/docker-selenium

### Step by step non-privileged do it yourself

General note: add `sudo` only if needed in your environment

#### 1. Build this image

Ensure you have the Ubuntu base image downloaded, this step is optional since docker takes care of downloading the parent base image automatically, but for the sake of curiosity:

    docker run -i -t ubuntu:14.04.1 /bin/bash

If you don't git clone this repo, you can simply build from github:

    docker build github.com/elgalu/docker-selenium
    ID=$(docker images -q | sed -n 1p)
    docker tag $ID elgalu/docker-selenium:latest

If you git clone this repo locally, i.e. cd into where the Dockerfile is, you can:

    docker build -t="elgalu/docker-selenium:local" .

If you prefer to download the final built image from docker you can pull it, personally I always prefer to build them manually except for the base images like Ubuntu 14.04.1:

    docker pull -t="elgalu/docker-selenium:latest" elgalu/docker-selenium

#### 2. Use this image

##### e.g. Spawn a container for Chrome testing:

    CH=$(docker run --rm --name=ch -p=127.0.0.1::4444 -p=127.0.0.1::5900 \
        -v /e2e/uploads:/e2e/uploads elgalu/docker-selenium:local)

Note `-v /e2e/uploads:/e2e/uploads` is optional in case you are testing browser uploads on your webapp you'll probably need to share a directory for this.

The `127.0.0.1::` part is to avoid binding to all network interfaces, most of the time you don't need to expose the docker container like that so just *localhost* for now.

I like to remove the containers after each e2e test with `--rm` since this docker container is not meant to preserve state, spawning a new one is less than 3 seconds. You need to think of your docker container as processes, not as running virtual machines if case you are familiar with vagrant.

A dynamic port will be binded to the container ones, i.e.

    # Obtain the selenium port you'll connect to:
    docker port $CH 4444
    #=> 127.0.0.1:49155

    # Obtain the VNC server port in case you want to look around
    docker port $CH 5900
    #=> 127.0.0.1:49160

In case you have RealVNC binary `vnc` in your path, you can always take a look, view only to avoid messing around your tests with an unintended mouse click or keyboard.

    ./bin/vncview 127.0.0.1:49160

##### e.g. Spawn a container for Firefox testing:

This command line is the same as for Chrome, remember that the selenium running container is able to launch either Chrome or Firefox, the idea around having 2 separate containers, one for each browser is for convenience plus avoid certain `:focus` issues you web app may encounter during e2e automation.

    FF=$(docker run --rm --name=ff -p=127.0.0.1::4444 -p=127.0.0.1::5900 \
        -v /e2e/uploads:/e2e/uploads elgalu/docker-selenium:local)

##### Look around

    docker images
    #=>

    REPOSITORY               TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
    elgalu/docker-selenium   local               1c68c8823418        2 minutes ago       813.2 MB
    ubuntu                   14.04.1             e54ca5efa2e9        2 weeks ago         199.8 MB

### Troubleshooting

All output is sent to stdout so it can be inspected by running:

``` bash
$ docker logs -f <container-id|container-name>
```

Container leaves a few logs files to see what happened:

    /tmp/Xvfb_headless.log
    /tmp/fluxbox_manager.log
    /tmp/x11vnc_forever.log
    /tmp/local-sel-headless.log
    /tmp/selenium-server-standalone.log
