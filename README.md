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
  ├── tv/
  ├── movies/
  └── downloads/
      ├── complete/
      └── incomplete/
  ```

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

## Running The Stuff

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

## Configuring Stuff

### Sabnzbd

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