![image](https://user-images.githubusercontent.com/44756128/115881739-ab48b400-a411-11eb-9a17-71fc6adaede5.png)

# The Scenario
We need to deploy a MySQL container to our development environment. Because we will be working with mock customer data that needs to be persistent, the container will need a volume. Create a volume called mysql_data. Then deploy a MySQL container that will use this volume to store database files.

# Create a Volume Called mysql_data
First we'll use the docker volume command to create a volume called mysql_data:
```sh
docker volume create mysql_data
```

# Create a MySQL Container
Then we'll run the docker container command to create a MySQL container:
```sh
docker container run -d --name app-database \
--mount type=volume,source=mysql_data,target=/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=P4ssW0rd0! \
mysql:latest
```

Create a volume called mysql_data, then deploy a MySQL container called app-database. Use the mysql latest image, and use the -e flag to set MYSQL_ROOT_PASSWORD to P4sSw0rd0. Use the mount flag to mount the mysql_data volume to /var/lib/mysql. The container should run in the background.

We can confirm that everything worked with:
```sh
docker container inspect app-database
```

![image](https://user-images.githubusercontent.com/44756128/115882373-4fcaf600-a412-11eb-823d-41bd62985402.png)
