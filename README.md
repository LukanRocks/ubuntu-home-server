# Ubuntu Home Server

This repo contains my Docker Compose file and documentation for my home server with Ubuntu.

## Docs

## Docker Secrets
I created a secrets folder in the root and used the commands:

sudo chown root:root ./secrets
sudo chmod 600 ./secrets

to limit the access to root user.

Then, whenever I need to create a secret, I use:

sudo su
ls ./secrets
nano ./secrets/SECRET-NAME

### Backups
I have Duplicati running and backing up though FTP to my local TrueNas Scale.

To allow restoring each service persistant data, I created a backup schedule for each one.

To make this process easier I have a template on ./duplicati-base-config.json where I can just add folders and passwords.

I also change the ./.vscode/settings with a material icon definition so I visually know which folders are being backed up.

### Using VS Code
After installing ubuntu and Docker, I created a folder called 'services' where I do all my projects.

Because most of the time I'm accessing this folder through SSH using VSCode I ran this command to allow me to use VS code write/read files when remoting access though the file browser.

sudo chown -R $USER /home/lukanvanderlinde/services
