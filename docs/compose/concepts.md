
Docker compose is a tool for defining and running multi-container Docker application.

### Docker compose file 123

Docker uses a YAML format file to define the multi-container application, also known as "docker compose file".

Following a simple example of a docker compose file:

```yaml
services:
  db:
    image: influxdb:2.1
    volumes:
    - db_data:/var/lib/influxdb2:rw
    ports:
    - "8086:8086"
    networks:
    - db_net

volumes:
  db_data:

networks:
  db_net:
```

In the YAML file you can notice an inner structure:

* the **services** keyword starts the section containing all services. 
* **db** is an arbitraty service name, it must be any unique identifier.
* inside each service, with the **image**, **volumes**, **ports** and **networks** keywords you can insert the same information used when you run a single container in the command line
* the **volumes** keyword allows to define volumes to be attach in the containers. 
* the **networks** keyword allows to define networks to be used in the containers. 

### Docker compose command to control the application status

With a single command, it is possible to create, start, stop and destroy all services contaiined in the application:

* **docker compose up**: initializes and starts the application environment
* **docker compose start**: starts the application environment
* **docker compose restart**: restarts the already created application environment
* **docker compose stop**: stops the application environment
* **docker compose down**: stops and destroys the application environment

#### Start your first multi-container application

Let's start a multi-container application. At this point it's not important the content of the docker compose file but that the docker-compose.yaml is inside the `prova_1` folder. 

Docker compose file:

```yaml
version: "3.8"
services:
  db:
    image: influxdb:2.1
    volumes:
    - db_data:/var/lib/influxdb2:rw
    ports:
    - "8086:8086"
    networks:
    - db_net

volumes:
  db_data:

networks:
  db_net:
```

Commands:

```bash
user@vm:~/prova_1$ docker compose up -d
✔ db 10 layers [⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿]  	0B/0B  	Pulled 39.4s                              
[+] Running 3/3
✔ Network prova_1_db_net	Created                                                                                                                 
✔ Volume "prova_1_db_data"	Created                                                                                                                
✔ Container prova_1-db-1	Started 

user@vm:~/prova_1$ docker container ps
CONTAINERID   IMAGE      	 NAMES
Be72e5..      influxdb:2.1     prova_1_db_1

user@vm:~/prova_1$ docker volume ls
DRIVER	VOLUME NAME
local 	46f4a..
local 	prova_1_db_data

user@vm:~/prova_1$ docker network ls
NETWORK ID 	NAME         	  DRIVER	SCOPE
51039c996454   bridge       	  bridge	local
85ffa41af96f   host          	  host  	local
d6f329122060   none         	  null  	local
ec31eefd60d5   prova_1_db_net   bridge	local
```

!!! tip
    Every object name (image, volume and network) starts with `prova_1` that represents the folder name containing the docker compose file. 

#### Stop your first multi-container application

Commands:

```bash
user@vm:~/prova_1$ docker compose down
[+] Running 2/2
 ✔ Container prova_1-db-1  Removed                                                                                                                
 ✔ Network prova_1_db_net  Removed

user@vm:~/prova_1$ docker container ps
CONTAINERID   IMAGE      	 STATUS          NAMES

user@vm:~/prova_1$ docker volume ls
DRIVER	VOLUME NAME
local 	46f4..
local 	prova_1_db_data

user@vm:~/prova_1$ docker network ls
NETWORK ID 	NAME  	DRIVER	SCOPE
51039c996454   bridge	bridge	local
85ffa41af96f   host  	host  	local
d6f329122060   none  	null  	local
```

Some comments:
- Volumes are not removed with `docker compose down`
- Compose preserves all volumes used by your services
- Data inside volume is not lost and can be reused once the app is restarted
- To remove volume as well, use `docker compose down --volumes` or `docker compose down -v`

### Project name

Compose uses project names to isolate multi-container applications from each others.

If not set, the folder name is taken as project name.

