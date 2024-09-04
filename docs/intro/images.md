The storage location of Docker images and containers depends on the operating system. The command `docker info` provides information about your Docker configuration, including the `Storage Driver` and the `Docker Root Dir`.

On Ubuntu, Docker stores images and containers files under `/var/lib/docker`:

```bash
total 44
drwx--x--x 4 root root 4096 Sep  4 10:46 buildkit
drwx--x--- 3 root root 4096 Sep  4 10:53 containers
-rw------- 1 root root   36 Sep  4 10:46 engine-id
drwx------ 3 root root 4096 Sep  4 10:46 image
drwxr-x--- 3 root root 4096 Sep  4 10:46 network
drwx--x--- 6 root root 4096 Sep  4 10:53 overlay2
drwx------ 4 root root 4096 Sep  4 10:46 plugins
drwx------ 2 root root 4096 Sep  4 10:46 runtimes
drwx------ 2 root root 4096 Sep  4 10:46 swarm
drwx------ 2 root root 4096 Sep  4 10:53 tmp
drwx-----x 2 root root 4096 Sep  4 10:46 volumes
```

Docker images are stored in `/var/lib/docker/overlay2`.

Let's explore the content of our image `hello-world`:

???+ info inline end
     The command `docker inspect` returns low-level information on Docker objects (images, containers, networks, etc.).

     More info in the Docker official [doc](https://docs.docker.com/engine/reference/commandline/inspect/){:target=_blank}.

```bash
docker image inspect hello-world
```

```bash
[
    {
        "Id": "sha256:d2c94e258dcb3c5ac2798d32e1249e42ef01cba4841c2234249495f87264ac5a",
        "RepoTags": [
            "hello-world:latest"
        ],
....
        "GraphDriver": {
            "Data": {
                "MergedDir": "/var/lib/docker/overlay2/7fea1105b18fd2b9fb8237cd6ff7bf2e2ba1521012006625ef2b1542e9b5f285/merged",
                "UpperDir": "/var/lib/docker/overlay2/7fea1105b18fd2b9fb8237cd6ff7bf2e2ba1521012006625ef2b1542e9b5f285/diff",
                "WorkDir": "/var/lib/docker/overlay2/7fea1105b18fd2b9fb8237cd6ff7bf2e2ba1521012006625ef2b1542e9b5f285/work"
            },
            "Name": "overlay2"
        },
....
```

The `GraphDriver.Data` dictionary contains information about the layers of the image.

In general, the **LowerDir** contains the read-only layers of an image. The read-write layer that represents changes are part of the **UpperDir**.

The `hello-world` image is built starting from the base image [`scratch`](https://hub.docker.com/_/scratch){:target="_blank"} that is an explicitly empty image. 

The `hello-world` image therefore contains just one layer that adds the `hello` executable (statically linked) to the base empty image. 

Look at the content of the **UpperDir**:

```bash
sudo ls -latr /var/lib/docker/overlay2/7fea1105b18fd2b9fb8237cd6ff7bf2e2ba1521012006625ef2b1542e9b5f285/diff 
total 24
-rwxr-xr-x 1 root root 13256 Dec 15  2023 hello
drwxr-xr-x 2 root root  4096 Sep  4 10:53 .
drwx--x--- 3 root root  4096 Sep  4 10:53 ..
``` 

Another interesting command is `docker history` that shows the hystory of an image. Try it!

```bash
docker history hello-world
```

Output:
```bash
IMAGE          CREATED         CREATED BY                SIZE      COMMENT
d2c94e258dcb   16 months ago   CMD ["/hello"]            0B        buildkit.dockerfile.v0
<missing>      16 months ago   COPY hello / # buildkit   13.3kB    buildkit.dockerfile.v0
```

Again, you can see here that the image has only one layer (0B lines are neglected).
 
