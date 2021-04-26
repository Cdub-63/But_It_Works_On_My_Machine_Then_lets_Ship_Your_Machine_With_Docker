![image](https://user-images.githubusercontent.com/44756128/116120779-89576780-a685-11eb-86c6-1e90f03d6677.png)

# Introduction
After a few months of debate, we’ve decided to set up a cooking blog. After researching different platforms, we've chosen Wordpress with MySQL. We have a swarm cluster already set up where we host customer sites. To make maintaining Wordpress easier, we’ve decided to set it up as a stack. We'll need to create the Docker Compose file, deploy the stack, and finish the Wordpress setup.

# Complete the Swarm Setup
We'll start off by getting a join token. On the master node, run this:
```sh
[cloud_user@manager]$ docker swarm join-token worker
```

Copy the join token, and then run it on the worker node:
```sh
[cloud_user@worker]$ docker swarm join --token [TOKEN] [MANAGER_PRIVATE_IP]:2377
```

Now, to see if things are running, execute this on the manager:
```sh
[cloud_user@manager]$ docker node ls
```

We should see two nodes, each with a STATUS of Ready and an Availability of Active.

# Create the Compose File
Next up, we'll create the compose file on the master node. Use whatever text editor you like, but the file needs to be named docker-compose.yml, and these are the contents:
```yml
version: '3.1'

services:
   db:
     image: mysql:5.7
     volumes:
       - db_data:/var/lib/mysql
     networks:
       mysql_internal:
     environment:
       MYSQL_ROOT_PASSWORD: P4ssw0rd0!
       MYSQL_DATABASE: wordpress
       MYSQL_USER: wordpress
       MYSQL_PASSWORD: P4ssw0rd0!

   blog:
     depends_on:
       - db
     image: wordpress
     networks:
       mysql_internal:
       wordpress_public:
     ports:
       - "80:80"
     environment:
       WORDPRESS_DB_HOST: db:3306
       WORDPRESS_DB_USER: wordpress
       WORDPRESS_DB_PASSWORD: P4ssw0rd0!

volumes:
    db_data:
networks:
  mysql_internal:
    internal: true
  wordpress_public:
```

# Create the Wordpress Blog
Deploy the stack with:
```sh
docker stack deploy --compose-file docker-compose.yml wp
```

To make sure it was created, run:
```sh
docker stack ls
```

We should see out wp stack. To look further, use this command:
```sh
docker service ls
```

This will show us the running services, and we can see that each has 1/1 replicas. This means everything is working right.

# Complete the Wordpress Setup
NOTE: Please be patient while the service comes up for its first run, this can take some time due to the size of the wordpress application, once loaded, also allow some time for first login to complete successfully, delays are normal.

Now that we know everything is running, we can browse (in a web browser) to the public IP of our master node and land on the Wordpress setup page. Use whatever language you're comfortable with, fill out the form, and let Wordpress do the rest. We'll land at the Wordpress login page once the install finishes.

# Conclusion
We're done. We've got a brand-spanking new blog just waiting for cooking content, and it's sitting on the the Docker stack, just like we wanted. Congratulations!
