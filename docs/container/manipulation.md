If you read the output from our `hello world`, they even recommend what to try next.

```bash
docker container run -it ubuntu bash
```
Let's see what happens:

```bash
Unable to find image 'ubuntu:latest' locally
latest: Pulling from library/ubuntu
31e907dcc94a: Pull complete
Digest: sha256:8a37d68f4f73ebf3d4efafbcf66379bf3728902a8038616808f04e34a9ab63ee
Status: Downloaded newer image for ubuntu:latest
root@0e539c9ccee4:/#
```

We are inside the docker container!

!!! tip
    You need to use the `-it` option whenever you want to run a container in interactive mode.
    - The `-i` or `--interactive` option connects you to the input stream of the container, so that you can send inputs to bash;
    - The `-t` or `--tty` option makes sure that you get some good formatting and a native terminal-like experience by allocating a pseudo-tty. 

### Playing with a running container 

This is a fully fledged Ubuntu host, and we can do anything we like in it. Let's explore it a bit, starting with asking for its hostname:

```bash
root@0e539c9ccee4:/# hostname
0e539c9ccee4
```

!!! tip
    A container's hostname defaults to be the container's ID in Docker. You can override the hostname using `--hostname`.

Let's have a look at the `/etc/hosts` file too.
```bash
root@0e539c9ccee4:/# cat /etc/hosts
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
ff00::0	ip6-mcastprefix
ff02::1	ip6-allnodes
ff02::2	ip6-allrouters
172.17.0.2	0e539c9ccee4
```
Docker has also added a host entry for our container with its IP address. Let's also check out its networking configuration.

```bash
root@0e539c9ccee4:/# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
6: eth0@if7: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
```
!!! warning "ip: command not found"
    Install the package `iproute2` that provides a collection of utilities for networking and traffic control.
    ```bash
       apt update && apt install -y iproute2
    ```

As we can see, we have the `lo` loopback interface and the `eth0@if7` network interface with an IP address of 172.17.0.2, just like any other host. 

We can also check its running processes:

```bash
root@0e539c9ccee4:/# ps aux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.0  0.0   4588  3968 pts/0    Ss   11:04   0:00 bash
root         334  0.0  0.1   7888  4224 pts/0    R+   11:08   0:00 ps aux
```

Note that the process `bash` has PID 1. 

Now type `exit` or the `CTRL-d` key sequence...you'll return to the command prompt of your Ubuntu host. So what's happened to our container? 
The container only runs for as long as the command we specified, `/bin/bash`, is running. Once we exited the container, that command ended, and the container was stopped.

So the container still exists but it's stopped:
```bash
docker container ps -a
CONTAINER ID   IMAGE         COMMAND    CREATED          STATUS                      PORTS     NAMES
0e539c9ccee4   ubuntu        "bash"     5 minutes ago    Exited (0) 15 seconds ago             frosty_satoshi
a621c49ed3d6   hello-world   "/hello"   15 minutes ago   Exited (0) 15 minutes ago             bold_galileo
```
### Starting a stopped container
We can start again our stopped container with `docker container start <container-id or container-name>`:

```bash
docker container start 0e539c9ccee4
0e539c9ccee4
```
Our container will restart with the same options we had specified when we launched it with the `docker run` command.

### Attaching to a container

The `docker container attach` command allows you to attach your terminal to the running container. 

!!! tip
    The command that is executed when starting a container is specified using the ENTRYPOINT and/or CMD instruction in the Dockerfile.
    The `attach` command allows you to connect and interact with the container’s main process which has `PID 1`.
    Remember that **if you kill the main process the container will terminate**.

This is useful when you want to see what is written in the standard output in real-time, or to control the process interactively.

So running the `attach` command on our Ubuntu container will bring us back to our bash prompt:
```bash
docker attach 0e539c9ccee4
root@0e539c9ccee4:/#
```

You can detach from a container and leave it running using the `CTRL-p CTRL-q` key sequence.

What happens if you type `exit`?

### Getting a shell to a container

The `docker exec` command allows you to run commands inside a running container.
The command can be run in background using the option `-d` or interactively using the option `-i`.

Try the following command on your Ubuntu container:

```bash 
docker exec -it 0e539c9ccee4 bash
root@0e539c9ccee4:/#
```
Let's look at the processes inside the container:
```bash
root@0e539c9ccee4:/# ps aux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.0  0.0   4588  3712 pts/0    Ss+  13:56   0:00 bash
root           9  0.1  0.0   4588  3968 pts/1    Ss   13:57   0:00 bash
root          17  0.0  0.1   7888  4096 pts/1    R+   13:57   0:00 ps aux
```
We can see that the `exec` command started a new shell session. 


!!! tip
    Usually the `exec` command is used to launch `bash` within the container and work with that. 
    The `attach` command primarily is used if you quickly want to see the output of the main process (`PID 1`) directly and/or want to kill it.
    

