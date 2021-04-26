![image](https://user-images.githubusercontent.com/44756128/116119583-36c97b80-a684-11eb-88a2-374d1d7c7188.png)

# The Scenario
After years of running containers on a single Docker host, we've decided to migrate over to using Docker Swarm. Using Swarm will allow our clients to scale the number of containers up, as demand increases, and then down as demand dies off.

Before we can do this, we first need to set up a swarm cluster consisting of a manager and a worker node. Once setup is complete, create an Nginx service to test the cluster.

# Initialize the Swarm
On the manager node, initialize the swarm:
```sh
docker swarm init \
--advertise-addr [MANAGER_PRIVATE_IP]
```

After this runs, we'll get a docker swarm join command that we can then go run from the worker. Copy it.

# Add the Worker to the Cluster
Now we can paste the docker swarm join command in a terminal on the worker:
```sh
docker swarm join --token [TOKEN] \
[MANAGER_PRIVATE_IP]:2377
```

On the manager, we can run docker node ls and see both of them, if all went well.

# Create a Swarm Service
Let's create an Nginx service in the swarm. Run this command on the master node:
```sh
docker service create -d \
--name nginx_service \
-p 8080:80 \
--replicas 2 \
nginx:latest
```

Now we can run (still on the master node) docker service ls and see nginx_service running, and that it's running on both nodes.

# Conclusion
With this little Docker swarm set up, we're on our way to the migration. 
