![image](https://user-images.githubusercontent.com/44756128/115287572-0fb7fa80-a116-11eb-848c-315b1feb15ea.png)

# Introduction
For the last six months, the Acme Anvil Corporation has been migrating some of their bare metal infrastructure to Docker containers. A schism has developed between the members of your team on whether to use Docker Swarm or Kubernetes. Your manager has decided to settle the dispute by creating two competing demos for Docker Swarm and Kubernetes. You have been tasked with helping to create the Docker Swarm demo. Create a swarm with three nodes and then create a service using the weather-app image.

You are tasked with setting up a Docker swarm.
  - Log in to Swarm Server 1. This will be your swarm master.
  - Initialize a new Docker swarm.
  - Add two worker nodes to the swarm.
  - Create a new swarm service using the weather-app image.
  - Name the service weather-app.
  - Publish port 80 and map it to port 3000 on the container.
  - Create 3 replicas.

Begin by logging in to Swarm Server 1 using ssh:
```sh
ssh username@PUBLIC_IP_ADDRESS
```

Become the root user:
```sh
sudo su -
```

![image](https://user-images.githubusercontent.com/44756128/115292748-fd40bf80-a11b-11eb-9e4e-e012ac2bccd4.png)

Repeat these steps for Swarm Server 2 and Swarm Server 3 in new tabs.

# Initialize the Docker swarm
On Swarm Server 1:

Initialize the Docker swarm.
```sh
docker swarm init
```

![image](https://user-images.githubusercontent.com/44756128/115292813-16497080-a11c-11eb-986e-a9fa69a40bf9.png)

Copy the docker swarm join command that is displayed for the next step.

# Add additional nodes to the swarm
Add your worker nodes to the swarm.

Run the following command on Swarm Server 2 and Swarm Server 3:
```sh
docker swarm join --token TOKEN IP_ADDRESS:2377
```

![image](https://user-images.githubusercontent.com/44756128/115292903-3d07a700-a11c-11eb-95c9-763d52b1e7e7.png)

Note: This is the command that was copied from the previous step.

# Create a swarm service
From the master node (Swarm Server 1), create a service to test your swarm configuration.
```sh
docker service create --name weather-app --publish published=80,target=3000 --replicas=3 weather-app
```

Verify that this completed successfully:
```sh
docker service ls
```

![image](https://user-images.githubusercontent.com/44756128/115293004-63c5dd80-a11c-11eb-95d9-a69ccc6f5d8e.png)

We can also paste one of the public IP addresses of our worker nodes in a browser to view the weather app.

![image](https://user-images.githubusercontent.com/44756128/115293337-cb7c2880-a11c-11eb-9ac4-b4558bf2ce1e.png)
