---
title: "Setting Up Dev Environment"
categories: [Dev,Web]
---

I guess this is a follow-up to a previous post [The Quest For The Ultimate Dev Environment](2019-11-17-The-Quest-For-The-Ultimate-Dev-Environment). Some of the setup has gotten simpler with advances in technology as well as synching.
I've recently rebuilt my PC with Windows 11 so these are some notes on the current set up.

### Install WSL2 ###
This has gotten much easier. Simply open a terminal window and type `wsl2`. All the relevant dependencies are installed automatically.

### Install Docker Desktop ###
Again, Docker Desktop now includes `docker-compose` so simply install. WSL2 support is out-of-the-box. 

### Install VSCode ###
Install VSCode and sign in to sync your extensions with GitHub. Now you don't need to remember what you had that worked well or not.

#### Configure VSCode for XDebug ####
Your `.vscode/launch.json` should look something like this (assuming you are using the `twohill/silverstripe4` docker image)

```json
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Listen for Xdebug",
            "type": "php",
            "request": "launch",
            "port": 9003,
            "pathMappings": {
                "/var/www/html": "${workspaceFolder}"
            },
            "hostname": "localhost"
        }        
    ]
}
```

### Install relevant PHP versions and modules ###

```bash
sudo add-apt-repository ppa:ondrej/php
sudo apt-get install php*-{curl,cgi,cli,fpm,pdo,gd,mbstring,mysqlnd,opcache,xml,zip,intl}
```

### Install composer ###
https://getcomposer.org/download/

### Get coding!! 🧑‍💻 ###
