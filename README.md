# Docker Media Stack

A docker-compose setup to easily install everything that's required to start downloading your favourite series and movies.

## Components
Program | Link
--- | ---
Sonarr | https://sonarr.tv/
Radarr | https://radarr.video/
Deluge | https://deluge-torrent.org/
Jackett | https://github.com/Jackett/Jackett
Plex | https://www.plex.tv
Tautulli | http://tautulli.com/

---
## Instructions

Docker has known networking limitations on MacOS and Windows.  
Therefore, it is best to run this on **Linux**.

### Requirements
- docker
- docker-compose

### Setup
1. Copy `.env.example` to `.env` and make the relevant changes
2. Start all containers
```
docker-compose up -d
```

---
## Futher Information

### Tutorial
This is the final stage of a tutorial series.  

If you're interested to learn more about Docker, you can find the tutorial here:  
https://jaketreacher.github.io/docker-media-stack