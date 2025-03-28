# Media Server Stack

Docker Compose setup for a home media server with Jellyfin, Sonarr, Radarr, Lidarr, and SABnzbd.

Use the .env file to set up the various variables in the docker-compose.yml. Mine's committed if you need an example.

This is a README, not a _GUIDE_, and is not written to be totally noob-proof. It's mostly just here for me if I need to jog my memory. If you want something a little more hand-holdy, go check out [trash-guides.info](https://trash-guides.info/), or dig around on reddit. But if you're halfway savvy, this might save you some time getting your own stuff set up.

## Services

- **Jellyfin** (8096): Media server
- **Sonarr** (8989): TV show management
- **Radarr** (7878): Movie management
- **Lidarr** (8686): Music management
- **SABnzbd** (8080): Usenet downloader

## Prerequisites

- Docker and Docker Compose
- The ability to read
- Basic knowledge of Docker and Linux
- Reading and doing the following sections on Host Setup and File Organization

### Host Setup

1. Create required system user and group (you don't _necessarily_ have to do this, and it's probably more trouble than it's worth, but it's what I did)
   ```bash
   sudo useradd -r -s /bin/false media
   sudo groupadd media
   sudo usermod -a -G media media

   # you'll probably also want to add your regular user to this group.
   ```

2. Create config directories:
   ```bash
   # configs, swap in the right CONFIG_ROOT for wherever you're putting container config volumes:
   (CONFIG_ROOT=foo; mkdir -p ${CONFIG_ROOT}/{sonarr,radarr,lidarr,sabnzbd,jellyfin})
   sudo chown -r media:media ${MEDIA_}
   ```

3a. Ensure media drive is mounted, fstab entry set up, funny name, all that jazz. Then, see below for info on setting up dir structure.

3b. If you like, you can run the following to make all your directories the way I did:
```

# media stuff, swap in the right MEDIA_ROOT for wherever you're putting media:
(MEDIA_ROOT=foo; mkdir -p $MEDIA_ROOT/{torrents,usenet,media} $MEDIA_ROOT/torrents/{books,movies,music,tv} $MEDIA_ROOT/usenet/{incomplete,complete} $MEDIA_ROOT/usenet/complete/{books,movies,music,tv} $MEDIA_ROOT/media/{books,movies,music,tv})
```

Again, make sure the ownership on these is set appropriately. For me this meant chown'ing the directories with media:media (my media user).

3c. Ensure media drive is owned by media group:
   ```bash
   sudo chown -R media:media /media/colemine
   ```

### Media and Container File Organization
- A media drive mounted at `$MEDIA_ROOT` (set this correctly in the .env file)
   - Note: You want to structure your media drive in a sensible way that allows for fast file moves via hardlinks (when using torrents) or atomic file moves (when using Usenet). (see [Trash Guides](https://trash-guides.info/File-and-Folder-Structure/How-to-set-up/Docker/) for rationale / more info).  alternatively, do it however you want, it's your shit.
```
  /media/colemine/
   ├── torrents
   │   ├── books
   │   ├── movies
   │   ├── music
   │   └── tv
   ├── usenet
   │   ├── incomplete
   │   └── complete
   │       ├── books
   │       ├── movies
   │       ├── music
   │       └── tv
   └── media
      ├── books
      ├── movies
      ├── music
      └── tv
```


## Network

Services run on a dedicated Docker network (`medianet`) for isolation. All services can communicate with each other using their container names as hostnames.

## Running The Media Stuff

```bash
docker compose up -d
```

Access services at:
- Jellyfin: http://host:8096
- Sonarr: http://host:8989
- Radarr: http://host:7878
- Lidarr: http://host:8686
- SABnzbd: http://host:8080

## Configuring Media Stuff

### Sabnzbd
Go through the little wizard, set up access creds, etc.

Open settings, do a coupla three things:
- Servers tab: Add your Usenet Server Creds (or multiples)
- Special tab: Add `sabnzbd` to the hostname whitelist at the bottom (this lets the other containers talk to sabnzbd over the Docker `medianet` network)
- General tab: Turn off "Launch browser at startup" (because you're running this headless, like a l33t d00d.)

Also, grab the API Key out of the General -> Security settings, because your various `.*arr`s need it.

### Sonarr / Radarr / Lidarr

##### Settings -> Media Management
Add the appropriate root folder for each. If you're using the docker compose file unmodified, that would be `/data/media/tv`, `/data/media/movies`, and `/data/media/music`, respectively.

##### Settings -> Indexers
Add your creds from your indexer(s) (I used NZBGeek + nzb.su) in the Indexers settings.

##### Settings -> Download Clients
Setup sabnzbd in here. Use `sabnzbd` for the host and the API key you got from your sabnbzd instance. Hit the little test button. If it doesn't work, you probably forgot to whitelist the `sabnzbd` host in the whitelist (see above).

Note: Lidarr wants to use a "music" category by default. sabnzbd doesn't have one by default - instead it has "audio". You can either create a "music" category in sabnzbd, or change the category from lidarr to "audio", or just not use one. Not sure if this matters much.

### Jellyfin

You should just need to add libraries, pointed at `/data/media/movies`, `/data/media/music`, and `/data/media/tvshows`.

However, it's been my experience that Jellyfin is a little fussy about libraries if the directory starts off empty. The only thing that I could figure to make it ... not fussy ... was to download something with each arr, restart Jellyfin, and re-add the library. Dumb, but whatever.