![image](https://user-images.githubusercontent.com/44756128/116007448-6cfbf200-a5d5-11eb-8a9a-e86bcb7d23c7.png)

# Description
Tired of having to manually update several containers, you’ve decided to look for an automated solution.

After doing some research, you have discovered Watchtower. Watchtower is a container that updates all running containers when changes are made to the image that it is running.

You will need to create a Dockerfile that will be used to create a Docker image. The image will be pushed to Docker Hub.

Next, you will create a container using this image. Once the image is created, you will deploy the Watchtower container. After Watchtower is deployed, you will update the Dockerfile, rebuild the image, and push the changes to Docker Hub.

Watchtower checks for changes every 30 seconds. Once it detects the changes, Watchtower will update the running container.

For more information on Watchtower you can go here:

https://github.com/containrrr/watchtower

We will need a Docker Hub account to accomplish this project.

# Create the Dockerfile
We need to create a Dockerfile (with any text editor you like). First get into the lab directory:
```sh
cd lab
```

Now create the Dockerfile (we're using vi here):
```sh
vi Dockerfile
```

Now put the following into it:
```sh
FROM node
RUN mkdir -p /var/node
ADD src/ /var/node/
WORKDIR /var/node
RUN npm install
EXPOSE 3000
CMD ./bin/www
```

![image](https://user-images.githubusercontent.com/44756128/116007492-a3397180-a5d5-11eb-9983-137e14e711c0.png)

# Log in to Docker Hub
```sh
docker login -u [DOCKER_HUB_USERNAME]
```

![image](https://user-images.githubusercontent.com/44756128/116007555-e5fb4980-a5d5-11eb-833f-7d9abc077149.png)

# Build the Docker Image
```sh
docker image build -t [USERNAME]/lab-watchtower -f Dockerfile .
```

![image](https://user-images.githubusercontent.com/44756128/116007579-04f9db80-a5d6-11eb-8061-f9d2286d603b.png)

# Create a Demo Container
Create the container that Watchtower will monitor:
```sh
docker container run -d --name demo-app -p 80:3000 --restart always [USERNAME]/lab-watchtower
```

Make sure it's running:
```sh
docker container ls
```

![image](https://user-images.githubusercontent.com/44756128/116007620-44c0c300-a5d6-11eb-8ecb-e6940ccfdc05.png)

# Create the Watchtower Container
This next container will be the one monitoring the demo-app container. Start it up with this:
```sh
docker run -d --name watchtower --restart always -v /var/run/docker.sock:/var/run/docker.sock v2tec/watchtower -i 30
```

List the containers again to make sure the watchtower container is running:
```sh
docker container ls
```

![image](https://user-images.githubusercontent.com/44756128/116007640-66ba4580-a5d6-11eb-8b6a-02c75c3cdc88.png)

# Update the Docker Image
We need to edit Dockerfile, making a change that Watchtower will see and implement. Add a mkdir command after the one that's already in there:
```sh
FROM node
RUN mkdir -p /var/node
RUN mkdir -p /var/test
ADD src/ /var/node/
WORKDIR /var/node
RUN npm install
EXPOSE 3000
CMD ./bin/www
```

![image](https://user-images.githubusercontent.com/44756128/116007660-89e4f500-a5d6-11eb-9c1b-c47ced6b4f62.png)

# Rebuild the Image:
```sh
docker build -t [USERNAME]/lab-watchtower -f Dockerfile .
```

![image](https://user-images.githubusercontent.com/44756128/116007697-b7ca3980-a5d6-11eb-9850-f1d6c16c2444.png)

# Push the New Image:
```sh
docker image push [USERNAME]/lab-watchtower
```

![image](https://user-images.githubusercontent.com/44756128/116007721-d4667180-a5d6-11eb-9ff6-60b58c166198.png)

# Checking
Now if we run docker container ls periodically, we'll eventually see that the demo-app container has a newer CREATED time than the watchtower container does. This means that Watchtower saw the change we made, and restarted the other container using the new image.

![image](https://user-images.githubusercontent.com/44756128/116007766-f3fd9a00-a5d6-11eb-809c-ffacb1ad73f3.png)

# Conclusion
Our days of manually updating containers are over! Watchtower will see any changes we make to images, then go ahead and implement them automatically.
