---
title: Docker Compose
parent: Tutorial
permalink: /tutorial/docker-compose
nav_order: 5
---
# Docker Compose
{:.no_toc}

---
## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---
## Network

By default, Docker will use the `bridge` network when a container is started.  

The `bridge` network uses the subnet `172.17.0.0/16`, which means you can connect to another container by referencing it's IP address.  
This can be troublesome, as the container won't necessarily have the same IP address each time.  

It is possible to manually setup a virtual private network for the containers, but that it outside the scope of this tutorial. And honestly, I've never had a need to do it as I'm about to show you an easier way.

---
## Automation

If your system has multiple containers it can be troublesome and tedious to have to manually run each container just to get the system working.  

Fortunately, we can automate this with `docker-compose`.  

Let's start with Deluge.  

1. Create a new file, `docker-compose.yml`.  

2. Add a deluge service within the file   
    ```
    version: '3'
    services:
        deluge:
            build:
                context: .
                dockerfile: deluge.dockerfile
            image: tutorial/deluge
            ports:
                - 8112:8112
            volumes:
                - ./downloads:/downloads
                - ./config/deluge:/config
    ```
    
    • `version` will, as you may have guessed, specify the version. If certain features don't work, ensure to check they are compatible with the version being used.  
    • `services` is a dictionary of all the containers you intend to run. The key used can be referenced, so ensure to name it appropriately.  
    • `build` specifies that you intend to build an image for container.  
    • `image` will run that particular image.  If `build` is provided, it will tag the image with this value.  
    • `ports` will map the ports.  
    • `volumes` will map the volumes.  

3. Add a sonarr service  

    ---
    Challenge
    {:.text-delta}
    Add a sonarr service to `docker-compose.yml`.  

    • to add another component to docker compose, you simply list another service under the `services` key  
    • the config for sonarr is: `/root/.config/NzbDrone`

    <details><summary markdown="span">Solution</summary>
    ```
    services:
        ... 
        sonarr:
            build: .
            image: tutorial/sonarr
            ports:
                - 8989:8989
            volumes:
                - ./downloads:/downloads
                - ./config/sonarr:/root/.config/NzbDrone
                - ./media:/media
    ```

    Given the dockerfile for Sonarr uses the default name `Dockerfile`, it is not necessary to supply the `build.context` and `build.dockerfile` keys.  
    </details>
    
    ---

3. Build the image with: `docker-compose build`  
    
    Given our configuration, this is the equivalent of running:  
    ```
    docker build -t tutorial/deluge -f deluge.dockerfile . && \
    docker build -t tutorial/sonarr .
    ```

    If our docker-compose file is not named `docker-compose.yml`, we can use the `-f` flag to specify an alternative filename.  

4. Run with:  `docker-compose up -d`  
    The `-d` argument will detach the containers, as it does when using `run`.  

    Given our configuration, this is the equivalent of running:  
    ```
    docker run --rm -d -p 8112:8112 -v $(pwd)/downloads:/downloads -v $(pwd)/config/deluge:/config tutorial/deluge && \
    docker run --rm -d -p 8989:8989 -v $(pwd)/downloads:/downloads -v $(pwd)/config/sonarr:/root/.config/NzbDrone -v $(pwd)/media:/media tutorial/sonarr
    ```
    The name of the container will be `{directory}_{service_key}`. So if the `docker-compose.yml` was on the desktop, the sonarr container would be called `desktop_sonarr`. You can explicity name the container with the key `container_name`.  

    Take note that by running `docker-compose up`, it will automatically build any images that don't already exist. If you modify the dockerfile, you will have to explicity run either `docker-compose build` or `docker-compose up --build` to force the image(s) to rebuild.  

5. Stop with: `docker-compose down`

---
## Modify docker-compose

By creating an image, it will essentially lock the service to a particular version, as that is what is installed when the image was built.  

We could create a cron job to periodically rebuild our images, as that could potentially install a later version of the service – but that's a bit of a hack. Given build pipelines are outside the scope of this tutorial, we'll use the linuxservice images from now on.  

1. Create a file `.env`  
    ```
    TZ=Australia/Perth
    PUID=1000
    PGID=1000
    CONFIG_DIR=./config
    MEDIA_DIR=./Media
    DOWNLOAD_DIR=./Downloads
    ```
    Ensure to change TZ to be appropriate for your location.  

