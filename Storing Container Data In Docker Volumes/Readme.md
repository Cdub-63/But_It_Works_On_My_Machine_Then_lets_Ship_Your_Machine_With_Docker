![image](https://user-images.githubusercontent.com/44756128/113886787-91a73b80-9786-11eb-9d21-11ad903f0505.png)

# Introduction
Storing data within a container image is one option for automating a container with data, but it requires a copy of the data to be in each container you run.

For static files, this can be a waste of resources. Each file might not amount to more than a few megabytes, but once the containers are scaled up to handle a production load, that few megabytes can turn into gigabytes of waste.

Instead, you can store one copy of the static files in a Docker volume for easy sharing between containers.

We will learn how Docker volumes interact with containers. We will do this by creating new volumes and attaching them to containers. We'll then clean up space left by anonymous volumes created automatically by the containers. Finally, We'll learn about backup strategies for our volumes.

Log in to the server using the credentials provided:
```sh
ssh username@<PUBLIC_IP_ADDRESS>
```

# Discover Anonymous Docker Volumes
Check the docker images:
```sh
docker images
```

Run a container using the postgres:12.1 repository:
```sh
docker run -d --name db1 postgres:12.1
```

Run a second container using the same image:
```sh
docker run -d --name db2 postgres:12.1
```

Check the status of the containers:
```sh
docker ps
```

List the anonymous volumes:
```sh
docker volume ls
```
Inspect the db1 container:
```sh
docker inspect db1 -f '{{ json .Mounts }}' | python -m json.tool
```

Inspect the db2 container:
```sh
docker inspect db2 -f '{{ json .Mounts}}' | python -m json.tool
```

Create a third container using the --rm flag:
```sh
docker run -d --rm --name dbTmp postgres:12.1
```

Check the status of the container:
```sh
docker ps -a
```

List the anonymous volumes:
```sh
docker volume ls
```

Stop the db2 and dbTmp containers:
```sh
docker stop db2 dbTmp
```

List the anonymous volumes:
```sh
docker volume ls
```

Check the status of the containers:
```sh
docker ps -a
```

![image](https://user-images.githubusercontent.com/44756128/113890049-712cb080-9789-11eb-81f3-c3e12d908f97.png)

# Create a Docker Volume
Create a Docker volume:
```sh
docker volume create website
```

Verify that the volume was created successfully:
```sh
docker volume ls
```

Copy the widget-factory-inc data to the website container:
```sh
sudo cp -r /home/cloud_user/widget-factory-inc/web/* /var/lib/docker/volumes/website/_data/
```

List the copied data:
```sh
sudo ls -l /var/lib/docker/volumes/website/_data/
```

![image](https://user-images.githubusercontent.com/44756128/113890693-02038c00-978a-11eb-84a0-b80ffc73a2a2.png)

# Use the Website Volume with Containers
Run a docker container with the website volume:
```sh
docker run -d --name web1 -p 80:80 -v website:/usr/local/apache2/htdocs:ro httpd:2.4
```

Check the status of the container:
```sh
docker ps
```

![image](https://user-images.githubusercontent.com/44756128/113891020-4e4ecc00-978a-11eb-820a-b090a40c4067.png)

In a web browser, verify connectivity to the container:
```sh
<PUBLIC_IP_ADDRESS>
```

![image](https://user-images.githubusercontent.com/44756128/113891193-750d0280-978a-11eb-81ce-4b562dab5423.png)

Run a second container with the --rm flag:
```sh
docker run -d --name webTmp --rm -v website:/usr/local/apache2/htdocs:ro httpd:2.4
```

Check the status of the containers:
```sh
docker ps
```

Stop the webTmp container:
```sh
docker stop webTmp
```

Check the status of the containers:
```sh
docker ps -a
```

![image](https://user-images.githubusercontent.com/44756128/113892974-2f513980-978c-11eb-8295-073065742a3f.png)

Verify that the website can still be accessed through a web browser:
```sh
<PUBLIC_IP_ADDRESS>
```

![image](https://user-images.githubusercontent.com/44756128/113893140-59a2f700-978c-11eb-9457-276e15efa919.png)

# Clean up Unused Volumes
Clean up the unused volumes:
```sh
docker volume prune
```

Check the currently running containers:
```sh
docker ps -a
```

Remove the db2 container:
```sh
docker rm db2
```

Clean up the unused volumes again:
```sh
docker volume prune
```

List the current volumes:
```sh
docker volume ls
```

![image](https://user-images.githubusercontent.com/44756128/113893614-d03ff480-978c-11eb-9f8a-7a17342f0174.png)

# Back up and Restore the Docker Volume
Switch to the root user:
```sh
sudo -i
```

Find where the website volume data is stored:
```sh
docker volume inspect website
```

Copy the Mountpoint

Back up the volume:
```sh
tar czf /tmp/website_$(date +%Y-%m-%d-%H%M).tgz -C /var/lib/docker/volumes/website/_data .
```

![image](https://user-images.githubusercontent.com/44756128/113896238-5eb57580-978f-11eb-9e32-598e2e4c127f.png)

Verify that the data was backed up properly:
```sh
ls -l /tmp/website_*.tgz
```

List the contents of the tgz file:
```sh
tar tf /tmp/<BACKUP_FILE_NAME>.tgz
```

Exit out of root:
```sh
exit
```

Run a new container using the website volume, and create a backup:
```sh
docker run -it --rm -v website:/website -v /tmp:/backup bash tar czf /backup/website_$(date +%Y-%m-%d-%H-%M).tgz -C /website .
```

Verify that the data was backed up properly:
```sh
ls -l /tmp/website_*.tgz
```

Switch to the root user:
```sh
sudo -i
```

Change to the /docker/volumes/ directory:
```sh
cd /var/lib/docker/volumes/
```

List the volumes:
```sh
ls -l
```

Change to the website/_data directory:
```sh
cd website/_data/
```

Remove the contents of the directory:
```sh
rm * -rf
```

Verify that the backups are still available:
```sh
ls -l /tmp/website_*.tgz
```

![image](https://user-images.githubusercontent.com/44756128/113896366-7bea4400-978f-11eb-8f6f-d4e481ee2abd.png)

Restore the data to the current directory:
```sh
tar xf <BACKUP_FILE_NAME>.tgz .
```

Verify that the data was restored successfully:
```sh
ls -l
```

![image](https://user-images.githubusercontent.com/44756128/113896424-886e9c80-978f-11eb-9730-c55f85f1f418.png)