!!! tip
    A custom project name can be set using the `-p` command line option or the `COMPOSE_PROJECT_NAME` env var.

```bash
user@vm:~$ cd ..
user@vm:~$ cp -r prova_1/ prova_2/
user@vm:~$ cd prova_2

user@vm:~/prova_2$ docker compose up -d
[+] Running 3/3
✔ Network prova_2_db_net   Created
✔ Volume "prova_2_db_data" Created
✔ Container prova_2-db-1   Started

user@vm:~/prova_2$ docker ps
CONTAINERID     IMAGE           NAMES
297f44..        influxdb:2.1    prova_2_db_1

user@vm:~/prova_2$ docker compose down
[+] Running 2/2
✔ Container prova_2-db-1 Removed
✔ Network prova_2_db_net Removed
```

### Network

By default Compose sets up a single network for your app.

Each container joins the default network and is both reachable by other containers on that network and discoverable by them at a hostname identical to the container name.

If you make a conﬁguration change to a service and run `docker compose up` to update it, the old container is removed and the new one joins the network under a different IP address but the same name. So it is suggested to use container names as hostname instead of IP to connect services.

You can specify your own network with the top-level networks key. Following an example.

```yaml
networks:
  front:
    driver: bridge
    driver_opts:
      com.docker.network.enable_ipv6: "true"
    ipam:
      driver: default
      config:
        - subnet: 172.16.238.0/24
        gateway: 172.16.238.1
        - subnet: "2001:3984:3989::/64"
        gateway: "2001:3984:3989::1"
```

### Write your first docker compose file

A simple approach to learn how to write a docker compose file is to convert some command line instructions to the corresponding YAML format file, that are well managed from Docker Compose. 

```bash
docker volume create influxdb-storage
export INFLUXDB_USERNAME=admin
export INFLUXDB_PW=admin
docker container run --name influxdb
  -p 8086:8086 \
  -v influxdb-storage:/var/lib/influxdb \
  -e INFLUXDB_DB=db0 \
  -e INFLUXDB_ADMIN_USER=${INFLUXDB_USERNAME} \
  -e INFLUXDB_ADMIN_PASSWORD=${INFLUXDB_PW} \
  influxdb:2.1

docker volume create grafana-storage
export GRAFANA_USERNAME=admin
export GRAFANA_PW=admin
docker container run --name grafana
  -p 3000:3000 \
  -v grafana-storage:/var/lib/grafana \
  -e GF_SECURITY_ADMIN_USER=${GRAFANA_USERNAME} \
  -e GF_SECURITY_ADMIN_PASSWORD=${GRAFANA_PW} \
  grafana/grafana:8.3.4-ubuntu
```

The resulting YAML file is:

```yaml
version: '3.8'
services:
  influxdb:
    image: influxdb:2.1
    ports:
      - '8086:8086'
    networks:
      - my_net
    volumes:
  	  - influxdb-storage:/var/lib/influxdb
    environment:
  	  - INFLUXDB_DB=db0
  	  - INFLUXDB_ADMIN_USER=admin
  	  - INFLUXDB_ADMIN_PASSWORD=admin

  grafana:
    image: grafana/grafana:8.3.4-ubuntu
    ports:
      - '3000:3000'
    networks:
      - my_net
    volumes:
  	  - grafana-storage:/var/lib/grafana
    environment:
  	  - GF_SECURITY_ADMIN_USER=admin
  	  - GF_SECURITY_ADMIN_PASSWORD=admin

volumes:
  influxdb-storage: {}
  grafana-storage: {}

networks:
  my_net: {}
```

As you can see, there is a one-to-one correspondence between information in the command line instructions and the docker compose file.

#### Using default network definition

Docker Compose creates a default network if not present any network definitions in the docker compose file. In such a case, the section `networks` inside each application should be avoided as well. In this second scenario, the YAML file become: 

