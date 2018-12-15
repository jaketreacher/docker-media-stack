# Docker Media Stack

A docker-compose setup to easily install everything that's required to start downloading your favourite series and movies.

## Components
Program | Project | Local URL
--- | --- | ---
Sonarr | https://sonarr.tv/ | `http://localhost:8989`
Radarr | https://radarr.video/ | `http://localhost:7878`
Deluge | https://deluge-torrent.org/ | `http://localhost:8112`
Jackett | https://github.com/Jackett/Jackett | `http://localhost:9117`
Plex | https://www.plex.tv | `https://app.plex.tv/desktop`
Tautulli | http://tautulli.com/ | `http://localhost:8181`

---
## Instructions

Docker has known networking limitations on MacOS and Windows.  
Therefore, it is best to run this on **Linux**.

### Requirements
- docker
- docker-compose

### Setup
1. Copy `.env.example` to `.env` and make the relevant changes
2. Start all containers: `docker-compose up -d`

### Configuration
Given each service in running inside a container, the paths are relative to the container and not to the host.  

#### Deluge
- Download to: `/downloads` 
- Enable the `Label` plugin  

#### Sonarr/Radarr
- When adding Jackett as an Indexer, use the url `jackett:9117` rather than `localhost:9117`
- When adding Deluge as a Download Client, use the host `172.17.0.1`  

#### Plex
- Initial setup: Browse to `http://localhost:32400/web`
- Subsequent use: Plex will be acccessible from `https://app.plex.tv/desktop`
- Movies: `/media/movies`
- TV Shows: `/media/series`

---
## Futher Information

### Tutorial
This is the final stage of a tutorial series.  

If you're interested to learn more about Docker, you can find the tutorial here:  
https://jaketreacher.github.io/docker-media-stack