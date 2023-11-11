# Docker local registry with GUI
This repo allow you to deploy a docker registry stack with a GUI management tool.

You can provide SSL certs and restrict access to each managed container.



## Requirements
- Docker



## Prepare
- put your certs in the `certs/` directory (certs/domain.pem, certs/domain.pem) and edit `config/dynamic/tls.yml` (you can edit it when then container is running).
- certs **must be in .pem**
- set your own labels in the `docker-compose.yml` file.



## Install/Launch
- run `docker compose up -d` to start all the stack.



## Containers
- **registry-server** : https://hub.docker.com/_/registry
- **registry-ui** : https://github.com/Joxit/docker-registry-ui
- **reverse-proxy** : https://github.com/traefik/traefik 



## Entrypoints
Two entrypoints are configured in the `config/traefik/traefik.yml` file : 
- `web` (80)
- `websecure` (443) 

Feel free to add as much entrypoints as you need.

## Docker-compose labels
Traefik use the labels to redirect flux ; in a same container, all labels must have the same **[application_name]**

### BASIC PARAMS (required)
This parameter is used to **set the url** (aka Vhost)
- traefik.http.routers.**[application_name]**.rule=Host(\`**[url]**\`)

This parameter allow you to specify the **entrypoint** to use
- traefik.http.routers.**[application_name]**.entrypoints=**[entrypoint]**

### SSL (not required)
This parameter is used to **enable ssl**
- traefik.http.routers.**[application_name]**.tls=true

**Ensure that the entrypoint is properly configured too :** 
- traefik.http.routers.**[application_name]**.entrypoints=**websecure**


### AUTH (not required)
Equivalent of .htaccess / .htpasswd
- traefik.http.middlewares.**[application_name]**.basicauth.users=**[username:password,username:password,username:password ...]**

- generate a login with the command `docker run --rm --name apache httpd:alpine htpasswd -nb username password`

- don't forget to **double the $** : 
  - ✅ username**$$**data**$$**data
  - ❌ username**$**data**$**data

### HTTPS REDIRECT (nor required)
This parameter is used to **redirect http trafic to https** ; [application_name] is **NOT** required

- traefik.http.middlewares.httpsonly.redirectscheme.scheme=https
- traefik.http.middlewares.httpsonly.redirectscheme.permanent=true
- traefik.http.routers.httpsonly.rule=HostRegexp(`{any:.*}`)
- traefik.http.routers.httpsonly.middlewares=httpsonly



## Networks
All containers needs to be in the same network web ; internal network is here if containers need to communicate between themselves (website <> database )

Ensure all containers have the netwok block :
```
    networks:
      - web
```

### Versions
Developped with : 
- Registry 2.8.2
- Docker-registry-ui 2.5.6
- Traefik 2.10.5