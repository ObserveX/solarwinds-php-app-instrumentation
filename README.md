# solarwinds-php-app-instrumentation

### Application used: wordpress
Reference: https://hub.docker.com/_/wordpress

Approach used for APM Instrumentation:   
[A] Manual Setup(Not recommended in Prod)  
[B] Using Dockerfile to include SolarWinds Instrumentation steps(Recommended method)

## [A] Manual Setup(Not recommended in Prod)

1) Create a docker-compose.yml file
```
services:
  db:
    # We use a mariadb image which supports both amd64 & arm64 architecture
    image: mariadb:10.6.4-focal
    # If you really want to use MySQL, uncomment the following line
    #image: mysql:8.0.27
    command: '--default-authentication-plugin=mysql_native_password'
    volumes:
      - db_data:/var/lib/mysql
    restart: always
    environment:
      - MYSQL_ROOT_PASSWORD=somewordpress
      - MYSQL_DATABASE=wordpress
      - MYSQL_USER=wordpress
      - MYSQL_PASSWORD=wordpress
    expose:
      - 3306
      - 33060
  wordpress:
    image: wordpress:latest
    volumes:
      - wp_data:/var/www/html
    ports:
      - 80:80
    restart: always
    environment:
      - WORDPRESS_DB_HOST=db
      - WORDPRESS_DB_USER=wordpress
      - WORDPRESS_DB_PASSWORD=wordpress
      - WORDPRESS_DB_NAME=wordpress
volumes:
  db_data:
  wp_data:
```
2) Run docker compose
```
docker-compose up -d
```
3) Execute a command inside a running container
```
docker exec -it <Container ID or Name> bash
```
4) Download the wrapper script
```
curl -sSO https://agent-binaries.cloud.solarwinds.com/apm/php/latest/solarwinds-apm-php.sh
```
5) Run the wrapper script
```
sh solarwinds-apm-php.sh --service-key=YourServiceKey
```
6) Exit and Restart the container. Generate traffic to see APM data in SolarWinds.

## [B] Using docker-compose to include SolarWinds Instrumentation steps(Recommended method)

1) Create a docker-compose.yml file
```
services:
  db:
    # We use a mariadb image which supports both amd64 & arm64 architecture
    image: mariadb:10.6.4-focal
    # If you really want to use MySQL, uncomment the following line
    #image: mysql:8.0.27
    command: '--default-authentication-plugin=mysql_native_password'
    volumes:
      - db_data:/var/lib/mysql
    restart: always
    environment:
      - MYSQL_ROOT_PASSWORD=somewordpress
      - MYSQL_DATABASE=wordpress
      - MYSQL_USER=wordpress
      - MYSQL_PASSWORD=wordpress
    expose:
      - 3306
      - 33060
  wordpress:
    image: wordpress:latest
    volumes:
      - wp_data:/var/www/html
    ports:
      - 80:80
    restart: always
    environment:
      - WORDPRESS_DB_HOST=db
      - WORDPRESS_DB_USER=wordpress
      - WORDPRESS_DB_PASSWORD=wordpress
      - WORDPRESS_DB_NAME=wordpress
    command: >
      sh -c 'curl -sSO https://agent-binaries.cloud.solarwinds.com/apm/php/latest/solarwinds-apm-php.sh &&
             sh solarwinds-apm-php.sh --service-key=<INSERT SERVICE KEY HERE> --collector=apm.collector.cloud.solarwinds.com &&
             apache2-foreground'
volumes:
  db_data:
  wp_data:

```
2) Run docker compose
```
docker-compose up -d
```
Generate traffic to see APM data in SolarWinds. 

Note: It is not recommended to keep sensitive information such as service keys, API keys, or passwords in a Dockerfile or any source code file. Instead, you can use environment variables or Docker secrets to securely pass sensitive information to your containers.

To use environment variables, you can either set them in your docker-compose.yml file or pass them when running the docker run command.

## Real User Monitoring via Manual Injection
To receive RUM data from your website, add the script immediately before the </body> element on below wordpress web pages.
```bash
/var/www/html/wp-content/themes/
```

Generate load on application and verify RUM data in SolarWinds.
