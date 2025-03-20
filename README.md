# Media Server Stack

Docker Compose setup for a home media server with Jellyfin, Sonarr, Radarr, Lidarr, Ombi, and SABnzbd.

## Services

- **Jellyfin** (8096): Media server
- **Sonarr** (8989): TV show management
- **Radarr** (7878): Movie management
- **Lidarr** (8686): Music management
- **Ombi** (3579): Media request management
- **SABnzbd** (8080): Usenet downloader

## Prerequisites

- Docker and Docker Compose
- A media drive mounted at `/media/colemine` with the following structure:
  ```
  /media/colemine/
  ├── tv/          # TV shows library (managed by Sonarr)
  ├── movies/      # Movies library (managed by Radarr)
  ├── music/       # Music library (managed by Lidarr)
  └── downloads/   # Downloads directory (managed by SABnzbd)
      ├── complete/    # Completed downloads
      └── incomplete/  # Downloads in progress
  ```

## File Organization

The stack uses a specific file organization pattern:

1. SABnzbd downloads files to `/media/colemine/downloads/complete`
2. The `*arr` services monitor this directory and handle file organization:
   - Sonarr moves TV shows to `/media/colemine/tv`
   - Radarr moves movies to `/media/colemine/movies`
   - Lidarr moves music to `/media/colemine/music`
3. Jellyfin serves the organized content from these library directories

This is why each `*arr` service needs:
- Access to the downloads directory
- Access to its respective library directory
- Proper configuration of root folders in its settings

## Host Setup

1. Create required system user and group:
   ```bash
   sudo useradd -r -s /bin/false media
   sudo groupadd media-access
   sudo usermod -a -G media-access media
   ```

2. Create config directories:
   ```bash
   sudo mkdir -p /etc/jellyfin/config
   sudo mkdir -p /etc/sonarr
   sudo mkdir -p /etc/radarr
   sudo mkdir -p /etc/lidarr
   sudo mkdir -p /etc/ombi
   sudo mkdir -p /var/sabnzbd
   ```

3a. Ensure media drive is mounted, fstab entry set up, funny name, all that jazz.

3b. Ensure media drive is owned by media-access group:
   ```bash
   sudo chown -R :media-access /media/colemine
   ```

## Network

Services run on a dedicated Docker network (`medianet`) for isolation. All services can communicate with each other using their container names as hostnames.

## Web and Reverse Proxy

A separate HAProxy setup in the `haproxy/` directory provides:
- Reverse proxy for all media services
- Clean URLs using subdomains (e.g., `jellyfin.tharmas.local`)
- A landing page at `tharmas.local` with links to all services

To set up:
1. Ensure media services are running first
2. `cd haproxy/`
3. `docker compose up -d`

Services will be available at:
- http://tharmas.local (landing page)
- http://jellyfin.tharmas.local
- http://sonarr.tharmas.local
- http://radarr.tharmas.local
- http://lidarr.tharmas.local
- http://ombi.tharmas.local
- http://sabnzbd.tharmas.local

## Running The Media Stuff

```bash
docker compose up -d
```

Access services at:
- Jellyfin: http://host:8096
- Sonarr: http://host:8989
- Radarr: http://host:7878
- Lidarr: http://host:8686
- Ombi: http://host:3579
- SABnzbd: http://host:8080

## Configuring Media Stuff

### Sabnzbd
Go through the little wizard, set up access creds, etc.

Open settings, do a coupla three things:
- Servers tab: Add your Usenet Server Creds
- Special tab: Add `sabnzbd` to the hostname whitelist at the bottom (this lets the other containers talk to sabnzbd over the Docker `medianet` network)
- General tab: Turn off "Launch browser at startup" (because you're running this headless, like a l33t d00d.)

Also, grab the API Key out of the General -> Security settings, because your various `.*arr`s need it.

### Sonarr / Radarr / Lidarr

##### Settings -> Media Management
Add the appropriate root folder for each. If you're using the docker compose unmodified, that would be `/tv`, `/movies`, and `/music`, respectively.

##### Settings -> Indexers
Add your creds from your indexer(s) (I used NZBGeek) in the Indexers settings.

##### Settings -> Download Clients
Setup sabnzbd in here. Use `sabnzbd` for the host and the API key you got from your sabnbzd instance. Hit the little test button. If it doesn't work, you probably forgot to whitelist the `sabnzbd` host in the whitelist (see above).

Note: Lidarr wants to use a "music" category by default. sabnzbd doesn't have one by default - instead it has "audio". You can either create a "music" category in sabnzbd, or change the category from lidarr to "audio", or just not use one. Not sure if this matters much.

### Jellyfin

You should just need to add libraries, pointed at `/data/movies`, `/data/music`, and `/data/tvshows`.

However, it's been my experience that Jellyfin is a little fussy about libraries if the directory starts off empty. The only thing that I could figure to make it ... not fussy ... was to download something with each arr, restart Jellyfin, and re-add the library. Dumb, but whatever.