```yaml
version: '3.8'
services:
  influxdb:
    image: influxdb:2.1
    ports:
      - '8086:8086'
    volumes:
  	  - influxdb-storage:/var/lib/influxdb
    environment:
  	  - INFLUXDB_DB=db0
  	  - INFLUXDB_ADMIN_USER=admin
  	  - INFLUXDB_ADMIN_PASSWORD=admin

  grafana:
    image: grafana/grafana:8.3.4-ubuntu
    ports:
      - '3000:3000'
    volumes:
  	  - grafana-storage:/var/lib/grafana
    environment:
  	  - GF_SECURITY_ADMIN_USER=admin
  	  - GF_SECURITY_ADMIN_PASSWORD=admin

volumes:
  influxdb-storage: {}
  grafana-storage: {}
```
#### Environment variable definition

As you can see from the docker compose file above, the environment variable values are inserted directly inside the YAML file. This could be unacceplable in some cases.

A second approach allows to link existed environment variables by inserting their names inside the docker compose file.

```yaml
version: '3.8'
services:
  influxdb:
    image: influxdb:2.1
    ports:
      - '8086:8086'
    volumes:
  	  - influxdb-storage:/var/lib/influxdb
    environment:
  	  - INFLUXDB_DB=db0
  	  - INFLUXDB_ADMIN_USER=${INFLUXDB_USERNAME}
  	  - INFLUXDB_ADMIN_PASSWORD=${INFLUXDB_PW}

  grafana:
    image: grafana/grafana:8.3.4-ubuntu
    ports:
      - '3000:3000'
    volumes:
  	  - grafana-storage:/var/lib/grafana
    environment:
  	  - GF_SECURITY_ADMIN_USER=${GRAFANA_USERNAME}
  	  - GF_SECURITY_ADMIN_PASSWORD=${GRAFANA_PW}

volumes:
  influxdb-storage: {}
  grafana-storage: {}
```

The above multi-container application can be started with the following commands:

```bash
export INFLUXDB_USERNAME=admin
export INFLUXDB_PW=admin
export GRAFANA_USERNAME=admin
export GRAFANA_PW=admin
docker compose up 
```

Or write a further file containing all environment variables and this file must be named `.env`. 

```bash
user@vm:~/example$ cat .env
INFLUXDB_USERNAME=admin
INFLUXDB_PW=admin
GRAFANA_USERNAME=admin
GRAFANA_PW=admin
```

Docker checks if in the directory is present a `.env` file. In a such a case, it will import the environment variables when the multi-container application is started.

A third approach is to define an environment variable file per service and then insert the file name inside the description of the service in the Docker Compose file.

Following the files' content:

```bash
user@vm:~/example$ cat influxdb.env
INFLUXDB_DB=db0
INFLUXDB_USERNAME=admin
INFLUXDB_PW=admin

user@vm:~/example$ cat grafana.env
GRAFANA_USERNAME=admin
GRAFANA_PW=admin
```

And how the YAML file changes:

```yaml
version: '3.8'
services:
  influxdb:
    image: influxdb:2.1
    ports:
      - '8086:8086'
    volumes:
  	  - influxdb-storage:/var/lib/influxdb
    env_file:
      - influxdb.env

  grafana:
    image: grafana/grafana:8.3.4-ubuntu
    ports:
      - '3000:3000'
    volumes:
  	  - grafana-storage:/var/lib/grafana
    env_file:
      - grafana.env

volumes:
  influxdb-storage: {}
  grafana-storage: {}
```

Further comments on Environment variables in Docker compose. 

Environment variables can be used to customize your application, e.g. to define the image version of an service:

```yaml
service:
  influxdb:
    image: influxdb:{$INFLUX_VERSION}
```

Default values are allowed:

```yaml
service:
  influxdb:
    image: influxdb:{$INFLUX_VERSION:-2.1}
```

The default value should be inserted after the `:-` characters.

