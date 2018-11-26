---
title: Building Images
parent: Tutorial
permalink: /tutorial/building-images
nav_order: 4
---
# Building Images
{:.no_toc}

---
## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---
## Dockerfile
We previously used an existing image created by linuxserver. Now, we're going to make our own.  

When creating a Docker image, each step in the build process creates a 'layer'.  

This is beneficial as common layers will only be downloaded once. This results in images being downloaded faster and taking up less space.  

1. Create a file called `Dockerfile`
    When building an image, this is the default name that is used.

2. Add the following line:
    ```
    FROM ubuntu
    ```
    Most images have a base image that they will build on top of. In our case, we're building on top of ubuntu.  
    Since we haven't specified a tag, it will default to `ubuntu:latest`.  
    In most production environments, it is best to explicity specify which tag to use as relying on `latest` can result in dependency issues. But for this project, that won't be necessary.  

2. Add the following lines:
    ```
    RUN apt-get update
    RUN apt-get install -y curl
    ```
    The `RUN` instruction will cause the shell to execute the provided commands. This is one way we're able to manipulate the base image.  
    
    There are two forms for the `RUN` command:  
    1) `RUN <command>` - shell form, which will run the commands with `/bin/sh -c`  
    2) `RUN ["executable", "param1", "param2"]` - exec form, in which you can specifiy what the commands will run with

    As an example, `apt-get update` could be typed as follows:    
    ```
    RUN ["/bin/sh", "-c", "apt-get", "update"]
    ```
    However, the first variation is the most commonly used.  

3. Add the following lines:
    ```
    VOLUME [ "/downloads" ]
    ```
    The `VOLUME` instruction will create a mount point. This is the preferred mechanism for persisting data.  
    This will create the directory `/downloads` inside the container. If we do not provide a `-v` arugment when running the container, Docker will automatically mount it for us.  

4. Add the following line:
    ```
    WORKDIR /downloads
    ```
    The `WORKDIR` instruction sets the working directory for the instructions that will follow it.  

5. Add the following line:
    ```
    CMD ["/usr/bin/curl", "-O", "https://raw.githubusercontent.com/jaketreacher/docker-media-stack/master/README.md"]
    ```
    The `CMD` instruction provides the default execution for a running container. There can be only one `CMD` instruction. If more than one is provided, only the last will be used.  

    The command we've provided will download the `README.md` file from the `master` branch of this project. Not exactly the most useful task - but don't worry, we'll be changing it soon!  

    Similarly to `RUN`, `CMD` can be described in both an shell form and an exec form. The exec form that we've used is the most common for `CMD`, whereas shell is most common for `RUN`.  

    Our Dockerfile should look like this:
    ```
    FROM ubuntu
    RUN apt-get update
    RUN apt-get install -y curl
    VOLUME [ "/downloads" ]
    WORKDIR /downloads
    CMD [ "/usr/bin/curl", "-O", "https://raw.githubusercontent.com/jaketreacher/docker-media-stack/master/README.md" ]
    ```

6. Build the image by running the following command:  
    ```
    docker build -t tutorial/curl .
    ```
    The `-t` argument will set the tag in the format `name:tag`. Given we haven't explicity specified a `:tag` it will default to `:latest`.  
    The `.` is the build context - this is the directory that will be sent into the docker daemon.  
    The `-f` argument can be used to specific a Dockerfile that isn't named `Dockerfile`.  

    You will see the output of each step during the build process.  

7. List your images:
    ```
    docker image ls -a
    ```

    ![]({{site.baseurl}}/assets/building-images/build-images.png)
    We should see our image `tutorial/curl` in addition to four blank images. This is because each instruction in the Dockerfile will create a new image. If we were to build again, it will use these cached images. Modifications in the Dockerfile will result in only the subsequent steps building a new image.  

    Something to keep in mind is that if we were to build this image again, the apt cache would not be up-to-date as Docker will use the previously cached image. If you want to force Docker to rebuild everything, simply add the `--no-cache` argument.  

8. Run our image:
    ```
    docker run -it --rm -v $(pwd)/downloads:/downloads tutorial/curl
    ```

    Running this image will:  
    1) create a new `downloads` directory in our current working directory;  
    2) run the `curl` command inside the container; and  
    3) download `README.md`.  
    
    _Take note that any directories that Docker creates will be owned by the root user and may result in permission conflicts._

    ---
    Challenge
    {:.text-delta}
    Why does the container stop after `curl` finishes the download?

    <details><summary markdown="span">Solution</summary>
    `CMD` will assign a process to PID 1. Once PID 1 ends, the container will exit.
    </details>

    ---

