![image](https://user-images.githubusercontent.com/44756128/115482877-47f23280-a215-11eb-831c-ed96d4f69c9f.png)

# Introduction
For the last six months, the Acme Anvil Corporation has been migrating some of their bare metal infrastructure to Docker containers. A schism has developed between the members of your team on whether to use Docker Swarm or Kubernetes. To settle the dispute, your manager has decided to create a series of challenges. You have been tasked with creating a demo on how to back up and restore a Docker swarm. You are to set up a Docker swarm with 3 nodes, scale the backup service up to 3 nodes, and back up your master node and restore it to a backup instance.

#Instructions
We are tasked with backing up and restoring a Docker swarm.
  - Use cat on the file swarm-token.txt to show the join token for your worker nodes.
  - Add 2 worker nodes to the swarm.
  - Scale the backup service up to 3 replicas.
  - Stop the Docker service on the master node and use tar to compress the swarm directory.
  - Use scp to securely copy the swarm tarball to /home/cloud_user on the backup master node.
  - Use tar to extract the tarball.
  - Stop the Docker service, restore the swarm directory, and start up the Docker service.
  - Force the swarm to reinitialize a new cluster.
  - Add the two worker nodes to the swarm.

# Join the worker nodes to the swarm
On server1, view the docker swarm join command:
```sh
cat swarm-token.txt
```

![image](https://user-images.githubusercontent.com/44756128/115483538-948a3d80-a216-11eb-8bd1-9fe70cfeb452.png)

Copy the docker swarm join command provided in this output, and run it on server2:
```sh
docker swarm join --token $JOIN_TOKEN_HERE
```

![image](https://user-images.githubusercontent.com/44756128/115483716-ee8b0300-a216-11eb-983d-26897299fc05.png)

Now we'll run it again on server3:
```sh
docker swarm join --token $JOIN_TOKEN_HERE
```

![image](https://user-images.githubusercontent.com/44756128/115483731-f480e400-a216-11eb-9b14-4e55fcdab0e3.png)

Back on the master node, let's view the running services:
```sh
docker service ls
```

We'll see a service named backup, and we want to scale it up to 3 nodes:
```sh
docker service scale backup=3
```

![image](https://user-images.githubusercontent.com/44756128/115483833-28f4a000-a217-11eb-8ffb-4ac41fa2ad90.png)

Now we can check by showing the service across our nodes:
```sh
docker service ps backup
```

![image](https://user-images.githubusercontent.com/44756128/115483882-4295e780-a217-11eb-9d36-b0ff86127652.png)

# Back up the swarm master
Before we can back up the Docker swarm, we've got to stop the docker service on the master node:
```sh
systemctl stop docker
```

Now back up the /var/lib/docker/swarm/ directory:
```sh
tar czvf swarm.tgz /var/lib/docker/swarm/
```

![image](https://user-images.githubusercontent.com/44756128/115483933-5f321f80-a217-11eb-9055-e06b7a4cdbaf.png)

# Restore the swarm on the backup master
Let's copy the swarm backup from the master node to the backup master:

Note: Be sure to replace BACKUP_IP_ADDRESS with the private IP address of the backup server.
```sh
scp swarm.tgz cloud_user@BACKUP_IP_ADDRESS:/home/cloud_user/
```

![image](https://user-images.githubusercontent.com/44756128/115484008-8f79be00-a217-11eb-99a7-ae84d3dc5153.png)

Now we can get into the backup server and extract the backup file:
```sh
cd /home/cloud_user
tar xzvf swarm.tgz
```

Then copy the swarm directory to /var/lib/docker/swarm. First let's get into the directory:
```sh
cd var/lib/docker
```

Now do the actual copying:
```sh
[root@backup ~]# cp -rf swarm/ /var/lib/docker/
```

With that finished, we can restart the Docker service:
```sh
systemctl restart docker
```

Now we'll reinitialize the swarm:
```sh
docker swarm init --force-new-cluster
```

![image](https://user-images.githubusercontent.com/44756128/115484236-1038ba00-a218-11eb-86a3-7e53b16daaae.png)

We need the docker swarm join command that's in the output.

# Add the worker nodes to the restored cluster
On server2 and server3, we need to leave the first swarm and join the new one. First, server2:
```sh
docker swarm leave
docker swarm join --token $NEW_JOIN_TOKEN_HERE
```

![image](https://user-images.githubusercontent.com/44756128/115484375-568e1900-a218-11eb-838e-2ee859dd2d76.png)

Oops, that didn't work. Look at that new join token. The IP address is for server1. We need the IP for backup in there. Edit the command (swapping out the bad IP address for the good one) and try to join the swarm again.

Now let's repeat on server3:
```sh
docker swarm leave
docker swarm join --token $NEW_JOIN_TOKEN_HERE
```

![image](https://user-images.githubusercontent.com/44756128/115484394-5d1c9080-a218-11eb-8ca2-88539282f9b0.png)

Remember to change that IP again.

# Distribute the replicas across the swarm
On the backup server, make sure the backup service is still running:
```sh
docker service ls
```

Now let's look at the current service list:
```sh
docker service ps backup
```

This shows us that we have three replicas, but they're all on our backup master (backup). To make sure our service is distributed across our swarm, we've first got to scale things down to one, then back up to three:
```sh
docker service scale backup=1
docker service scale backup=3
```

Once that's done, let's check on our service again:
```sh
[root@backup ~]# docker service ps backup
```

![image](https://user-images.githubusercontent.com/44756128/115484617-b684bf80-a218-11eb-834d-aa8536cc4862.png)

We'll see that our service is now running across all of the nodes.
