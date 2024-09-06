
### Hands-on 1

Write the docker-compose.yml file for the following Docker CLI inserting the `depends_on` condition on db health check

=== "Exercise details"
    ```bash
    docker network create wordpress_net
    docker volume create db_data
    docker volume create wp_data
    docker container run --name hands-on1-db \
      --network wordpress_net \
      -v db_data:/var/lib/mysql \
      -e MYSQL_ROOT_PASSWORD=somewordpress \
      -e MYSQL_USER=wordpress-user \
      -e MYSQL_PASSWORD=wordpress-password \
      -e MYSQL_DATABASE=wordpress-db \
      --restart always \
      --health-cmd="mysqladmin ping --silent" \
      --health-interval=10s \
      --health-start-period=10s \
      --health-timeout=10s \
      --health-retries=60 \
      --restart always \
      -d \
      mariadb:10.6.4-focal

    docker run --name hands-on1-wp \
      --network wordpress_net \
      -v wp_data:/var/www/html \
      -p 8080:80 \
      -e WORDPRESS_DB_HOST=db \
      -e WORDPRESS_DB_USER=wordpress-user \
      -e WORDPRESS_DB_PASSWORD=wordpress-password \
      -e WORDPRESS_DB_NAME=wordpress-database \
      -d \
      wordpress
    ```
=== "Solution"
    ```yaml
    services:
      db:
        image: mariadb:10.6.4-focal
        container_name: hands-on1-sol-db
        volumes:
          - db_data:/var/lib/mysql
        networks:
          - wordpress_net
        restart: always
        environment:
          - MYSQL_ROOT_PASSWORD=somewordpress
          - MYSQL_DATABASE=wordpress-database
          - MYSQL_USER=wordpress-user
          - MYSQL_PASSWORD=wordpress-password
        healthcheck:
          test: ["CMD", "mysqladmin", "ping", "--silent"]
          interval: 10s
          timeout: 10s
          retries: 60
          start_period: 10s
    
      wordpress:
        image: wordpress:latest
        container_name: hands-on1-sol-wordpress
        volumes:
          - wp_data:/var/www/html
        networks:
          - wordpress_net
        ports:
          - 8081:80
        restart: always
        environment:
          - WORDPRESS_DB_HOST=db
          - WORDPRESS_DB_USER=wordpress-user
          - WORDPRESS_DB_PASSWORD=wordpress-password
          - WORDPRESS_DB_NAME=wordpress-database
        depends_on:
          db:
            condition: service_healthy

    volumes:
      db_data:
      wp_data:
    
    networks:
      wordpress_net:
    ```

### Hands-on 2

Write the docker-compose.yml file that builds the following Dockerfile and uses it

=== "Exercise details"
    ```
    base-image: jupyter/minimal-notebook
    python module to install: pandas, numpy
    expose port: 8888
    command: /opt/conda/bin/python3.11 /opt/conda/bin/jupyter-lab \
                --no-browser \
                --allow-root \
                --NotebookApp.token='' \
                --NotebookApp.password=''
    ```
=== "Solution"
    ```bash
    mkdir exercise2
    cd exercise2
    cat > requirements.txt << EOF
    pandas
    numpy
    EOF

    cat > Dockerfile << EOF
    FROM jupyter/minimal-notebook
    COPY requirements.txt /requirements.txt
    RUN pip install -r /requirements.txt
    EOF

    cat > compose.yaml << EOF
    services:
      my_jupyter:
        build: .
        ports:
          - 8888:8888
        command: /opt/conda/bin/python3.11 /opt/conda/bin/jupyter-lab --no-browser --allow-root --NotebookApp.token='' --NotebookApp.password=''
    EOF

    # To execute the application run:
    # docker compose up
    ```