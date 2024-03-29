# Selfhosting

## Requirements

Hardware: 

1. A computer. Linux or windows would work. 
2. Compatible hardware board and a shocker

Software: 

1. [Python 3.6+](https://www.python.org/downloads/) (Current latest should work fine.)
2. [Visual Studio Code](https://code.visualstudio.com/) with [PlatformIO addon](https://marketplace.visualstudio.com/items?itemName=platformio.platformio-ide)
3. [Docker Desktop](https://docs.docker.com/desktop/install/windows-install/) on Windows or [Docker Engine](https://docs.docker.com/engine/install/) on Linux.
4. node [TODO: input the rest of data]

## Preparing the server

Install software from the [Requirements](#requirements) on the server, make a new folder in a known location. In it, create a file called `docker-compose.yaml` and open it in VSCode or a preferred text editor. (editor with an integrated terminal is preferred)

### Docker compose setup

Configuration template, paste it in `docker-compose.yaml` file.

???Template
    ```yaml
    version: '3.8'

    services:
        postgres:
            image: postgres:16
            restart: unless-stopped
            networks:
                - openshock
            environment:
                - POSTGRES_PASSWORD=password
                - POSTGRES_USER=openshock
                - POSTGRES_DB=openshock
            volumes:
                - openshock-pg-data:/var/lib/postgresql/data

        redis:
            image: redislabs/redisearch:latest
            restart: unless-stopped
            networks:
                - openshock

        api:
            image: ghcr.io/openshock/api:latest
            restart: unless-stopped
            networks:
                - openshock
                - reverse-proxy
            depends_on:
                - postgres
                - redis
                - cron
            environment:
                OPENSHOCK__DB__CONN: Host=postgres;Port=5432;Database=openshock;Username=openshock;Password=password
                OPENSHOCK__REDIS__HOST: redis
                OPENSHOCK__FRONTENDBASEURL: https://custom-domain.com
                OPENSHOCK__COOKIEDOMAIN: custom-domain.com
                OPENSHOCK__MAIL__SENDER__EMAIL: admin@custom-domain.com
                OPENSHOCK__MAIL__SENDER__NAME: custom-domain-admin
                OPENSHOCK__MAIL__TYPE: SMTP
                OPENSHOCK__MAIL__SMTP__HOST: smtp.custom-domain.com
                OPENSHOCK__MAIL__SMTP__USERNAME: admin
                OPENSHOCK__MAIL__SMTP__PASSWORD: not-existing
                OPENSHOCK__TURNSTILE__ENABLE: False

        webui:
            image: ghcr.io/openshock/webui:latest
            restart: unless-stopped
            environment:
                - OPENSHOCK_NAME=Custom-name
                - OPENSHOCK_URL=https://custom-domain.com
                - OPENSHOCK_SHARE_URL=https://shared.custom-domain.com
                - OPENSHOCK_API_URL=https://api.custom-domain.com
            depends_on:
                - api
                - live_control
            networks:
                - reverse-proxy

        live_control:
            image: ghcr.io/openshock/live-control-gateway:latest
            restart: unless-stopped
            networks:
                - openshock
                - reverse-proxy
            environment:
                OPENSHOCK__DB: Host=postgres;Port=5432;Database=openshock;Username=openshock;Password=password
                OPENSHOCK__REDIS__HOST: redis
                OPENSHOCK__COUNTRYCODE: DE
                OPENSHOCK__FQDN: lcg.custom-domain.com

        cron:
            image: ghcr.io/openshock/cron:master
            restart: unless-stopped
            networks:
                - openshock
            environment:
                OPENSHOCK__DB: Host=postgres;Port=5432;Database=openshock;Username=openshock;Password=password
                OPENSHOCK__REDIS__HOST: redis

    networks:
        openshock:
        reverse-proxy:

    volumes:
        openshock-pg-data:
    ```

variables that must be changed:

* POSTGRES_PASSWORD, and the password field in every DB_CONN string (Password=PUT_NEW_PASSWORD_HERE;)
* Every instance of "custom-domain.com". it needs to be replaced with a domain you own.
* Everything in email configuration. consult api's README.MD for information on how to configure that [TODO: LINK]. add a note about that the config is not verified and you can put bollocks data, and the api will still work? tested as of version 2.5.2 of api


variables that can be changed:

* OPENSHOCK_NAME, can be set to whatever. it'll show in places around the web ui.
* subdomain parts of each url/fqdn. (note, when not running on a domain, the live-control gateway needs to be specified with a ip address in the firmware [TODO: check if this is actually true])
* OPENSHOCK__COUNTRYCODE, can be set to whatever. I'd leave it as default if you are running only one. It is used to select a gateway closest to you if there are multiple.

variables you shouldn't change

* Host=postgres and Port=5432 parts of the DB_CONN variables. The host must be the same as the name of the postgres service definition.
* OPENSHOCK__REDIS__HOST, same as above. Must equal name of the redis service definition.

For more information consult readme.md files of each service [TODO: add links]




### reverse proxy

1. Acquiring a domain
2. configure reverse proxy
3. making share links work

## Firmware setup

1. Install vscode
2. install latest python ( i used 3.11 )
3. install platformio
4. edit the env
   1. Note about the version popup
5. build and upload images onto the board

## Troubleshooting

1. use the monitor from platformio and logs from the api service, in most cases the error is a misconfiguration of the api