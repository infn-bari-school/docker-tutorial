
### Hands-on 1

Write the compose.yaml file for the following Docker CLI inserting the `depends_on` condition on db health check

=== "Exercise details"
    ```bash
    docker network create wordpress_net
    docker volume create db_data
    docker volume create wp_data
    docker container run --name db \
      --network wordpress_net \
      -v db_data:/var/lib/mysql \
      -e MYSQL_ROOT_PASSWORD=somewordpress \
      -e MYSQL_USER=wordpress_user \
      -e MYSQL_PASSWORD=wordpress_password \
      -e MYSQL_DATABASE=wordpress_database \
      --restart always \
      --health-cmd="mysqladmin ping --silent" \
      --health-interval=10s \
      --health-start-period=10s \
      --health-timeout=10s \
      --health-retries=60 \
      --restart always \
      -d \
      mariadb:10.6.4-focal

    docker run --name wp \
      --network wordpress_net \
      -v wp_data:/var/www/html \
      -p 8080:80 \
      -e WORDPRESS_DB_HOST=db \
      -e WORDPRESS_DB_USER=wordpress_user \
      -e WORDPRESS_DB_PASSWORD=wordpress_password \
      -e WORDPRESS_DB_NAME=wordpress_database \
      -d \
      wordpress
    ```
=== "Solution"
    ```yaml
    services:
      db:
        image: mariadb:10.6.4-focal
        volumes:
          - db_data:/var/lib/mysql
        environment:
          - MYSQL_ROOT_PASSWORD=somewordpress
          - MYSQL_USER=wordpress_user
          - MYSQL_PASSWORD=wordpress_password
          - MYSQL_DATABASE=wordpress_database
        restart: always
        healthcheck:
          test: "mysqladmin ping --silent"
          interval: 10s
          start_period: 10s
          timeout: 10s
          retries: 60

      wp:
        image: wordpress:latest
        volumes:
          - wp_data:/var/www/html
        ports:
          - 8080:80
        environment:
          - WORDPRESS_DB_HOST=db
          - WORDPRESS_DB_USER=wordpress_user
          - WORDPRESS_DB_PASSWORD=wordpress_password
          - WORDPRESS_DB_NAME=wordpress_database
        restart: always
        depends_on:
          db:
            condition: service_healthy

    volumes:
      db_data:
      wp_data:
    ```

### Hands-on 2

Write the compose.yaml file that builds the following Dockerfile and uses it

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
    Jupyter notebook to test the configuration

    ```python
    import pandas as pd
    output_path = 'my_output.csv'

    data = {
        'apples': [3, 2, 0, 1], 
        'oranges': [0, 3, 7, 2]
    }
    df = pd.DataFrame(data)

    df.to_csv(output_path)
    ```

=== "Solution 1"
    ```bash
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
=== "Solution 2"
    ```bash
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
        command: "/opt/conda/bin/python3.11 \
                  /opt/conda/bin/jupyter-lab \
                    --no-browser \
                    --allow-root \
                    --NotebookApp.token='' \
                    --NotebookApp.password='' "
    EOF

    # To execute the application run:
    # docker compose up
    ```

=== "Solution 3"
    ```bash
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
        command: 
          - /opt/conda/bin/python3.11
          - /opt/conda/bin/jupyter-lab
          - --no-browser
          - --allow-root
          - --NotebookApp.token=''
          - --NotebookApp.password=''
    EOF

    # To execute the application run:
    # docker compose up
    ```