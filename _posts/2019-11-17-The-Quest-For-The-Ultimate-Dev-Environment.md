---
title: "The Quest for the Ultimate Dev Environment"
categories: [Dev,Web]
---

I've been a long-time user of PhpStorm for my PHP and Javascript (React) development. It's an extremely powerful tool but let's face it - it's a pretty big beast. You can tell it's running by the fans on the laptop spinning up.

## Enter WSL2 ##

I recently rebuilt my PC using the latest Windows Insider build in order to take advantage of Docker on WSL2 - this is a _massive_ step-change from WSL1 in terms of performance and reliability. Unfortunately, PhpStorm doesn't support it, and doesn't look like it will any time soon. So, I decided to step out on a limb and give Visual Studio Code a shot.

## Visual Studio Code - First Impressions ##

Number one: it is fast. Unbelievably so! The workspace opens instantly, and there's no delays searching. There's some magic pixie dust in there I'm sure.

Number two: VSCode Remote is a game-changer. Essentially the editor is split into a client/server, and the server runs directly in my WSL2 virtual machine. Broken file watchers are a thing of the past.

Number three: for PHP development, it's kinda...basic. At least to start with. As I've discovered, there are a plethora of extensions that enhance the editor in almost every way imaginable.

## Configuration ##

Because a large amount of the functionality is in the extensions, I'm likely to forget what I've installed and how I've configured it, so here is a rundown of what I've set up.

### Basics ### 

You need to be running Windows 10 build 18917 (currently Insider Preview) in order to get and [install WSL2](https://docs.microsoft.com/en-us/windows/wsl/wsl2-install). You'll also need Docker build 36883 or higher (currently edge channel) and [enable WSL2 in Docker](https://docs.docker.com/docker-for-windows/wsl-tech-preview).

### Extensions ###

With VSCode Remote, extensions can be installed locally (ie in your profile or workspace) or remotely. WSL2 acts as a remote environment. So, the first extension we need, and the one you'll probably be prompted for when VSCode detects the presence of WSL2 is  [Remote - WSL](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-wsl). 

I do have some other extensions installed locally, but they're mainly for convenience, not development.

Extensions installed in WSL:

 * [Code Spell Checker](https://marketplace.visualstudio.com/items?itemName=streetsidesoftware.code-spell-checker) - I'm a terrible speller, and it also helps to keep on top of variable misnaming
 * [Composer](https://marketplace.visualstudio.com/items?itemName=ikappas.composer) - PHP's defacto package manager. Handles all the updates and stuff
 * [Docker](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-docker) - Docker integration
 * [Docker Compose](https://marketplace.visualstudio.com/items?itemName=p1c2u.docker-compose) - Docker Compose integration
 * [PHP Debug](https://marketplace.visualstudio.com/items?itemName=felixfbecker.php-debug) - PHP Debug tools
 * [PHP DocBlocker](https://marketplace.visualstudio.com/items?itemName=neilbrayfield.php-docblocker) - Helpers for writing php docblocks
 * [PHP Intelephense](https://marketplace.visualstudio.com/items?itemName=bmewburn.vscode-intelephense-client) - Intelisense for your PHP. This one is critical - I paid the ~$15 for the pro version
 * [PHP-CS-Fixer](https://marketplace.visualstudio.com/items?itemName=higoka.php-cs-fixer) - Fix any issues phpcs picks up
 * [phpcs](https://marketplace.visualstudio.com/items?itemName=ikappas.phpcs) - Code sniffer for bad php code

 A couple of points:

  * To get the most out of Intelephense you need to disable the built-in PHP Language Features (follow the quickstart guide)
  * Make sure you set the phpcs.standard to what your project adheres to, otherwise all the warnings will make you really sad

## Wrap-up ##

That's all for now. I'm new to VSCode so if you've got any great tips let me know via the comments. As I iterate on this environment I'll try and keep this page updated.