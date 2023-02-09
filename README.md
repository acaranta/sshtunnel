# Docker SSH Tunnel

This Container is just an easy way to setup an ssh tunnel and let is be run by docker and/or docker-compose.

This version uses [autossh](https://linux.die.net/man/1/autossh) to ensure that the established tunnel is up and running, restarting it if need be.

# Usage & Example
## Forward remote port to local machine
The below example is the same as :
1. SSH'ing to `host.example.com` on port `22`
2. forwarding port `2525` on the local machine to `localhost (which is host.example.com)` port `25` using the identity file `keyfile`. 

NB. The `ports` setting is only required if you want to use the tunnel on the host. It is not required for other containers to use the tunnel.

The configuration would look like this :
![](img/sshtunnel%20example%20direct.png)

And the compose file for this configuration would be :

```
version: '2'

services:

  sshtunnel:
    image: acaranta/docker-sshtunnel
    container_name: sshtunnel
    ports:
      - "2525:2525"
    volumes:
      - ./sshtunnel/data/:/data/:ro
    environment:
      - TUNNEL_HOST=host.example.com
      - TUNNEL_PORT=22
      - REMOTE_HOST=localhost
      - REMOTE_PORT=25
      - LOCAL_PORT=2525
      - KEY=/data/keyfile
    restart: always
```

## Forward local port to remote machine
To use the tunnel in reverse mode, allowing a port on a remote server to redirect to a port on a local container, use the following. 

The below example is the same as :
1. SSH'ing to `host.example.com` on port `22`
2. Forwarding port `8080` on the remote machine to the container `nginx (which is on the local sshtunnel machine)` with port `80` using the identity file `keyfile`.

which would look like this :
![](img/sshtunnel%20example%20reverse.png)

```
version: '2'

services:

  sshtunnel:
    image: acaranta/docker-sshtunnel
    container_name: sshtunnel
    volumes:
      - ./sshtunnel/data/:/data/:ro
    environment:
      - REMOTE=true
      - TUNNEL_HOST=host.example.com
      - TUNNEL_PORT=22
      - CONTAINER_HOST=nginx
      - CONTAINER_PORT=80
      - REMOTE_PORT=8080
      - KEY=/data/keyfile
    restart: always
```

## Misc Options
* You can override and specify your own [ssh config file](https://man7.org/linux/man-pages/man5/ssh_config.5.html) by mounting it into the container and specifying it using the `SSHCONFIGFILE` Env Variable
* You can change the verbosity level of autossh using the `AUTOSSH_LOGLEVEL`, setting it to a number between 0-7, defaults to 1 in this container
* You can activate ssh debug level by setting `SSHLOGLEVEL` to "-v", "-vv", etc