![image](https://user-images.githubusercontent.com/44756128/115110046-e729d800-9f3e-11eb-8c1e-0459a768d2aa.png)

# Introduction
For the last six months, the Acme Anvil Corporation has been migrating some of their bare metal infrastructure to Docker containers. After the initial implementation, you mention to the team that labels can be used for storing metadata about images and containers. This is useful for keeping track of image and container attributes like the build date and application version.

Your team thinks this is a great idea, and you’ve been tasked with creating a quick demo to show how useful labels can be. You created a small weather app a few months ago while learning Node.JS, and it’s perfect for a quick demo. On your Docker workstation, you will create a Dockerfile that contains four labels: the maintainer, build date, application name, and application version. You will then build the image and push it to Docker Hub. On your Docker server, you will create a new container using the weather-app image. Once the container is running, you will update the branch of your application, rebuild the image, and push the changes to Docker Hub. On the Docker server, Watchtower will update the running container with the new version of the image.

Log in to the Docker Server and Docker Workstation ssh:
```sh
ssh username@<PUBLIC_IP_ADDRESS>
```

In my case, 10.0.1.20 is my docker server and 10.0.1.10 is my docker workstation.

Become the root user:
```sh
sudo su -
```

# Create a Dockerfile
On Docker Workstation:

Create a Dockerfile:
```sh
vi Dockerfile
```

Use the following instructions:

Note: Replace EMAIL_ADDRESS with your email address.
```sh
FROM node

LABEL maintainer="EMAIL_ADDRESS"

ARG BUILD_VERSION
ARG BUILD_DATE
ARG APPLICATION_NAME

LABEL org.label-schema.build-date=$BUILD_DATE
LABEL org.label-schema.application=$APPLICATION_NAME
LABEL org.label-schema.version=$BUILD_VERSION

RUN mkdir -p /var/node
ADD weather-app/ /var/node/
WORKDIR /var/node
RUN npm install
EXPOSE 3000
CMD ./bin/www
```

# Build the Docker Image
Log in to Docker Hub:
```sh
docker login
```

![image](https://user-images.githubusercontent.com/44756128/115110507-64564c80-9f41-11eb-8347-4f2e37507457.png)

Build the Docker image using the following parameters:

Note: Be sure to replace <DOCKER_USERNAME> with your Docker Hub username.
```sh
docker build -t <DOCKER_USERNAME>/weather-app --build-arg BUILD_DATE=$(date -u +'%Y-%m-%dT%H:%M:%SZ') \
--build-arg APPLICATION_NAME=weather-app --build-arg BUILD_VERSION=v1.0  -f Dockerfile .
```

![image](https://user-images.githubusercontent.com/44756128/115110593-d7f85980-9f41-11eb-82e3-b3865f79c7e4.png)

Show image ID:
```sh
docker images
```

Use image ID to inspect:
```sh
docker inspect <IMAGE_ID>
```

![image](https://user-images.githubusercontent.com/44756128/115110631-024a1700-9f42-11eb-9388-658285eb46f9.png)

# Push the Image to Docker Hub
Push the weather-app image to Docker Hub.

Note: Be sure to replace <DOCKER_USERNAME> with your Docker Hub username.
```sh
docker push <DOCKER_USERNAME>/weather-app
```

![image](https://user-images.githubusercontent.com/44756128/115110670-26a5f380-9f42-11eb-8279-dcbfdf4c7d76.png)

![image](https://user-images.githubusercontent.com/44756128/115110698-4ccb9380-9f42-11eb-8890-7fd3ae8e4c0c.png)

# Create the weather-app Container
On the Docker Server:

Start the weather-app container.

Note: Be sure to replace <DOCKER_USERNAME> with your Docker Hub username.
```sh
docker run -d --name demo-app -p 80:3000 --restart always <DOCKER_USERNAME>/weather-app
```

Verify the image is running:
```sh
docker ps
```

![image](https://user-images.githubusercontent.com/44756128/115110818-e2672300-9f42-11eb-8127-74928f0f336a.png)

# Check Out Version 1.1 of the Weather App
On the Docker Workstation:

In the weather-app directory, check out version 1.1 of the weather app:
```sh
cd weather-app
git checkout v1.1
git branch
cd ../
```

![image](https://user-images.githubusercontent.com/44756128/115110848-00348800-9f43-11eb-9cba-3b1fea5b1461.png)

# Rebuild the weather-app Image
Rebuild and push the weather-app image.

Note: Be sure to replace <DOCKER_USERNAME> with your Docker Hub username.
```sh
docker build -t <DOCKER_USERNAME>/weather-app --build-arg BUILD_DATE=$(date -u +'%Y-%m-%dT%H:%M:%SZ') \
--build-arg APPLICATION_NAME=weather-app --build-arg BUILD_VERSION=v1.1  -f Dockerfile .
```

![image](https://user-images.githubusercontent.com/44756128/115110913-4853aa80-9f43-11eb-829e-88f73d7f67f9.png)

Just to double check, inspect the image (running docker images first to get the image ID):
```sh
docker inspect <IMAGE_ID>
```

![image](https://user-images.githubusercontent.com/44756128/115110942-72a56800-9f43-11eb-857c-acc6536a6bf6.png)

We should be running v1.1 now, so let's push it up:
```sh
docker push <DOCKER_USERNAME>/weather-app
```

![image](https://user-images.githubusercontent.com/44756128/115110979-a8e2e780-9f43-11eb-88eb-c5f9329230a9.png)

On the Docker Server:

Show image status:
```sh
docker ps
```

Using the container ID from the previous command, inspect the weather-app image:
```sh
docker inspect <IMAGE_ID>
```

![image](https://user-images.githubusercontent.com/44756128/115111029-e34c8480-9f43-11eb-8ad5-6493deaabc17.png)

This one should also be at v1.1.

![image](https://user-images.githubusercontent.com/44756128/115111056-0bd47e80-9f44-11eb-9357-575bdbd1aad4.png)
