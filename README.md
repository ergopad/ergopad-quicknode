# Get going with ERGO node, FAST!
These steps are mean to prepare an ergonode to sync quickly.<br>
<br>
It can still take 30 mins or so to download the blockchain file and extract, so consider keeping that file around.

## prereqs
> `apt update && apt install p7zip-full -y`

## download blockchain at ~700k height
this may change at some point
> `wget -O ergo.7z https://pixeldrain.com/api/file/2rXoYooy?download`

### extract .ergo and optionally remove 7z archive
> `7z x ergo.7z`<br>
> `rm ergo.7z`

## download ergo jar file
You may prefer to keep the original filenames and cp to ergo.jar for testing, and to avoid re-downloading.<br>
https://github.com/ergoplatform/ergo/releases<br>
<br>
One way to minimize file space is to map the current jar file to /app/ergo.jar in compose.yml
> `wget https://github.com/ergoplatform/ergo/releases/download/v4.0.24/ergo-4.0.24.jar`

## start node
> `docker compose up -d`

<hr>

# Additional

## Don't forget to update your API hash
update the ergo.conf file with your key from the node blake2 api call
> `http://localhost:9053/swagger#/utils/hashBlake2b`

## If you want to refresh the container
If you change something in the conf file or update to latest jar file, you may want to refresh your instance:
> `docker compose --force-recreate -d`