Since more approaches are supported, Docker compose uses the following priority order, overwriting the less important with the higher ones:
1. Compose ﬁle (highest important)
2. Shell environment variables
3. Environment ﬁle
4. Dockerﬁle
5. Variable not deﬁned (lowest important)

#### Health checks
As in the command line instructor, also in the Docker compose file can be defined a health check for a given service, as showed following:

```bash
docker volume create influxdb-storage
export INFLUXDB_USERNAME=admin
export INFLUXDB_PW=admin
docker container run --name influxdb 
           -p 8086:8086 \ 
           -v influxdb-storage:/var/lib/influxdb \
           -e INFLUXDB_DB=db0 \
           -e INFLUXDB_ADMIN_USER=${INFLUXDB_USERNAME} \
           -e INFLUXDB_ADMIN_PASSWORD=${INFLUXDB_PW} \
           --health-cmd='curl -f http://localhost:8086||exit 1' \
           --health-start-period=30s \
           --health-interval= 30s \
           --health-timeout=10s \
           --health-retries=4 \
	       influxdb:2.1
```

And the corresponding Docker compose file:

```yaml
version: '3.8'
services:
  influxdb:
    image: influxdb:2.1
    ports:
      - '8086:8086'
    volumes:
  	- influxdb-storage:/var/lib/influxdb
    environment:
  	- INFLUXDB_DB=db0
  	- INFLUXDB_ADMIN_USER=${INFLUXDB_USERNAME}
  	- INFLUXDB_ADMIN_PASSWORD=${INFLUXDB_PASSWORD}
    healthcheck:
      test: "curl --fail http://localhost:8086 || exit 1"
      start_period: 30s
      interval: 30s
      timeout: 10s
      retries: 4
```

### Dependencies

Some applications need a dependency chain between services, so that some services get loaded before (and unloaded after) other ones.

The dependencies chain can be defined inside the Docker Compose file by using the `depends_on` keyword. 

Considering the above example, we would like to start the grafana service after the influxdb service. 

The Docker Compose file becomes:

```yaml
version: '3.8'
services:
  influxdb:
    image: influxdb:2.1
    ports:
      - '8086:8086'
    volumes:
  	- influxdb-storage:/var/lib/influxdb
    environment:
  	- INFLUXDB_DB=db0
  	- INFLUXDB_ADMIN_USER=${INFLUXDB_USERNAME}
  	- INFLUXDB_ADMIN_PASSWORD=${INFLUXDB_PASSWORD}
  
  grafana:
    image: grafana/grafana:8.3.4-ubuntu
    ports:
      - '3000:3000'
    volumes:
  	- grafana-storage:/var/lib/grafana
    environment:
  	- GF_SECURITY_ADMIN_USER=${GRAFANA_USERNAME}
  	- GF_SECURITY_ADMIN_PASSWORD=${GRAFANA_PASSWORD}
    depends_on:
      - influxdb

volumes:
  influxdb-storage:
  grafana-storage:
```

#### Waiting for a service is ready

Some applications might need to wait for a tool is “ready”, not just running. Health checks help to identify the "ready" status. 

The Docker compose file becomes:

```yaml
version: '3.8'
services:
  influxdb:
    image: influxdb:2.1
    ports:
      - '8086:8086'
    volumes:
  	- influxdb-storage:/var/lib/influxdb
    environment:
  	- INFLUXDB_DB=db0
  	- INFLUXDB_ADMIN_USER=${INFLUXDB_USERNAME}
  	- INFLUXDB_ADMIN_PASSWORD=${INFLUXDB_PASSWORD}
    healthcheck:
      test: "curl --fail http://localhost:8086 || exit 1"
      start_period: 30s
      interval: 30s
      timeout: 10s
      retries: 4
 
  grafana:
    image: grafana/grafana:8.3.4-ubuntu
    ... 
    depends_on:
      influxdb:
        condition: service_healthy
```

### Profiles

Profiles allow adjusting the Compose application model for various usages and environments by selectively enabling services. 

