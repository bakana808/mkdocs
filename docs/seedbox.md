# Seedbox

## filebrowser - file browser webapp

### Installing
1. `docker pull filebrowser/filebrowser`

## mango - manga reader webapp

### Installing
1. `docker pull hkalexling/mango`
2. create `docker-compose.yml` (see https://hub.docker.com/r/hkalexling/mango)

### Running
3. `docker-compose up`

whatbox will probably prevent you from opening certain ports and will suggest another port, edit `docker-compose.yml` to use the suggested port.