2. Convert our `docker-compose.yml` file to use linuxserver images
    ```
    version: '3'
    services:
    sonarr:
        image: linuxserver/sonarr
        container_name: sonarr
        ports:
        - 8989:8989
        volumes:
        - ${CONFIG_DIR:?}/sonarr:/config
        - ${MEDIA_DIR:?}/series:/tv
        - ${DOWNLOAD_DIR:?}:/downloads
        env_file: ./.env
        restart: unless-stopped

    deluge:
        image: linuxserver/deluge
        container_name: deluge
        network_mode: host
        volumes:
        - ${CONFIG_DIR:?}/deluge:/config
        - ${DOWNLOAD_DIR:?}:/torrents
        env_file: ./.env
        restart: unless-stopped
    ```

    You may notice there are a few extra fields:  
    • `env_file`: specify an environment file to use.  
    • `${VAR_NAME:?}`: variable substitution from the `.env` file. The `:?` means that if the variable is unset or empty, it will display the error after `?`, which is this case is nothing.  
    • `restart`: specify a restart policy, with the default being `no`. By setting it to `unless-stopped`, it ensures that the container will be brought back up if the services crashes or even if the host machine restarts.  
    • `network_mode: host`: by default, the `network_mode` is `bridge`, of which docker-compose will create a virtual network for all the containers. This mode, though, causes the container to share the network of the host machine. Deluge (and Plex) has autodiscovery features which won't work when in bridge mode, hence they must be set to host mode.  

3. Add remaining services

    ---
    Challenge
    {:.text-delta}
    Add the following services:  
    • linuxserver/radarr  
    • linuxserver/jackett  
    • plexinc/pms-docker  
    • tautulli/tautulli  

    Tips:  
    • `docker pull <image>` to download the image; then  
    • `docker inspect <image>` to get details about ports and mounts.  
    • Read the image documentation on `hub.docker.com` for further instructions on how to use it.  

    <details><summary markdown="span">Solution</summary>
    ```
    radarr:
        image: linuxserver/radarr
        container_name: radarr
        ports:
        - 7878:7878
        volumes:
        - ${CONFIG_DIR:?}/radarr:/config
        - ${MEDIA_DIR:?}/movies:/tv
        - ${DOWNLOAD_DIR:?}:/downloads
        env_file: ./.env
        restart: unless-stopped

    jackett:
        image: linuxserver/jackett
        container_name: jackett
        ports:
        - 9117:9117
        volumes:
        - ${CONFIG_DIR:?}/jackett:/config
        env_file: ./.env
        restart: unless-stopped

    plex:
        image: plexinc/pms-docker
        container_name: plex
        network_mode: host
        volumes:
        - ${CONFIG_DIR:?}/plex:/config
        - ${MEDIA_DIR:?}:/media
        env_file: ./.env
        restart: unless-stopped

    tautulli:
        image: tautulli/tautulli
        container_name: tautulli
        ports:
        - 8181:8181
        volumes:
        - ${CONFIG_DIR:?}/tautulli:/config
        - ${CONFIG_DIR:?}/plex/Library/Application\ Support/Plex\ Media\Server/Logs/:/plex_logs:ro
        env_file: ./.env
        restart: unless-stopped
    ```

    As previously mentioned, Plex has autodiscovery features, which is why we've specified `network_mode: host`.  
    For tautulli, I've added `:ro` to the `/plex_logs`. This means `read-only`.  
    </details>
    
    ---

4. Bring up all services  
    ```
    docker-compose up -d
    ```

5. Configure your services  

    When adding jackett as an Indexer, are able to use the URL `jackett:9117` since it is on the same bridge network. This is preferred as the IP address can change if the containers start in a different order.  

    When adding deluge as a Download Client, this won't work as deluge is using the host network. The solution is to use the host `172.17.0.1`, which is the gateway of the default bridge network. All data sent through here, even if on a different private bridge network, will be forwarded to the host.  

## WatchTower

A useful container to include is `v2tec/watchtower`.  

This will periodically check if new images are available, pull them, and replace your running containers with the latest images.  

To use WatchTower, add the following into `docker-compose.yml`  
```
watchtower:
    image: v2tec/watchtower
    container_name: watchtower_media-stack
    volumes:
        - /var/run/docker.sock:/var/run/docker.sock
    command: sonarr radarr deluge jackett plex tautulli watchtower_media-stack -s "0 0 3 ? * *" --cleanup
    env_file: ./.env
    restart: unless-stopped
```

`command` will override the default command.  

By default, watchtower will watch all containers. By specifying container names explicity, it will limit which ones are updated.  

The other default setting is for watchtower to check every 5 minutes – which is far too frequent for this type of project. Instead, I've provided the `-s` arugment which expects a cron expression. `-s "0 0 3 ? * *"` equates to checking at 3am every day.  

Finally, `--cleanup` will ensure that images are pruned as they are replaced.  
