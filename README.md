[![](https://badge.imagelayers.io/sedlund/acdcli:latest.svg)](https://imagelayers.io/?images=sedlund/acdcli:latest 'Get your own badge on imagelayers.io')

# sedlund/acdcli

Alpine Linux base with [acd_cli](https://github.com/yadayada/acd_cli) and fuse installed

## Usage Opportunities:

### Pass your docker hosts local directory where your acd_cli oauth files are and run a listing

    docker run -it --rm -v /home/ubuntu/.cache/acd_cli:/home/user/.cache/acd_cli sedlund/acdcli ls
----

* This is a good line to wrap in a script.

### Create a data volume that will contain your oauth, and other acd_cli files

    docker run -d --name acdcli-data -v /home/user tianon/true

### Fix the permissions of the home directory in the container

    docker run --rm --volumes-from acdcli-data busybox chown -R 1000:1000 /home/user

### Generate an oauth token in that container and sync

    docker run -it --rm --volumes-from acdcli-data sedlund/acdcli init

This will start elinks for you to authorize the connection and create your token.  Save the file in ~/.cache/acd_cli/oauth_data

    docker run -it --rm --volumes-from acdcli-data sedlund/acdcli sync
    
### Copy existing token inside the data container and set permissions

    docker cp ~/.cache/acd_cli acdcli-data:/home/user/.cache
    docker run --rm --volumes-from acdcli-data busybox chown -R 1000:1000 /home/user

### Mounting via fuse inside the container

    docker run -itd --privileged --name acdmount --volumes-from acdcli-data --entrypoint=/bin/sh sedlund/acdcli -c "acdcli -v mount /acd; sh"
    docker run -itd --cap-add SYS_ADMIN --device /dev/fuse --name acdmount --volumes-from acdcli-data --entrypoint=/bin/sh sedlund/acdcli -c "acdcli -v mount /acd; sh"
----

* `--entrypoint=/bin/sh` is needed to change the default entrypoint
* `--cap-add SYS_ADMIN` is needed for using FUSE
* `--device /dev/fuse` is needed for FUSE
* `--privileged` is required to use the docker hosts /dev/fuse for mounting.
* passing `-c "acdcli -v mount; sh"` to sh shows how to run a command that would fork (causing the container to exit), and running sh to keep it running.

**NOTE**: You cannot export a fuse mount as a volume from the container.  Check out `sedlund/acdcli-webdav` (based on this image) to mount ACD and provide a secure webdav share.  You could do other things like setup a sftp server in this container, but that is left as an excersize to the user.

### Using it to upload files from your Docker host
    docker run -it --privileged --rm --name acdmount --volumes-from acdcli-data -v "$(pwd)":/mnt --entrypoint=/bin/sh sedlund/acdcli

