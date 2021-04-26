![image](https://user-images.githubusercontent.com/44756128/116121887-b1939600-a686-11eb-863f-6e2d69d17d5e.png)

# The Scenario
In order to secure a MySQL database, we’ve decided to redeploy the container it sits in as a Swarm service, using secrets.

We'll use OpenSSL to generate secure passwords for both the MySQL users root and user. Then we'll save them to separate files. Next we'll create secrets for these passwords, and finally create the MySQL service using these secrets.

# Complete the Swarm Setup
We'll start off by getting a join token. On the master node, run this:
```sh
docker swarm join-token worker
```

![image](https://user-images.githubusercontent.com/44756128/116152928-2b8a4600-a6ac-11eb-92b2-631e6cf3169d.png)

Copy the join token, and then run it on the worker node:
```sh
docker swarm join --token [TOKEN] [MANAGER_PRIVATE_IP]:2377
```

![image](https://user-images.githubusercontent.com/44756128/116153058-5eccd500-a6ac-11eb-8525-b56442a81777.png)

We should get a message about this node joining a swarm as a worker. We're good to go, and we can shut the worker terminal down.

# Create Secrets
Back in the manager node, we need to create the MySQL root password:
```sh
openssl rand -base64 20 > mysql_root_password.txt
docker secret create mysql_root_password mysql_root_password.txt
```

Create a MySQL user password:
```sh
openssl rand -base64 20 > mysql_password.txt
docker secret create mysql_password mysql_password.txt
```

# Create an Overlay Network for the Service
```
docker network create -d overlay mysql_private
```

# Create the MySQL Service
```sh
docker service create \
--name mysql_secrets \
--replicas 1 \
--network mysql_private \
--mount type=volume,destination=/var/lib/mysql \
--secret mysql_root_password \
--secret mysql_password \
-e MYSQL_ROOT_PASSWORD_FILE="/run/secrets/mysql_root_password" \
-e MYSQL_PASSWORD_FILE="/run/secrets/mysql_password" \
-e MYSQL_USER="myUser" \
-e MYSQL_DATABASE="myDB" \
mysql:5.7
```

# Conclusion
If we list our services with a docker service ls, we'll see that everything is up and running, with the right number of replicas.

![image](https://user-images.githubusercontent.com/44756128/116153309-ba975e00-a6ac-11eb-9142-0fb406814dd5.png)