This is achieved by assigning each service to zero or more profiles. 

If unassigned, the service is always started but if assigned, it is only started if the profile is activated.

Let's considering the following Docker compose file

```yaml
version: "3.8"
services:
  frontend:
    image: frontend
    profiles: 
      - frontend

  phpmyadmin:
    image: phpmyadmin
    depends_on:
      - db
    profiles:
      - debug

  backend:
    image: backend

  db:
    image: mysql
```

!!! tip
    - the `db` service is always started since it does not contain the `profile` keyword
    - the `phpmyadmin` service is started only when the `debug` profile is enabled
    - the `frontend` service is started only when the `frontend` profile is enabled

Examples:

```bash
docker compose up                                     # Only the db service is started
docker compose --profile frontend up                  # Only the db and frontend services are started
COMPOSE_PROFILES=frontend docker compose up           # Other syntax for the above command

docker compose --profile frontend --profile debug up  # All services are started
COMPOSE_PROFILES=frontend,debug docker compose up     # Other syntax for the above command
```

### Build Dockerfile and run a multi-container application

Images are defined in the `image` section on each service. Docker Compose allows to use for a service a custom image (described with a Dockerfile) that is built in run-time. 

For those services the `image` keyword should be replaced with the `build` keyword. 

Following an example of an application composed of a single service whom image is build in run-time. The custom image consist of a `python:3` official image with additional python modules installed.

Folder content:

```bash
user@vm:~/prova_4$ ls -l
Dockerfile
requirements
docker-compose.yml
my-script.py
```

Dockerfile file content:

```dockerfile
FROM python:3

COPY requirements /requirements
RUN pip install -r requirements
COPY my-script.py /my-script.py
 
CMD [ "python3", "/my-script.py"]
```

Requirements file content:

```bash
pandas
numpy
```

my-script file content:

```python
import socket
import sys
from time import sleep
sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server_address = ('localhost', 1234)
print(f'starting up on {server_address[0]} port {server_address[1]}')
sock.bind(server_address)
sleep(60)
```

Docker compose file content:

```yaml
version: '3.8'
services:
  my_service:
    build: .
    ports:
      - '1234:1234'
    volumes:
      - dir_data:/data
```

Let's start the application:
=== "Command"
    ```bash
    docker compose up -d -–build
    ```
=== "Output"
    ```bash
    Creating network "prova_4_default" with the default driver
    Building my_service
    Sending build context to Docker daemon   5.12kB
    Step 1/4 : FROM python:3
    ---> de529ffbdb66
    Step 2/4 : COPY requirements ./
    ---> 8f6840fe4dab
    Step 3/4 : RUN pip install --no-cache-dir -r requirements.txt
    ---> Running in 363ffa40779c
    Collecting pandas
    Downloading pandas-1.4.0-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (11.7 MB)
    Collecting numpy
    Downloading numpy-1.22.2-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (16.8 MB)
    Collecting python-dateutil>=2.8.1
    Downloading python_dateutil-2.8.2-py2.py3-none-any.whl (247 kB)
    Collecting pytz>=2020.1
    Downloading pytz-2021.3-py2.py3-none-any.whl (503 kB)
    Collecting six>=1.5
    Downloading six-1.16.0-py2.py3-none-any.whl (11 kB)
    Installing collected packages: six, pytz, python-dateutil, numpy, pandas
    Successfully installed numpy-1.22.2 pandas-1.4.0 python-dateutil-2.8.2 pytz-2021.3 six-1.16.0
    Removing intermediate container 363ffa40779c
    ---> 79f0ed627dd6
    Step 4/4 : CMD [ "python3", "/my-script.py" ]
    ---> Running in 4f45fc08e6df
    Removing intermediate container 4f45fc08e6df
    ---> 4fd169b43c5f
    Successfully built 4fd169b43c5f
    Successfully tagged prova_4_my_service:latest
    Creating prova_4_my_service_1 ... done
    ```
