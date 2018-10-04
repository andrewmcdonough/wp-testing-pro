---
ID: 9677
post_title: 'Learn to program with Minecraft &#8211; Docker edition.'
author: Michael Ducy
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/learn-program-minecraft-docker-edition/
published: true
post_date: 2018-07-25 13:37:30
---
My son has started to show interest in programming so I've been looking for a way to get him started. He loves Minecraft so I picked him up a copy of "<a href="https://nostarch.com/programwithminecraft" target="_blank" rel="noopener">Learn to Program with Minecraft</a>" which shows you how to programmatically control and build in Minecraft using the Python library <a href="https://github.com/py3minepi/py3minepi" rel="noopener" target="_blank">py3minepi</a>. Unfortunately, my son immediately got stuck on trying to setup his development environment on his Mac, so we took the time to recreate the instructions leveraging modern tools such as <a href="https://www.docker.com/" target="_blank" rel="noopener">Docker</a> and <a href="https://code.visualstudio.com/" target="_blank" rel="noopener">Visual Studio Code</a>.

The books instructions were written to locally install Minecraft, Python 3.6, Spigot, and py3minepi. There's a url to use to download the versions of software that you need, and fairly detailed guides to installing it all. The challenge he had was if he hit an error, he was stuck in a debugging mess, trying to make changes to his local system to get things to work. This of course creates a bespoke local dev environment that's hard to recreate. Additionally, one of the first problems he hit was a conflict between the operating system's version of Python and the Python we needed to install, version 3.6. We could have used tools like <a href="https://realpython.com/python-virtual-environments-a-primer/" target="_blank" rel="noopener">virtual environments</a> to manage which version of Python we were using, but I wanted to reduce changes to the local system and wanted something that was a little more first timer friendly. So we looked at how we could run this all in Docker. You can grab all the code you need to run this environment in t<a href="https://github.com/sysdiglabs/program-minecraft" target="_blank" rel="noopener">his Github repository</a>.

[tweet_box design="default" float="none"]Get started programming with Minecraft quickly by leveraging @Docker and #containers.[/tweet_box] 

## Architecture and Setup {#architectureandsetup}

<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/08/Learning-to-Program-with-Minecraft-Docker-Edition.png" alt="" width="634" height="428" class="alignnone size-full wp-image-9680" />  
The architecture of the development environment is shown above. We will leverage Docker to run the server instance of Minecraft that we need to develop against. This gives us a quick and easy way to create disposable server instances, and it's simple to completely reset the Minecraft state if needed (delete everything in ./data). While we could install Python 3.6 locally, we can also run a Docker container running Python 3.6 to run our code. Our Python client connects to the Spigot Minecraft server and then issues commands to control the game. Also on our local workstation we're using Visual Studio Code as an IDE, and Minecraft to see the results of our Python scripts (like blocks or buildings we create). Note that we are doing this all on MacOS, but the experience should be similar on Windows (thanks to containers).

The first thing to do is install <a href="https://www.docker.com/get-docker" target="_blank" rel="noopener">Docker</a> and <a href="https://code.visualstudio.com/download" target="_blank" rel="noopener">Visual Studio Code</a>. Both provide the standard MacOS "drag to Applications" installation method. When you launch Docker, it will need to perform some additional steps to configure itself, thus you'll need to provide your password to finish the installation.

If you're using Visual Studio Code, you might want to have it installed in your $PATH so it's easy to launch from a command line. Open Code, and then open the command palette (Cmd+shift+P). Type in 'shell' and choose `Install code command in PATH`. This is a nice time saver because now you can run `code .` and automatically open Code in that directory.

## Creating our Python Development Container {#creatingourpythondevelopmentcontainer}

Instead of installing Python 3.6 locally, we can leverage a prebuilt Docker container to provide the language runtime. The Python maintainers <a href="https://hub.docker.com/_/python/" target="_blank" rel="noopener">publish container images</a> we can us for this purpose, as well as instructions for how add in your own code base. We will need to add in the py3minepi library that will allow us to communicate with the Minecraft server. To do this we created a small Dockerfile based on the `python:3.6` image, and run `pip` to install our client library.

    FROM python:3.6
    
    WORKDIR /usr/src/app
    
    RUN git clone https://github.com/py3minepi/py3minepi && pip install --no-cache-dir py3minepi/ && rm -fr py3minepi
    
    CMD [ "python" ]

We've published this Docker image on <a href="https://hub.docker.com/r/sysdiglabs/mc-pi-dev/" target="_blank" rel="noopener">Docker Hub</a> for anyone to use, or you can find the complete Dockerfile in our <a href="https://github.com/sysdiglabs/program-minecraft" target="_blank" rel="noopener">Github repo</a>. If you want to build your own Docker image, you can do so by running `docker build -t mc-pi-dev .` in the directory where you've placed the Dockerfile.

