![image](https://user-images.githubusercontent.com/44756128/115153165-3ef93980-a03a-11eb-82bb-a6b48601a226.png)

# Introduction
For the last six months, the Acme Anvil Corporation has been migrating some of their bare metal infrastructure to Docker containers. The initial implementation was very basic and lacked any kind of load balancing. Your manager has tasked you with creating two proofs of concept. For the first proof of concept, you are to use Docker Compose to create an Nginx load balancer and three instances using your weather-app image. Nginx will use port 80 and send traffic to port 3000 on the weather-app containers. For the second proof of concept, you are to create a Docker Swarm service called nginx-app that has two replicas using the Nginx image. The service should be published to port 8080 on the host and target port 80 on the containers.

# Create a Docker Compose file
On Swarm Server Manager

Change to the lb-challenge directory:
```sh
cd lb-challenge
```

Create our Docker compose file:
```sh
vi docker-compose.yml
```

The contents of your docker-compose.yml file should look like the following:
```yml
version: '3.2'
services:
  weather-app1:
      build: ./weather-app
      tty: true
      networks:
       - frontend
  weather-app2:
      build: ./weather-app
      tty: true
      networks:
       - frontend
  weather-app3:
      build: ./weather-app
      tty: true
      networks:
       - frontend
  loadbalancer:
      build: ./load-balancer
      image: nginx
      tty: true
      ports:
       - '80:80'
      networks:
       - frontend
networks:
  frontend:
```

![image](https://user-images.githubusercontent.com/44756128/115153485-c1cec400-a03b-11eb-8739-3ffecd188c37.png)

# Update nginx.conf
Change to the load-balancer directory:
```sh
cd load-balancer
```

Edit the nginx.conf file:
```sh
vi nginx.conf
```

The contents of your nginx.conf file should look like the following:
```
events { worker_connections 1024; }

http {
  upstream localhost {
    server weather-app1:3000;
    server weather-app2:3000;
    server weather-app3:3000;
  }
  server {
    listen 80;
    server_name localhost;
    location / {
      proxy_pass http://localhost;
      proxy_set_header Host $host;
    }
  }
}
```

# Execute docker-compose up
Change directories:
```sh
cd ../
```

Execute a docker-compose up:
```sh
docker-compose up --build -d
```

![image](https://user-images.githubusercontent.com/44756128/115153649-c051cb80-a03c-11eb-912d-9ad91677fc22.png)

Check our work:
```sh
docker ps
```

![image](https://user-images.githubusercontent.com/44756128/115153689-ec6d4c80-a03c-11eb-9139-0c3e351e8801.png)

Copy the public IP address of your Swarm Manager and paste it into a new tab in your browser.

![image](https://user-images.githubusercontent.com/44756128/115153746-1a529100-a03d-11eb-8610-462661c57227.png)

![image](https://user-images.githubusercontent.com/44756128/115153765-32c2ab80-a03d-11eb-9b41-ee96d4421dd4.png)

![image](https://user-images.githubusercontent.com/44756128/115153771-36eec900-a03d-11eb-96ea-69078e13ae1c.png)

# Create a Docker service using Docker Swarm
Change to the root directory:
```sh
cd ~/
```

View the contents of swarm-token.txt:
```sh
cat swarm-token.txt
```

Copy the docker swarm join command from the previous step.

![image](https://user-images.githubusercontent.com/44756128/115153793-5ede2c80-a03d-11eb-984c-fd5f897c992c.png)

## On Swarm Worker
Execute the command that was copied from the previous step.

![image](https://user-images.githubusercontent.com/44756128/115153853-9351e880-a03d-11eb-842e-38f874d8e10f.png)

## On Swarm Manager
Create a Docker service by executing the following command:
```sh
docker service create --name nginx-app --publish published=8080,target=80 --replicas=2 nginx
```

Verify this completed successfully by running the following command on both servers:
```sh
docker ps
```

![image](https://user-images.githubusercontent.com/44756128/115153878-b41a3e00-a03d-11eb-8059-5c9d837a1c51.png)

Verify that the default nginx page loads in the browser:
```
PUBLIC_IP_ADDRESS:8080
```

![image](https://user-images.githubusercontent.com/44756128/115153911-d6ac5700-a03d-11eb-9886-2e53104f1d33.png)
