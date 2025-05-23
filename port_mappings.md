# Port Mappings for THARMAS Services

This file tracks the host ports mapped to container services across different Docker Compose setups.

## Networking (`tharmas/networking/docker-compose.yml`)

| Service     | Host Port(s)         | Container Port(s) | Protocol(s) | Notes                    |
|-------------|----------------------|-------------------|-------------|--------------------------|
| adguardhome | 53                   | 53                | tcp, udp    | DNS                      |
| adguardhome | 3000                 | 3000              | tcp         | Web UI (Initial Setup)   |
| *caddy*     | *(80, 443)*          | *(80, 443)*       | *(tcp)*     | *(Currently Commented)* |

## Schooner (`tharmas/schooner/docker-compose.yml`)

| Service                           | Host Port(s) | Container Port(s) | Protocol(s) | Notes           |
|-----------------------------------|--------------|-------------------|-------------|-----------------|
| jellyfin                          | 8096         | 8096              | tcp         | Web UI          |
| jellyfin                          | 8920         | 8920              | tcp         | HTTPS (Default) |
| jellyfin                          | 7359         | 7359              | udp         | Auto Discovery  |
| jellyfin                          | 1900         | 1900              | udp         | Auto Discovery  |
| sabnzbd                           | 8080         | 8080              | tcp         | Web UI          |
| sonarr                            | 8989         | 8989              | tcp         | Web UI          |
| radarr                            | 7878         | 7878              | tcp         | Web UI          |
| lidarr                            | 8686         | 8686              | tcp         | Web UI          |
| calibre-web                       | 8083         | 8083              | tcp         | Web UI          |
| calibre-web-automated-downloader  | 8084         | 8084              | tcp         | Web UI          |