## Creating our Minecraft Server Instance {#creatingourminecraftserverinstance}

Now that we've created our Docker image for our development container, we can create the server container. Luckily there's already a <a href="https://github.com/itzg/dockerfiles/tree/master/minecraft-server" target="_blank" rel="noopener">container image</a> that can be used to run various types of Minecraft servers. We'll want to create three directories - data, plugins, and src - that we will use with the server and later the development container we created previously (if you're using the Github repo we provided, we've already created the directories for you).

    $ mkdir data plugins src

Next we should create a `docker-compose.yml` file to allow us to more easily stand up our server in a repeatable fashion. Docker Compose allows us to define settings we'll need to run the server, and we can avoid entering a long command line everytime we want to create a new server instance. We can also create an instance of our development container we created previously, and connect the to containers together via a private network.

To create our `docker-compose.yml` file we cribbed from <a href="https://github.com/itzg/dockerfiles/wiki/Minecraft-Pi" target="_blank" rel="noopener">an example</a> we found on Github. We modified this example to include the development container running python, as well as creating a link between the two containers.

    version: '3'
    services:
     dev-workstation:
       image: "sysdiglabs/mc-pi-dev"
       container_name: "mc-pi-dev"
       volumes:
         - "./src:/usr/src/app"
       tty: true
       stdin_open: true
       restart: always
       links:
        - minecraft-server
     minecraft-server:
       image: "itzg/minecraft-server"
       container_name: "mc-pi-srv"
       ports:
         - "25565:25565"
         - "4711:4711"
       volumes:
         - "./data:/data"
         - "./plugins:/plugins"
       environment:
         EULA: "TRUE"
         TYPE: "SPIGOT"
         VERSION: "1.11.2"
         MEMORY: "1G"
       tty: true
       stdin_open: true
       restart: always

The last bit to set up our server is to install a required plugin. In order to provide an API to the Minecraft server, a plugin called RaspberryJuice must be downloaded and installed in the `plugins` directory we created earlier. You can get a <a href="https://github.com/zhuowei/RaspberryJuice/blob/master/jars/raspberryjuice-1.11.jar" target="_blank" rel="noopener">precompiled version</a> on the Gtihub repo for the <a href="https://github.com/zhuowei/RaspberryJuice/" target="_blank" rel="noopener">RaspberryJuice</a> plugin. We downloaded version 1.11, as we are using version 1.11.2 for our Minecraft/Spigot server, and placed the jar in the `plugins` folder. Once the plugin is installed, we can start our containers by running `docker-compose up`. You should see the console output of your Minecraft server, and you'll know the server is up and running if you see messages such as below

<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/08/docker_minecraft_logs.png" alt="" width="1289" height="325" class="alignnone size-full wp-image-9679" />  


We can test the that the server is working by posting a simple message to the server's chat. We created a file called `hello.py` in the `src` directory. Any python source code you want to be able to run will need to be stored in the `src` directory, as this directory is mounted into the python development container. We added the below to the `hello.py` file.

    from mcpi.minecraft import Minecraft
    
    mc = Minecraft.create("minecraft-server")
    
    mc.postToChat("Hello World!")

You can open a shell on the python container by running `docker run -it mc-pi-dev bash` in another terminal. You can then execute the `hello.py` script by running `python hello.py`. You should see `Hello World` posted to the Minecraft server's console output.

## Specifying the Correct Minecraft Server {#specifyingthecorrectminecraftserver}

Throughout the code examples in the book "Learn to Program with Minecraft" the author calls the `Minecraft.create` method without any arguments. When using this Dockerized setup you'll need to specify the name (or IP) of the Minecraft server to connect to. Docker compose gives us an easy name that we can use to do just that. As shown in the above example, simply replace any calls to `Minecraft.create()` method listed in the book with `Minecraft.create("minecraft-server")`.

## Installing Minecraft {#installingminecraft}

My son already had Minecraft installed locally on his laptop, so we didn't need to go through that process. If you do need to install Minecraft, I also highly recommend using <a href="https://multimc.org/" target="_blank" rel="noopener">MultiMC</a> instead of the default Minecraft launcher. MultiMC makes it much easier to have multiple instances of Minecraft configurations, including difference versions or mods.

It's pretty amazing how easy Docker makes things for creating disposable development environments. My son now has a development environment he can use, easily reset, and, if he decides he doesn't want it any more, destroy. We didn't have to go through all the steps of installing lots of software on his laptop, and all the problems that we could have encountered throughout that process. Hopefully this guide will help you and your kid get programming with Minecraft quickly and easily.