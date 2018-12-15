---
title: Docker Basics
parent: Tutorial
permalink: /tutorial/docker-basics
nav_order: 2
---
# Docker Basics
{:.no_toc}


---
## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---
## Ensure Docker is working
Check to see which versions of docker and docker-compose you have installed by running 
`docker -v` and
`docker-compose -v`, respectively.

I'm currently on:

| Docker version 18.09.0, build 4d60db4
| docker-compose version 1.23.1, build b02f1306

Let's run the following command to check that Docker is working:
```
docker run hello-world
```

This will take the following steps:
- Check if the image 'hello-world' exists
- If not, fetch the image from docker hub (<https://hub.docker.com/>)
- Create a new container from the image
- Run the container

You should see something similiar to the following:  
![]({{site.baseurl}}/assets/docker-basics/hello-world-output.png)

---
## Containers and images

Images are an inert, immutable file that is essentially a snapshot of a container.

Containers are a lightweight and portable encapsulations of an environment in which to run applications which could simply be thought of as 'an instance of an image'.

We can see the images that currently exist with either `docker image ls` or `docker images`.

Given we previous ran "hello-world", we should see one image in our list:

![]({{site.baseurl}}/assets/docker-basics/docker-images-output.png)

We can see the active containers by running either `docker container ls` or `docker ps`.

However, we do not currently have any active containers. If we append `-a` to our last command, it will show _all_ containers.

![]({{site.baseurl}}/assets/docker-basics/docker-container-output.png)

Now we can see that we have one container with the _exited_ status.

---
## PID 1

When a container starts, it has a process assigned to PID 1. When PID 1 ends, the container will exit.

If you look at the previous screenshot, you will notice the "hello-world" container has the command "/hello" specified.  
This means that when the container starts up, it will run the program "/hello" which will print a message and then end.  
Given there is no longer a process running on PID 1, the container will exit.  

---
## Interactive Container
It is possible to interact with containers. To further explore the concept of PID 1, we need to understand this first.

Rather than doing this with the `run` command, we shall manually `pull`, `create`, and `start`.

1. Download the "busybox" image:
    ```
    docker pull busybox
    ```
    We can verify the image has downloaded with: `docker images`.

    ![]({{site.baseurl}}/assets/docker-basics/busybox-downloaded.png)

2. Create a new container:
    ```
    docker create busybox
    ```
    Upon creation it will display the container ID - this cannot be changed.

    Let's look at our containers with `docker ps -a`. Remember - we haven't started the container just yet, only created it (hence the `-a`). We can see it has the status "Created".

    To interact with the container we use either the ID on the name. Given we haven't specified a name, a unique one has been generated. In my case, it is "boring_roentgen".

    ![]({{site.baseurl}}/assets/docker-basics/busybox-view-id.png)

3. Start the container:
    ```
    docker start <ID|name>
    ```
    In place of `<ID|name>`, I could specify either "a65a4d556ba1" or "boring_roentgen".

    But... nothing happened. Check our containers again with `docker ps -a`. You'll notice the status of the busybox container has changed to "exited".  

    This is a similiar situation we experienced with "hello-world". "busybox" will run "sh" when started. However, given the standard input (STDIN) has not been kept open, the shell exited and thus so did our container.

4. Let's create an interactive busybox container:
    ```
    docker create --name busybox_interactive -it busybox
    ```

    `-i`: interactive, which keeps STDIN open  
    `-t`: allocate a pseudo-TTY, so we can interact with the terminal
    These are typically used together.  
    `--name`: as you may have guessed, set the name of the container.  

    To confirm, run `docker ps -a`. You should see a new busybox container with the name "busybox_interactive".

5. Start the container:
    ```
    docker start -i busybox_interactive
    ```

    Our terminal is now displaying the contents from inside the container.  

    To return our terminal to the host, we can either:  
    • Use the shortcut `Ctrl+P,Ctrl+Q`: this will keep the container running;  
    • `logout`, `exit`: this will end the process and thus the container will exit.  

    If the container is still running, we can reattach to the sh on PID 1 with: `docker attach <ID|name>`.

    ---
    Challenge
    {:.text-delta}
    Rather than using `pull`, `create`, and `start` like we've done above, it's easier and more common to simply use `run`.

    Using `run`, launch an interactive container named "busybox_interactive" from the "busybox" image.

    <small>_Given names must be unique, and we already have a container with the same name, is it not necessary to run this command. Simply note what you'd think it'd be._</small>

    <details><summary markdown="span">Solution</summary>
    `docker run -it --name busybox_interactive busybox`

    Note the similarly with the `create` command.  
    </details>

    ---

---
## PID 1 Explored Further

1. Start a new busybox container:
    ```
    docker run -it busybox
    ```
    This will be terminal-1.

2. View the processes inside the container with `ps`. We should see the following output:
    ![]({{site.baseurl}}/assets/docker-basics/container-ps-1.png)
    Note that PID 1 has the `sh` command assigned to it.

3. Open a new terminal window (terminal-2) and get the ID or name of the container with `docker ps`.  
    In my case, the name is `hungry_newton`.

4. Start a new `sh` process inside the container:
    ```
    docker exec -it hungry_newton sh
    ```
    We now have two `sh` processes running inside the same container.

5. In either terminal, type `ps` to view the running processes. We should see the following output:
    ![]({{site.baseurl}}/assets/docker-basics/container-ps-2.png)

6. In terminal-1, end PID 1 by typing `exit`.  
    Take note that terminal-2 has also exited. 

This confirmed that when PID 1 ends, the container will exit.  
We've also learned how to use `exec` to start a new process in the container.  

A container typically won't have a shell running on PID 1, so if we want to explore inside the container, using `attach` won't be of much use.  
Therefore, a common approach is to use `docker exec -it <ID|name> sh` to start a new shell process that we can navigate with.  

---
## Final Notes
We want containers to be disposable. If a container is stopped, killed, removed, etc., we should be able to simply spin up another one and our program should resume.

This means you should not be storing any state inside of the container. To store state, we mount a volume to the host and therefore store it on the host filesystem.

Keep this concept in mind going forward.