---
## Upgrade to Sonarr

1. Create a Dockerfile to build a Sonarr image  
    You're on your own! Attempt to use what we've discussed to create an image that will run Sonarr.  

    Here are some useful tips:  
    • Sonarr requires Mono to run (check for a Mono base image)  
    • Installation instructions [here](https://github.com/Sonarr/Sonarr/wiki/Installation#debianubuntu)  
    • Mono: `/usr/bin/mono`  
    • Sonarr: `/opt/NzbDrone/NzbDrone.exe`  
    • You should have two volumes, `/media` and `/downloads`  
    • Run `NzbDrone` with `--nobrowser`  

    ---
    <details><summary markdown="span">Solution</summary>
    ```
    FROM mono
    RUN apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 0xFDA5DFFC && \
        echo "deb http://apt.sonarr.tv/ master main" | tee /etc/apt/sources.list.d/sonarr.list && \
        apt-get update && \
        apt-get install nzbdrone -y
    VOLUME [ "/media", "/downloads" ]
    CMD ["/usr/bin/mono", "/opt/NzbDrone/NzbDrone.exe", "--nobrowser"]
    ```

    Since each instruction will create a new layer, I've opted to combine `RUN` into a single step.  
    For a production image, this is the preferred method as it will make the image smaller.  
    </details>

    ---

2. Build our image:
    ```
    docker build -t tutorial/sonarr .
    ```

3. Run our image:
    ```
    docker run --rm -d -p 8989:8989 \
        -v $(pwd)/media:/media \
        -v $(pwd)/downloads:/downloads \
        -v $(pwd)/config:/root/.config/NzbDrone \
        tutorial/sonarr
    ```

---
## Multiple Processes
The Docker developers advocate the philosophy of running a single logical service per container.  

A logical service can consist of multiple OS processes. For example, if you had a webapp and a database, they are two different services so you wouldn't run them in the same container.  

Deluge is a single service that requries two components: the daemon `deluged`, and the web-ui `deluge-web`.  

Given the Dockerfile will only accept a single `CMD` instruction, the way to run multiple processes is to create a bash script to act as a supervisor. However, this is outside of the scope of this tutorial so instead we will use a base image that already has this configured.  

The two popular images for this are:  
1) [phusion/baseimage](https://github.com/phusion/baseimage-docker); and  
2) [just-containers/s6-overlay](https://github.com/just-containers/s6-overlay).  

Linuxserver uses the latter for their images, but I found the former to be easier to configure. So we'll continue with that.  

---
## Deluge Dockerfile

1. Create a new directory called `runit`  
    The name is not important but given `phusion/baseimage` utilises [runit](http://smarden.org/runit/) for service supervision, it seems fitting.  

2. Create `runit/deluged.sh` with the following content
    ```
    #!/bin/sh
    exec /usr/bin/deluged -c /config -d --loglevel=info -l /config/deluged.log
    ```

3. And another `runit/deluge-web.sh`  
    ```
    #!/bin/sh
    exec /usr/bin/deluge-web -c /config --loglevel=info
    ```

4. Ensure our scripts are executable  
    ```
    sudo chmod +x runit/*
    ```

5. And finally `deluge.dockerfile`  
    ```
    FROM phusion/baseimage
    COPY runit /etc/sv/
    RUN apt-get update && \ 
        apt-get install -y deluged deluge-web && \
        mkdir -p /etc/service/deluged && \ 
        ln -s /etc/sv/deluged.sh /etc/service/deluged/run && \
        mkdir -p /etc/service/deluge-web && \ 
        ln -s /etc/sv/deluge-web.sh /etc/service/deluge-web/run
    VOLUME [ "/config", "/downloads" ]
    ```

    You may have noticed a new instruction, `COPY`. This will copy the _contents_ of our `runit` directory into the `/etc/sv/` directory of the container.  

    The default scripts provided by `phusion/baseimage` will scan all directories in `/etc/service` and execute the containing `run` files. Each directory should be named after it's corresponding process.  

    Looking at our above dockerfile, you can see that we've created the following soft links:
    ```
    /etc/service/deluged/run --> /etc/sv/deluged.sh
    /etc/service/deluge-web/run --> /etc/sv/deluge-web.sh
    ```

6. Build the image
    ```
    docker build -f deluge.dockerfile -t tutorial/deluge .
    ```
    Given our dockerfile isn't named `Dockerfile`, we specific the name with the `-f` argument.  

7. Run
    ```
    docker run --rm -d -p 8112:8112 tutorial/deluge
    ```
    The default port for deluge-web is 8112.  

    Open your browser to `http://localhost:8112` to confirm it is working.  
