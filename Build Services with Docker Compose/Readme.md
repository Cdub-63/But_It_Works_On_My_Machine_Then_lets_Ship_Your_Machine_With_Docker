![image](https://user-images.githubusercontent.com/44756128/115154071-9e594880-a03e-11eb-8e2c-8840bd2a63f8.png)

# Introduction
For the last six months, the Acme Anvil Corporation has been migrating some of their bare metal infrastructure to Docker containers. Your team wants to find an easier way to deploy applications that consist of multiple containers and has decided to use Docker Compose. You have been tasked with setting up an internal blog so the team can write technical articles. This blog will consist of two services: a Ghost Blog service and a MySQL service. Both services will use volumes for persistent storage.

Solution
Begin by logging in to the server using ssh:
```sh
ssh username@PUBLIC_IP_ADDRESS
```

Become the root user:
```
sudo su -
```

![image](https://user-images.githubusercontent.com/44756128/115154112-ccd72380-a03e-11eb-838a-ea4b5bf1cd89.png)

# Create a Ghost Blog and MySQL Service
Create a docker-compose.yml file in the root directory.
```sh
vi docker-compose.yml
```

Add the following contents to it:
```yml
version: '3'
services:
  ghost:
    image: ghost:1-alpine
    container_name: ghost-blog
    restart: always
    ports:
      - 80:2368
    environment:
      database__client: mysql
      database__connection__host: mysql
      database__connection__user: root
      database__connection__password: P4sSw0rd0!
      database__connection__database: ghost
    volumes:
      - ghost-volume:/var/lib/ghost
    depends_on:
      - mysql

  mysql:
    image: mysql:5.7
    container_name: ghost-db
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: P4sSw0rd0!
    volumes:
      - mysql-volume:/var/lib/mysql

volumes:  
  ghost-volume:  
  mysql-volume:  
```

![image](https://user-images.githubusercontent.com/44756128/115154377-18d69800-a040-11eb-9b45-6ad057ec0410.png)

# Bring Up the Ghost Blog Service
Start up the Docker Compose service.
```sh
docker-compose up -d
```

![image](https://user-images.githubusercontent.com/44756128/115154433-7965d500-a040-11eb-99f2-b9897ff663ff.png)
