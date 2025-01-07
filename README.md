# Ubuntu Home Server

This repo contains my Docker Compose file and documentation for my home server with Ubuntu.

## Docs
After installing ubuntu and Docker, I created a folder called 'services' where I do all my projects.

Because most of the time I'm accessing this folder through SSH using VSCode I ran this command to allow me to use VS code write/read files when remoting access though the file browser.

sudo chown -R $USER /home/lukanvanderlinde/services/docker-compose.yaml

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