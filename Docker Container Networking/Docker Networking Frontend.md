![image](https://user-images.githubusercontent.com/44756128/115743835-a58e9800-a357-11eb-89de-8bbf886abf58.png)

# Scenario
We're developing a new containerized application for a client. The application will consist of two containers: one for the frontend application, and one for the database. Our client has security concerns about the database, and they want it to run on a private network that is not publicly accessible.

So, we'll need to create two networks. One will house the frontend application that is publicly accessible, and the other network, which is flagged as internal, is where the database will reside. We have to create a MySQL container connected to the private network and an Nginx container that is connect to both networks.

# Create the frontend network
We're going to create a bridge network called frontend that will be publicly accessible:
```sh
docker network create frontend
```

![image](https://user-images.githubusercontent.com/44756128/115744421-2baade80-a358-11eb-96b4-a1c2db71122f.png)

# Create the localhost network
Now we've got to create a second bridge network called localhost that will be internal:
```sh
docker network create localhost --internal
```

![image](https://user-images.githubusercontent.com/44756128/115744455-336a8300-a358-11eb-9027-18aabd89fc1a.png)

# Create a MySQL container
We need a database, so we'll deploy a MySQL container called database that uses the localhost network, and runs in the background. We'll use the mysql 5.7 image:
```sh
docker container run -d --name database \
 --network localhost \ 
 -e MYSQL_ROOT_PASSWORD=P4ssW0rd0! \
 mysql:5.7
```

![image](https://user-images.githubusercontent.com/44756128/115744526-43826280-a358-11eb-9090-67241b2b9df3.png)

# Create an Nginx container
Next, we've got to deploy a second container call frontend-app and publish port 80 on the host to port 80 on the container. The container will be attached to both of the frontend networks. We'll use the latest Nginx image, and set the container should run in the background:
```sh
docker container run -d \ 
 --name frontend-app \
 --network frontend  \
 nginx:latest
```

![image](https://user-images.githubusercontent.com/44756128/115744733-74629780-a358-11eb-8138-d5bb88df2e76.png)

# Connect frontend-app to the internal network
Now that the Nginx container is created, and is connected to the internet, we've got to connect it to the localhost network. To make sure both containers are running, run a quick docker container ls. Once we're sure they're up, we can do the actual connecting:
```sh
docker network connect localhost frontend-app
```

![image](https://user-images.githubusercontent.com/44756128/115745169-ca373f80-a358-11eb-83f3-e42ee7894e4a.png)

Now let's peek at the container frontend-app:
```sh
[user@host]$ docker container inspect frontend-app
```

![image](https://user-images.githubusercontent.com/44756128/115745219-d58a6b00-a358-11eb-9f60-cb27d3fb158f.png)

# Conclusion
A lot flies by, but in the output, we'll see (In the Networks section) that it's connected to both the frontend and localhost networks. 

![image](https://user-images.githubusercontent.com/44756128/115745338-f3f06680-a358-11eb-9f51-913fdc8bc280.png)
