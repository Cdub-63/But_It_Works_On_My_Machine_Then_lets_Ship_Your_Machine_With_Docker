![image](https://user-images.githubusercontent.com/44756128/115109275-61a42900-9f3a-11eb-9a17-ee95b42b5efe.png)

# Introduction
We'll be using Watchtower to monitor containers for updates, and then update them with the newer versions that are sitting up in Docker Hub.

# Prerequisites
In order to complete this, you will need a Docker Hub account (https://hub.docker.com/).

# Starting Off
Let's log into the live environment and become root:
```sh
sudo su -
```

# Create the Docker file
Let's create the Docker file in a text editor (we're using vi):
```sh
vi Dockerfile
```

We need to set these parameters and conditions:
  - The base image should be node.
  - Using the RUN instruction, make a directory called /var/node.
  - Use the ADD instruction to add the contents of the code directory into /var/node.
  - Make /var/node the working directory.
  - Execute an npm install.
  - Set ./bin/www as the command.
  - From the command line, log in to Docker Hub.
  - Build your image using <USERNAME>/express.
  - Push the image to Docker Hub.
  
When we're finished, the file should look like this:
```sh 
FROM node

RUN mkdir -p /var/node
ADD content-express-demo-app/ /var/node/
WORKDIR /var/node
RUN npm install
CMD ./bin/www
```

# Log in to Docker Hub
We've got to build our Docker image, but before we do that, let's get logged into our Docker Hub account. Do it from the command line, like this:
```sh
docker login
```

![image](https://user-images.githubusercontent.com/44756128/115109633-91ecc700-9f3c-11eb-9f94-667310e1765b.png)

# Build the Docker Image
Now we can build the image, where USERNAME in the command is your Docker Hub username:
```sh
docker build -t <USERNAME>/express -f Dockerfile .
``` 

![image](https://user-images.githubusercontent.com/44756128/115109661-b21c8600-9f3c-11eb-841f-b7c17dcc2cba.png)

# Push the Image to Docker Hub
Push the image up to Docker Hub with this:
```sh
docker push <USERNAME>/express
```

![image](https://user-images.githubusercontent.com/44756128/115109676-d11b1800-9f3c-11eb-8f67-1874960b88ca.png)

If we log into the Docker Hub with a web browser, we should see the image in our repository.

![image](https://user-images.githubusercontent.com/44756128/115109692-f019aa00-9f3c-11eb-9909-3ea5b89d91f0.png)

# Create the Demo Container
In order for Watchtower to watch anything, we've got to give it something to keep an eye on. We're going to create a Docker container and set some parameters, all in one command:
```sh
docker run -d --name demo-app -p 80:3000 --restart always <USERNAME>/express
```

To make sure it's running, let's execute a quick docker ps. We should see it in the list.

![image](https://user-images.githubusercontent.com/44756128/115109736-31aa5500-9f3d-11eb-9342-6a29273e9316.png)

# Create the Watchtower Container
Now we'll spin up Watchtower to watch our demo-app container. We can create the Watchtower container with this:
```sh
docker run -d --name watchtower --restart always -v /var/run/docker.sock:/var/run/docker.sock v2tec/watchtower -i 30
```

Again, run a docker ps real quick to make sure this container spun up properly.

![image](https://user-images.githubusercontent.com/44756128/115109769-58688b80-9f3d-11eb-9fe5-7376be47e7a3.png)

# Testing
We want to see if Watchtower is actually keeping watch, so we've got to take a couple more steps.

## Update the Docker Image
We're going to edit Dockererfile, rebuild the image, then push the new one up to Docker Hub.

Edit Dockerimage, and let's make a simple change, like a mkdir command. Add it below the existing one, so that the file looks like this now:
```sh
FROM node

RUN mkdir -p /var/node
RUN mkdir -p /var/test
ADD content-express-demo-app/ /var/node/
WORKDIR /var/node
RUN npm install
CMD ./bin/www
```

Rebuild the image with this:
```sh
docker build -t <USERNAME>/express -f Dockerfile .
```

Now push it up to Docker Hub:
```
docker push <USERNAME>/express
```

![image](https://user-images.githubusercontent.com/44756128/115109857-ed6b8480-9f3d-11eb-9b4c-f6793bbeaa67.png)

# Watch the Magic of Watchtower
Since we set the interval at 30 seconds (the -i in the last docker run command we ran), then our Watchtower container should be checking for updates every 30 seconds. And if we run docker ps in a little bit, we're going to see that the demo-app container gets updated.

![image](https://user-images.githubusercontent.com/44756128/115109919-39b6c480-9f3e-11eb-99b4-1f8204b7c93d.png)

# Conclusion
We've set up a system that watches for changes in container images up in Docker Hub, then updates our other ones out in the wild with those new container images.
