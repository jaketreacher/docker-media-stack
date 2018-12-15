---
title: Using Images
parent: Tutorial
permalink: /tutorial/using-images
nav_order: 3
---
# Using Images
{:.no_toc}

---
## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---
## Finding Image
We're going to try and get Sonarr running inside of a container.

The easy way would be to use an existing image. We can search for images from the command line:
```
docker search sonarr
```

![]({{site.baseurl}}/assets/using-images/sonarr-search.png)

Official repositories are those which have been curated by Docker. They have clear documentation, and timely security patches.  

Automated means that that particular image has been configured to be built automatically whenever the corresponding repo is updated.  

---
## Run Sonarr
Let's use the image from linuxserver. Since this isn't an official image, it has the developer name appended `linuxserver/sonarr` as opposed to just `sonarr`. Note there is no official image for sonarr.

1. Run the image:
    ```
    docker run linuxserver/sonarr
    ```

    Eventually it will display some log info, and it'll give the following lines:
    ```
    [Info] OwinHostController: Listening on the following URLs: 
    [Info] OwinHostController:   http://*:8989/ 
    [Info] NancyBootstrapper: Starting Web Server 
    [Info] ProfileService: Setting up default quality profiles 
    ```
    Looks like sonarr is running and listening on port 8989.

2. Open your browser to `http://localhost:8989`.
    
    You'll notice that we're unable to connect.

    Athough the container exposes the port 8989, but we haven't published this to our host machine.  

3. Exit the container (ctrl+c) and let's run try again:
    ```
    docker run -p 8989:8989 linuxserver/sonarr
    ```
    <small>Syntax: `-p host:container`</small>
    `-p 8989:8989` means that 8989 on the host will map to 8989 in the container. If we leave the host blank, Docker will automatically assign an available port for us.

4. Open your browser to `http://localhost:8989`.
    And that's it. Sonarr is running. Without having to install any Mono dependencies. Everything has been prepackaged inside of the container.

    The current issue we face is that our terminal is blocked by the log. Since we didn't run the container with `-it` we can't use `Ctr+P,Ctrl+Q` shortcut to leave.
    
    Let's exit (Ctrl+C) and try something else.

5. Run an container in detached mode:
    ```
    docker run -d -p 8989 linuxserver/sonarr
    ```
    We've done two things:  
    a) Add `-d` to make it detached  
    b) Leave off the host when publishing the port  

6. Let's check what's running with `docker ps`
    ![]({{site.baseurl}}/assets/using-images/sonarr-port.png)

    We can see our container is running and a port has been assigned. For me it's 32769.

    Browse to the appropriate port on localhost to confirm it's working.

---
## Clean
Keep note, though, each time we use `run`, it is creating a new container from the image we specify.

1. Get a list of all containers with `docker ps -a`
    ![]({{site.baseurl}}/assets/using-images/docker-ps-messy.png)


2. Let's make this cleaner.

    Remove a single container: `docker container rm <ID|name>`

    Remove all stopped containers: `docker container prune`

    And if you list all containers again, you should see a single sonarr container running.

    ![]({{site.baseurl}}/assets/using-images/docker-ps-clean.png)

---
## Inspecting and Volumes

When linuxserver created this image, they specified the following volumes to be created: `/tv`, `/config`, `/downloads`. Given we didn't specify where to mount these volumes, it has been done for us automatically... albeit in an inconvinient location.  

The host volume will always be mounted on top of the container volume.  

1. List all volumes with `docker volume ls`.
    ![]({{site.baseurl}}/assets/using-images/docker-volume-messy.png)
    
    Whoa, that's a lot! Let's clean this up.

    ---
    Challenge
    {:.text-delta}
    Guess the command used to remove volumes.  
    <small>_Hint: It's similiar to removing containers._</small>

    <details><summary markdown="span">Solution</summary>
    `docker volume rm <ID>`  
    `docker volume prune`

    ![]({{site.baseurl}}/assets/using-images/docker-volume-clean.png)
    </details>

    ---
2. Inspect our running container to get more details:
    ```
    docker container inspect zen_napier
    ```
    <small>_Ensure to replace the name with that of your container_</small>

    If we scroll up, we'll find a `Config` section. Note the following: `ExposedPort`, `Volumes`.
    ![]({{site.baseurl}}/assets/using-images/inspect-config.png)
    This is an easy way to determine which ports/volumes the image is expected to use.

    If we scroll up further, we will see where there volumes have been mounted.

    Using `/config` as an example we can see the container is mounting the host volume `/var/lib/docker/volumes/c514beb58fe6678a416b7140c702ce8cb2597b93cc64e5aaa7abfd37be96d939/_data` as `/config` inside the container.
    ![]({{site.baseurl}}/assets/using-images/inspect-mount.png)

3. Open a new terminal and watch the host directory:
    ```
    watch sudo ls <your source path>
    ```
    <small>_By default, this directory is is owned by root:root. Hence, we use sudo._</small>

4. Start a new shell process inside our container and inspect the `/config` directory:
    ```
    docker exec -it <ID|name> sh
    cd /config
    ls
    ```

5. Create a new file with `touch newfile.txt`. We should see it appear in the other terminal window.
    ![]({{site.baseurl}}/assets/using-images/newfile-1.png)
    ![]({{site.baseurl}}/assets/using-images/newfile-2.png)

---
## Mounting Volumes

In order to mount a volume, we use the `-v` argument.  
Syntax: `-v host:container`  

This works the same as publishing a port – if the host isn't specified, Docker will automatically assign one for you, as we just explored.  
When mounting volumes, the host path must be absolute.  

---
Challenge
{:.text-delta}
- Stop the current Docker container.  
- Remove the container.  
- Remove all dangling volumes.  

<details><summary markdown="span">Solution</summary>
```
docker stop <NAME>
docker container prune
docker volume prune
```
![]()
</details>

---
Finally, let's restart our sonarr container with the volumes mounted correctly:
```
docker run --rm -d -p 8989:8989 \
    -v $(pwd)/config:/config \
    -v $(pwd)/downloads:/downloads \
    -v $(pwd)/tv:/tv \
    -v /etc/localtime:/etc/localtime:ro \
    -e PUID=1000 \
    -e PGID=1000 \
    linuxserver/sonarr
```

As the host path must be absolute, we've provided `$(pwd)` to get the current directory.  
You will notice multiple directories have been created in your current working directory - this is where Docker is mounting the associated volumes.

We have also mounted `/etc/localtime` with `:ro` - read-only. This is because we want the container to use the timezone of the host, but without the ability to make modifications.  

Some environment arguments, `-e`, have been provided. These ensure that the user ID and group ID between the host and container will be the same (assuming your uid/gid is 1000). Take note, though, this only works beacuse the image created by linuxserver has been built to support it - this is not a standard Docker feature.  

If we stop/remove the container and start it again with the same arguments, it will retain all Sonarr configuration.  