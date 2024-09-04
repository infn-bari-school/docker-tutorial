You can get the basic information about your Docker configuration by executing:

=== "Command"
```bash
docker info
```

You will get something like the following output:

```bash linenums="1" hl_lines="19 30 31 32 53"
Client: Docker Engine - Community
 Version:    27.2.0
 Context:    default
 Debug Mode: false
 Plugins:
  buildx: Docker Buildx (Docker Inc.)
    Version:  v0.16.2
    Path:     /usr/libexec/docker/cli-plugins/docker-buildx
  compose: Docker Compose (Docker Inc.)
    Version:  v2.29.2
    Path:     /usr/libexec/docker/cli-plugins/docker-compose

Server:
 Containers: 0
  Running: 0
  Paused: 0
  Stopped: 0
 Images: 0
 Server Version: 27.2.0
 Storage Driver: overlay2
  Backing Filesystem: extfs
  Supports d_type: true
  Using metacopy: false
  Native Overlay Diff: true
  userxattr: false
 Logging Driver: json-file
 Cgroup Driver: systemd
 Cgroup Version: 2
 Plugins:
  Volume: local
  Network: bridge host ipvlan macvlan null overlay
  Log: awslogs fluentd gcplogs gelf journald json-file local splunk syslog
 Swarm: inactive
 Runtimes: runc io.containerd.runc.v2
 Default Runtime: runc
 Init Binary: docker-init
 containerd version: 472731909fa34bd7bc9c087e4c27943f9835f111
 runc version: v1.1.13-0-g58aa920
 init version: de40ad0
 Security Options:
  apparmor
  seccomp
   Profile: builtin
  cgroupns
 Kernel Version: 6.8.0-40-generic
 Operating System: Ubuntu 24.04 LTS
 OSType: linux
 Architecture: x86_64
 CPUs: 2
 Total Memory: 3.824GiB
 Name: tutorvm-1
 ID: 56833310-4f20-4141-9869-f6ff1e988f66
 Docker Root Dir: /var/lib/docker
 Debug Mode: false
 Experimental: false
 Insecure Registries:
  127.0.0.0/8
 Live Restore Enabled: false
```

The output contains information about your storage driver, your docker root directory, the supported plugins (volume, network, log), the default registry, etc.
