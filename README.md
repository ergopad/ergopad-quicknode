# Get going with DOCKER and ERGO node, FAST!
These steps are meant to prepare an ergonode to sync starting at block ~700k.<br>
<br>
It can still take some time to download the blockchain file, extract and catch up to current height, so consider keeping that file around.

## prereqs
Make sure you have the tools to extract a 7z file (for Ubuntu):
> `apt update && apt install p7zip-full -y`

## download blockchain at ~700k height
This URL may change at some point:
> `wget -O ergo.7z https://pixeldrain.com/api/file/2rXoYooy?download`

### extract .ergo and optionally remove 7z archive
Extract `ergo.7z` from the download directory in to `/data/ergo/`; this will create a .ergo folder and the path should match what is in `compose.yml`
> `cd /data/ergo`<br>
> `7z x /download/ergo.7z -o/data/ergo/`<br>
> `rm ergo.7z`<br>

_Note: this is a good folder to keep securables in also, like ergo.7z and ergo.conf_

## download ergo jar file
The `compose.yml` reference to this file is setup to map the proper versioned name to ergo.jar, allowing easy switching between versions.<br>
One way to minimize file space is to map the current jar file to /app/ergo.jar in compose.yml
> `wget https://github.com/ergoplatform/ergo/releases/download/v4.0.30/ergo-4.0.30.jar`<br>

_Note: to update to the latest version, after these steps you can simply restart the container, but be sure to update config and volume mappings_
> `docker compose up -d --force-recreate`

## create your ergo.conf
Create ergo.conf in the folder with all the other repo files.  This file will be referenced in the compose.yml file as: `./ergo.conf`
```ini
ergo {
    directory = "/ergo/.ergo"
    wallet.secretStorage.secretDir = ${ergo.directory}"/wallet/keystore"

    node {
        # if you wish to allow mining on node; be aware of EIP-27
        mining = true
    }

    # added in 4.0.31, if mining = true, this must be set
    chain {
        reemission {
            checkReemissionRules = true
        }
    }
}

scorex {
    # http://localhost:9053/swagger#/utils/hashBlake2b
    restApi {
        # changeme
        apiKeyHash = "261158787da7ccef7220e347ab82bb58fed02e9b5f399f7594a6ed9b176df2f7"
    }

    # this affects performance a TON
    network {
        declaredAddress = "[my.public.ip.address]:9030"
    }
}
```

## .ergo folder
Make sure you have proper permissions to the .ergo folder.  This is currently setup as a generic location: `/data/ergo/.ergo`<br>
<br>
Creating the network allows any container on the system to connect 
directly to the node

## start node
> `docker network create ergopad-net`
> `docker compose up -d`

## check node logs from cli
> `docker logs -n 50 -f quicknode`

<hr>

# Additional

## Don't forget to update your API hash
update the ergo.conf file with your key from the node blake2 api call
> `http://localhost:9053/swagger#/utils/hashBlake2b`

## If you want to refresh the container
If you change something in the conf file or update to latest jar file, you may want to refresh your instance:
> `docker compose up --force-recreate -d`<br>
> `docker compose up --build -d` (if you make changes to the dockerfile)

## Restarting the container
Ideally, you will always use the API endpoint, `/shutdown`, however it has been noticed that even with this you can receieve an error indicating that the node may not have been shutdown correctly.  Try to restart the docker (or perhaps the compose version --force-recreate above).  After a restart, check that the node continues to increment the current height.  If configured properly, you should see peers connect and the current height begin to increment in about 30 seconds.
> `docker restart quicknode`<br>

## Troubleshooting
- Sometimes during initial sync, there seems to be some issue that causes CPUs to throttle.  This does not seem to affect ARM systems, so perhaps a docker or JVM implementation issue.  If this happens, consider rebooting.  This does not appear to happen once the node is caught up with current.
- When behind a firewall, port forward to port 9030 for the node.  Without this config, the node appears to sync slowly and is more prone to stalling -and sometimes unable to recover from the stall.
- Although preferred by some, using docker may not work as well as running locally, which will require the proper JDK version to be installed locally.  There seem to be less issues with this configuration.