# Ubuntu Home Server

This repo contains my Docker Compose file and documentation for my home server with Ubuntu.

## Docs

### Using VS Code
After installing ubuntu and Docker, I created a folder called 'services' where I do all my projects.

Because most of the time I'm accessing this folder through SSH using VSCode I ran this command to allow me to use VS code write/read files when remoting access though the file browser.

sudo chown -R $USER /home/lukanvanderlinde/services

### Fixing HA trusted_proxies issue
HA requires a list of allowed ips for reverse proxy. That should be a easy thing to solve, just slap the nginx conteiner ip there and call it a day. However, because I have everything on one docer-compose file, everytime stuff gets restarted, the nginx ip changes. My understanding is that container creation happens at the same time, creating a race condition in the ip grabing. Sometimes ngix is ***.***.***.4  ***.***.***.8. This created a issue that I had to frequently update the HA config file with the new ip. After some research, I didn't found a simple solution to make the ngix container have a static ip. So I ended up making all services depend on ngix, this way everybody will wait ngix to spin up, that makes sure it has the first ip ***.***.***.2 and once that happens, the other services take whatever ip they can get.
    
    depends_on:
      - nginxproxymanager

### Setting up a new service
Everything stars by adding the service to my docker-compose file

Then, once the service is running I need to add it as a new proxy host to NGINX. For that I need to setup the following fields:
- Domain names: SERVICE.homeserver.lukan.rocks
- Forward Hostname: container_name
- Forward Port: container port
- SSL Certificate: Valid certificate
- Force SSL: true
- HTTP/2 Support: true

Now I can access the service by the defined domain name with https.

Now the service needs to be added to my starbase dashboard by editing the configuration.json and restarting the container using the CLI or Portainer.

Lastly, I need to add the service to Uptime Kuma.