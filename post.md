**Difficulty level:** Beginner

**Requirements:** Mac OS X (This tutorial assumes you're using a Mac, but you can find installation instructions for [Windows](https://docs.docker.com/installation/windows/) or [Ubuntu](https://docs.docker.com/installation/ubuntulinux/) and skip ahead the _Setup_ section)

[Docker](https://www.docker.com/) has just celebrated its [2nd birthday](https://blog.docker.com/2015/03/dockers-2nd-birthday-wishes-qa-with-solomon-hykes-founder-of-docker/), but it's still a "new" powerful piece of technology. A lot of developer friends that I talk to have either heard or read about it but haven't actually used it. It lets you do really cool things like quickly test your app in development with the exact same environment as in QA/Test/Production, or share that app with other developers for a quick a painless onboarding. A commonly used analogy for Docker is to **compare it to actual real-life containers or lego bricks**: it provides a **fundamental unit**, and with it a way for **an application** to be **portable and moveable**, regardless of hardware.

In this tutorial, I'll give a quick overview of what Docker is and why you might want to use it, how to install it, and then we'll work on setting up a Node container and creating an express starter app inside it. **This is a long tutorial**! The official Docker getting started guide gets you up and running quicker, what I aim to do here **is explain what's happening on each step along the way**.

**What we’ll cover:**

*   Introduction (What's Docker and why use it)
*   Installation
*   Docker Hub and Dockerfiles
*   Docker Pull: Pulling an Ubuntu image
*   Docker Run: Running our Ubuntu image and accessing the container
*   Docker Commit: Installing node, npm, express and committing the changes
*   Docker Push: Pushing our container back so other people can use it

#### Notes:

I'll be referring to commands executed in your own terminal with:

```bash,linenums=true
$ command
```

And commands inside a container with:

```bash,linenums=true
$ root: command
```

#### Introduction

You've probably heard of Docker by now. Every day there's some front-page HackerNews mention of it, or you see people on Twitter/IRC talking about it. Its popularity has grown enormously in the past couple years, and most cloud providers already support it. If you are curious about it, but still haven't tried it out, this tutorial is for you. ☺

Okay, so what is Docker? Well, Docker can be a reference to a few things:

*   **Docker client:** this is what's running in our machine. It's the docker binary that we'll be interfacing with whenever we open a terminal and type `$ docker pull` or `$ docker run`. It connects to the docker daemon which does all the heavy-lifting, either in the same host (in the case of Linux) or remotely (in our case, interacting with our VirtualBox VM).
*   **Docker daemon:** this is what does the heavy lifting of building, running, and distributing your Docker containers.
*   **Docker Images:** docker images are the blueprints for our applications. Keeping with the container/lego brick analogy, they're our blueprints for actually building a real instance of them. An image can be an OS like Ubuntu, but it can also be an Ubuntu with your web application and all its necessary packages installed.
*   **Docker Container:** containers are created from docker images, and they are the real instances of our containers/lego bricks. They can be started, run, stopped, deleted, and moved.
*   **Docker Hub (Registry):** a [Docker Registry](https://github.com/docker/docker-registry) is a hosted registry server that can hold Docker Images. Docker (the company) offers a public Docker Registry called the Docker Hub which we'll use in this tutorial, but they offer the whole system open-source for people to run on their own servers and store images privately.

Now that we cleared the different parts of Docker, here are a few reasons why you might want to use it:

*   Simplifying configuration of a development environment
*   Quickly testing your app in an environment similar to QA/Test/Production (less overhead compared to VMs)
*   Sharing your app+environment with other developers, which allows for fast/reliable onboarding.
*   Ability to diff containers (this can be immensely useful in debugging)

#### Installation

Running a container, and therefore Docker, requires a Linux machine. Since we're using a Mac, that means we'll need a VM. To make the installation process easier, we can use Boot2Docker which installs the Boot2Docker management tool, VirtualBox, and sets up a VM inside it with Docker installed.

<p align="center">
  <img src="https://d262ilb51hltx0.cloudfront.net/max/1600/1*HOqvVFhyxu52LxzLrL6z6Q.png" alt="boot2docker"/>
</p>

Head over to this link to download the latest release of Boot2Docker, and install it (Boot2Docker-1.5.0.pkg at the time this was written):

[https://github.com/boot2docker/osx-installer/releases/latest](https://github.com/boot2docker/osx-installer/releases/latest)

After the installation is done, go to your Applications folder and open Boot2Docker. That's going to open a new terminal and run a few commands which basically start a VM that already has Docker installed, inside VirtualBox, and then sets a few environment variables so we can access the VM from our terminal. If you don't want to always open Boot2Docker to interact with Docker, just run the following commands:

```bash,linenums=true
# Creates a new VM if you don't have one  
$ boot2docker init

# Starts the VM  
$ boot2docker start

# Sets the required environment variables  
$ $(boot2docker shellinit)
```

Now type in:

```bash,linenums=true
$ docker run hello-world
```

That's gonna make Docker download the hello-world image from Docker Hub and start a container based on it. Your terminal should give you an output that says:

```bash,linenums=true
Hello from Docker.  
This message shows that your installation appears to be working correctly.
```

Awesome! Docker is installed. ☺

(If you have any problems, feel free to ping me or you can find Docker's official installation instructions [here](https://docs.docker.com/installation/mac/))

#### Dockerfiles and Docker Hub

Before we move forward, I think it's important to understand what happened when we executed `$ docker run hello-world` so you're not just copy+pasting the next instructions. `docker run` is the basic command that we use to start a container based on an image while passing commands to it. In this case, we said, "Docker, start a container based on the image hello-world, no extra commands". Then it downloaded the image from Docker Hub and started a container inside the VirtualBox VM based on that image. But where does the hello-world image come from? That's where Docker Hub comes in. The Docker Hub, like we mentioned in the introduction, is the public registry containing container images to be used with Docker, created by Docker, other companies, and individuals. Here you can find the image for hello-world we just executed:

[Docker Hub Hello-World Image](https://registry.hub.docker.com/u/library/hello-world/)

Every image is built using a Dockerfile. In the description for the hello-world image, you can find a [link to its Dockerfile](https://github.com/docker-library/hello-world/blob/master/Dockerfile) which only has 3 lines:

```
FROM scratch  
COPY hello /  
CMD [“/hello”]
```

Dockerfiles are just text files containing instructions for Docker on how to build a container image. You can think of an image as a snapshot of a machine, and a container as being the actual running instance of the machine. Dockerfiles will always have the format:

```
INSTRUCTION arguments
```

So in our hello-world example, we can take a look at the root of the GitHub repo which contains the Dockerfile. The image is being created from another image called "[scratch](https://registry.hub.docker.com/u/library/scratch/)" (all Dockerfiles start with the FROM instruction), then copying the [hello file](https://github.com/docker-library/hello-world/blob/master/hello) to the root of the system, and finally running hello. You can also find the [contents of the hello file here](https://github.com/docker-library/hello-world/blob/master/hello.asm), which contains the output we just saw in our terminal.

#### Docker Pull: Downloading an Ubuntu image

Now that we know our Docker installation is correctly setup, let's start playing with it! Our next step is getting an Ubuntu image. To find an image we can either go to the [Docker Hub website](https://hub.docker.com/) or just run in the terminal:

```bash,linenums=true
$ docker search ubuntu
```

This is going to give a list of all the images containing Ubuntu in its name. This is what's shown in my terminal:

<p align="center">
  <img src="https://d262ilb51hltx0.cloudfront.net/max/1600/1*VyLXJYEaN9BQwQDRWsMZiw.png" alt="terminal docker search"/>
</p>

The output is sorted by number of stars in each image repository. You can see that there's an Official and Automated column there.

*   **Official images** are images maintained by the docker-library project and accepted by the Docker team. That means they adhere to a few guidelines found [here](https://docs.docker.com/docker-hub/official_repos/), some of which are living in a git repository and that repository being at least read-only so users can check its contents. You can count on those images for working correctly with Docker. Also, contrary to other images where you need to reference them to pull using USERNAME/IMAGE_NAME, these images can simply be referred to in commands by IMAGE_NAME (such as Ubuntu). All of their Dockerfiles can be found in [this organization](https://github.com/docker-library).
*   The automated column refers to [Automated Build](https://docs.docker.com/userguide/dockerrepos/#automated-builds) images. It simply means that the image is being built from a Dockerfile inside a GitHub or BitBucket repository, and it's automatically updated when changes are made to it.

Let's download the official Ubuntu image:

```bash,linenums=true
$ docker pull ubuntu
```

The `$ docker pull IMAGE_NAME` command is the way to explicitly download an image, but that is also done if you use the `$ docker run IMAGE_NAME` command, and Docker can't find the image you're referring to.

#### Docker Run: Running our Ubuntu image and accessing the container

We've got our Ubuntu image (our blueprint ☺). Now let's start a new container based on our image and pass a command to it:

```bash,linenums=true
$ docker run ubuntu /bin/echo ‘Hello world’
```

That should output in your terminal the message "Hello World". Well, it's pretty neat that we just started a container running a completely isolated instance of Ubuntu and executed a command, but that's not really useful.

So now, let's run a new container with Ubuntu and connect to it:

```bash,linenums=true
$ docker run -i -t ubuntu
```

> Note: The run command is huge (check `$ docker help run`) and we'll go more in-depth in the next blog post

The -t flag assigns a pseudo-tty or terminal inside our new container and the -i flag allows us to make an interactive connection by grabbing the standard in (STDIN) of the container. If it worked correctly, you should be connected to a terminal inside the container showing something like this:

```bash,linenums=true
$ root@c9989236296d:/# 
```

Run ls -ls and see that your running commands in the root of a Ubuntu system. ☺

<p align="center">
  <img src="https://d262ilb51hltx0.cloudfront.net/max/1600/1*rlznCcFV8BcJXKDxIf35Lw.png" alt="terminal ubuntu"/>
</p>

I think it's nice to stop for a minute and think about what we just did. This is just one of the awesome parts of containers. We just downloaded and started a container running Ubuntu. That happened (depending on your internet connection) in 5 minutes? Compare that to downloading a VM Ubuntu image and spinning up a new VM. That would probably take you around 15–30min? And then creating new VMs, stopping, rebooting, how long that would take? When you add all of those up, the time you can save using containers is enormous!

#### Docker Commit: Installing node, npm, express and committing the changes

Okay, now that we are inside a running Ubuntu container, let's install the tools we need to run a node application (remember that you only need to execute the part after `$ root:` ):

```bash,linenums=true
$ root: apt-get update  
$ root: apt-get install nodejs  
$ root: apt-get install nodejs-legacy
```

> Note: We need to install `nodejs-legacy` to run the express-generator module

Running `node -v` should give you an output:

```bash,linenums=true
$ root: node -v  
v0.10.25
```

With node installed, we can go ahead and install the [express generator module](http://expressjs.com/starter/generator.html) from npm:

```bash,linenums=true
$ root: npm install -g express-generator
```

Now we have our container with everything we're gonna need installed in it. Let's go ahead and exit from our container:

```bash,linenums=true
$ root: exit
```

When we exit our container, Docker will stop running it. We can use the `$ docker ps` command to list containers, so let's do:

```bash,linenums=true
$ docker ps -a
```

<p align="center">
  <img src="https://d262ilb51hltx0.cloudfront.net/max/2000/1*l-NGj6PaW9F15wsiOo7OzA.png" alt="terminal containers"/>
</p>

The `$ docker ps` command by default only displays running containers, so we pass the -a flag so we can see our Ubuntu container we just exited.

Now we can use that container to create a new image that other people can use. We do that by using the `commit` command:

```bash,linenums=true
$ docker commit -a "Your Name &lt;youremail@email.com&gt;" -m "node and express" CONTAINER_ID node-express:0.1
```

> Note: Change the contents from the -a flag, and the CONTAINER_ID with the ID from your container shown in the `$ docker ps -a` output. You can use just the first 3/4 characters from the ID. ☺

The commit command takes a few parameters. The -a flag sets the author, you can set a message using the -m flag, and finally we reference our container ID and the name of the image we're creating, in this case `node-express`. We also set a tag for our image by adding the `:0.1` after the image name. If we run:

```bash,linenums=true
$ docker images
```

We should see:

<p align="center">
  <img src="https://d262ilb51hltx0.cloudfront.net/max/1600/1*1pF330DlUkoTsUrae8dRKg.png" alt="terminal images"/>
</p>

Awesome, you just created your first Docker image!

Now let's add another tag to our newly created image. Run:

```bash,linenums=true
$ docker tag node-express:0.1 node-express:latest
```

It's good practice to tag images with a specific version so people can know exactly which image they're running. Adding the `latest` tag helps so that other people can simply refer to your image when downloading it by its name (node-express in our case), and Docker will automatically download the `latest` tag version. If you run `$ docker images` again, you can see that there's two rows with our image, but they both have the same ID, which means they're not ocuppying any extra space in our HD. ☺

Now we can start as many containers as we want ready to go with our image! Let's remove our old container:

```bash,linenums=true
$ docker ps -a   
$ docker rm YOUR_CONTAINER_ID
```

> Note: Remember that you can just use the ID first 3–4 characters.

And let's run a container based on our new image, connect to it using the -i -t flags, and expose port 8080 of the host (VirtualBox) as the port 3000 of the container (VM):

```bash,linenums=true
$ docker run -i -t -p 8080:3000 node-express
```

Let's use the express-generator we installed to create a new Node.js app:

```bash,linenums=true
$ root: express mynodeapp
```

Following the instructions in the terminal, move to the app folder, install the dependencies and start the application:

```bash,linenums=true
$ root: cd mynodeapp  
$ root: npm install  
$ root: npm start
```

Now we have a Node.js application running inside a container, and exposing port 3000\. To see our application we need to find the Boot2Docker VM IP, so open another terminal and run:

```bash,linenums=true
$ boot2docker ip  
192.168.59.103
```

And remember that we actually exposed port 8080 of our container to access port 3000\. So go to your browser and open:

```bash,linenums=true
192.168.59.103:8080
```

Ta-ra!

<p align="center">
  <img src="https://d262ilb51hltx0.cloudfront.net/max/1600/1*NHmZcrvxfAl3jQ0Ow1N6lw.png" alt="website express"/>
</p>

Now, you might start wondering: this is a lot of work just to have a running application! I already have my development environment, I could have done all of that in 30 seconds! Well, that's true, but in this tutorial we're running a super simple application that doesn't have many dependencies. When you are running a real project that has much more dependencies, you may require a development environment with different packages, Python, Redis, MongoDB, Postgres, Node.js or io.js, etc. There're so many things involved that can make an application running in your computer **not run** correctly **in another machine** (or in QA/Test/Production), that is the main reason why Docker is so popular. Going back to the tutorial introduction, by providing **a fundamental unit** (our container/lego brick) that can be executed independent of hardware, and also easily run, moved, shared, Docker absolutely changes the way we can develop, test and share applications.

#### Docker Push: Pushing our container image so other people can use it

Okay, now let's share our "great" Ubuntu image with node and node-express installed so other people can also use it. Exit our running Node application and the container:

```bash,linenums=true
# Ctrl+C to stop our node app  
$ root: exit
```

Head over to Docker Hub and create a free account: [http://hub.docker.com](http://hub.docker.com)

After that, go back to your terminal and run:

```bash,linenums=true
$ docker login
```

Now that we're logged in in the cli we can push our image to the Docker Hub. Let's first rename it and add our username to it, so just like adding a tag:

```bash,linenums=true
$ docker tag node-express your_docker_hub_username/node-express  
$ docker rmi node-express  
$ docker push your_docker_hub_username/node-express
```

Done! Now anyone with Docker can execute:

```bash,linenums=true
$ docker pull your_docker_hub_username/node-express
```

And have the exact same environment with Ubuntu, Node.js, npm and the express-generator package as the one we previously created.

#### Next post: Adding Docker to an existing application, running and linking containers

This is a big introduction, and there's still a lot more to cover but you should be equipped with a basic understanding of what Docker is, how to use its basic functionality and ready to go more in-depth.

In the next tutorial, I'll cover adding a Dockerfile to an existing app, go more in-depth on the `$ docker run` command, mounting an app directory inside a container, and linking to another container running a MongoDB instance. ☺