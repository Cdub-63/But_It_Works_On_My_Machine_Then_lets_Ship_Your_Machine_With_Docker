![image](https://user-images.githubusercontent.com/44756128/115484985-6d813b00-a219-11eb-9ec8-90c3ad3882e0.png)

We've got to create a couple of Docker swarm master nodes, and three worker nodes, then practice scaling replicas up and down.

There are five servers we'll need to log into: two will be swarm masters, and three will be swarm workers.

If we fire up five terminals, and log into each of the servers, we'll be able to switch back and forth quickly. And since we'll need to be root in all of them, we can just run a sudo su - as soon as we log into each one. Once we're in, we can get moving with Docker.

# Create a Swarm
On our first server, we're going to create a swarm master. Initialize the swarm with this:
```sh
docker swarm init
```

![image](https://user-images.githubusercontent.com/44756128/115485575-a077fe80-a21a-11eb-908d-d9937afc3a31.png)

There's going to be some output, including a docker swarm join command that we can copy.

# Add Worker Nodes
On our worker node servers, we can join the swarm by pasting in the command that our swarm master spit out. Add the 3 worker nodes with these:

## Worker 1
```sh
docker swarm join --token <TOKEN> <IP_ADDRESS:2377>
```

![image](https://user-images.githubusercontent.com/44756128/115485670-ca312580-a21a-11eb-890d-58cc9c1fc6fe.png)

## Worker 2
```sh
docker swarm join --token <TOKEN> <IP_ADDRESS:2377>
```

![image](https://user-images.githubusercontent.com/44756128/115485683-d1f0ca00-a21a-11eb-9742-0b6754af865f.png)

## Worker 3
```sh
docker swarm join --token <TOKEN> <IP_ADDRESS:2377>
```

![image](https://user-images.githubusercontent.com/44756128/115485699-d917d800-a21a-11eb-90fc-3501b2cc43ca.png)

# The Master Token
Back on swarmmaster1, we need to generate the master token:
```sh
docker swarm join-token manager
```

![image](https://user-images.githubusercontent.com/44756128/115485779-f9479700-a21a-11eb-9716-22a2b93287f7.png)

This will output yet another command. The first one we copied and pasted was for getting a worker node joined. This one is for when we want a master node to join. Let's log into the second master, and paste that command:
```sh
[root@swarmmaster2]# docker swarm join --token <TOKEN> <IP_ADDRESS:2377>
```

![image](https://user-images.githubusercontent.com/44756128/115485824-0bc1d080-a21b-11eb-888f-307847e50966.png)

# Check Our Work
A quick docker node ls from our first master server will show us that there are five nodes running. Three are workers, and two have a MANAGER STATUS: Leader (the first master) and Reachable (the second master).

![image](https://user-images.githubusercontent.com/44756128/115485872-24ca8180-a21b-11eb-84d5-3b0d48fe9c48.png)

# Ensure that the Two Masters Will Only Function as Masters
We want the masters to just be masters, not workers too. To do that we run these (one for each master):

Set the availability to drain for Master1.
```sh
docker node update --availability drain <MASTER1 ID>
```

Set the availability to drain for Master2.
```sh
docker node update --availability drain <MASTER2 ID>
```

![image](https://user-images.githubusercontent.com/44756128/115486006-622f0f00-a21b-11eb-9a07-ad3dde5b5e73.png)

<MASTER ID> here would be the ID column in that docker node ls command we ran earlier.

# Create a Service
We're going to create a service, and replicate it three times.
```sh
docker service create --name httpd -p 80:80 --replicas 3 httpd
```

![image](https://user-images.githubusercontent.com/44756128/115486136-a3bfba00-a21b-11eb-997b-ff644bb235b3.png)

In this command, we created the service, named it httpd, ran it on port 80, created three replicas, and used an image called httpd that was pulled from Docker Hub.

# Check What Is Running
We can look at what actually fired up with another docker command:
```sh
docker service ps httpd
```

![image](https://user-images.githubusercontent.com/44756128/115486170-b6d28a00-a21b-11eb-9376-f91fef4ea7ab.png)

The output shows that we have three replicas running, one on each of our worker nodes, and that they're only running on our worker nodes. Great. Now let's scale up.

# Scale the httpd Service up to Five Nodes
Instead of just running three replicas, let's scale the httpd service up to five:
```sh
docker service scale httpd=5
```

![image](https://user-images.githubusercontent.com/44756128/115486218-d23d9500-a21b-11eb-8a99-4ad7b5424d70.png)

If we run another docker service ps httpd, we'll see there are now five replicas running. If we look at the nodes, we'll also see that it's still only the worker nodes running these. This is what we intended though: we want our master being masters, and just want the workers working.

![image](https://user-images.githubusercontent.com/44756128/115486259-ec777300-a21b-11eb-8d4e-d0da0d28e14b.png)

Let's scale down and see what happens.

# Scale the httpd Service Back Down to Two Nodes
We're going to cut our workforce by three, so that we're only running two replicas:
```sh
docker service scale httpd=2
```

![image](https://user-images.githubusercontent.com/44756128/115486304-07e27e00-a21c-11eb-978d-a938eaa4a66d.png)

Now we can run docker service ps httpd again, and we'll see that three of the replicas have disappeared, leaving us with two.

![image](https://user-images.githubusercontent.com/44756128/115486327-16c93080-a21c-11eb-8c3a-dcbf94eb4563.png)
