# Media Server Stack

Docker Compose setup for a home media server with Jellyfin, Sonarr, Radarr, Lidarr, SABnzbd, Calibre-Web, and Calibre-Web-Automated-Downloader.

Use the .env file to set up the various variables in the docker-compose.yml. Mine's committed if you need an example. (there's nothing secret in there, so it's fine - but you'll almost certainly need to change the values unless you're _really_ committed to aping my stylo).

This is a README, not a _GUIDE_, and is not written to be totally noob-proof. It's mostly just here for me if I need to jog my memory. If you want something a little more hand-holdy, go check out [trash-guides.info](https://trash-guides.info/), or dig around on reddit. But if you're halfway [savvy](https://www.getyarn.io/yarn-clip/6b624dee-11cb-4791-8842-11ef3da01d19), this might save you some time getting your own stuff set up.

## Services

- **[Jellyfin](https://github.com/jellyfin/jellyfin)** (8096): Media server
- **[Sonarr](https://github.com/Sonarr/Sonarr)** (8989): TV show management
- **[Radarr](https://github.com/Radarr/Radarr)** (7878): Movie management
- **[Lidarr](https://github.com/lidarr/lidarr)** (8686): Music management
- **[SABnzbd](https://github.com/sabnzbd/sabnzbd)** (8080): Usenet downloader
- **[Calibre-Web](https://github.com/crocodilestick/Calibre-Web-Automated)** (8083): Book management
- **[Calibre-Web-Automated-Downloader](https://github.com/calibrain/calibre-web-automated-book-downloader)** (8084): Book downloader

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
   # configs, swap in the right CONFIG_ROOT for wherever you're putting container config volumes
   # i did /docker/appdata because these don't really comply with the standard linux intentions behind /etc and /var, so
   (CONFIG_ROOT=foo; mkdir -p ${CONFIG_ROOT}/{sonarr,radarr,lidarr,sabnzbd,jellyfin,calibre-web})
   sudo chown -r media:media ${MEDIA_ROOT}
   ```

3a. Ensure media drive is mounted, fstab entry set up, punny name, all that jazz. Then, see below for info on setting up dir structure.

3b. If you like, you can run the following to make all your directories the way I did:
```

# media stuff, swap in the right MEDIA_ROOT for wherever you're putting media:
(MEDIA_ROOT=foo; mkdir -p $MEDIA_ROOT/{torrents,usenet,media} $MEDIA_ROOT/torrents/{books,movies,music,tv} $MEDIA_ROOT/usenet/{incomplete,complete} $MEDIA_ROOT/usenet/complete/{books,movies,music,tv} $MEDIA_ROOT/media/{books,movies,music,tv}, mkdir -p $MEDIA_ROOT/calibre-ingest)
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
   ├── media
   │   ├── books
   │   ├── movies
   │   ├── music
   │   └── tv
   └── calibre-ingest
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
- Calibre-Web: http://host:8083
- Calibre-Web-Automated-Downloader: http://host:8084

## Configuring Media Stuff

### Sabnzbd
Go through the little wizard, set up access creds, etc.

Open settings, do a [coupla three things](https://getyarn.io/yarn-clip/d07f8355-500d-4c0e-9f94-c128768ff272):
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

Note: Lidarr wants to use a "music" category by default. sabnzbd doesn't have one by default - instead it has "audio". You can either create a "music" category in sabnzbd, or change the category from lidarr to "audio", or just not use one. Not sure if this matters much - I haven't used lidarr all that much.

### Jellyfin

You should just need to add libraries, pointed at `/data/media/movies`, `/data/media/music`, and `/data/media/tvshows`.

However, it's been my experience that Jellyfin is a little fussy about libraries if the directory starts off empty. The only thing that I could figure to make it ... not fussy ... was to download something with each arr, restart Jellyfin, and re-add the library. So if you're starting fresh and having issues, you might try that.

### Calibre-Web

This will all just basically work according to the directions at the [repo](https://github.com/crocodilestick/Calibre-Web-Automated).

_Except_ if you want to use the "Send to Kindle" feature, in which case, there's a bit of pain involved with setting up the email.

You _can_ do it with Gmail but that looked like a [whole pain in the ass](https://github.com/janeczku/calibre-web/wiki/Setup-Mailserver#Gmail), so I opted to set up a free account with [GMX](https://www.gmx.com/).

You'll have to go into the email settings on GMX to [Enable POP3/IMAP](https://support.gmx.com/pop-imap/toggle.html), because it's off by default.

Then you should be able to use the server settings [here](https://support.gmx.com/pop-imap/pop3/serverdata.html) to configure the email settings on Calibre-Web.

For posterity, I ended up using: mail.gmx.com, port 587, STARTTLS encryption. The rest of the fields are self-explanatory _except_ for "from email" - it was not clear what I was supposed to put in here. Putting the email I registered seemed to work.

To be able to use the little "send test email" button, you need to go into your user settings on Calibre Web (likely your admin account, unless you're getting fancier than me) and set the email.

Lastly, you _must_ go add your new GMX email to the list of authorized senders in your Amazon account. At the time of writing you can get to this by going "Account -> Devices -> Preferences -> Personal Document Settings", but I'm sure Amazon will [fiddle about](https://youtu.be/AOo1uhHb-jk?si=L6TgWnKfH-0j4zlV&t=84) with these menus at some point, so, good luck I guess.

If you've noticed this is all a big pain in the ass, I hear they [never have troubles, no troubles at all](https://archive.org/details/i-had-trouble-in-getting-to-sol/page/n31/mode/2up), in Kobo-land. YMMV.