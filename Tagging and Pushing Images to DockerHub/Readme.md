![image](https://user-images.githubusercontent.com/44756128/115964234-13b69480-a4e9-11eb-92cc-d98a5f3818a2.png)

# The Scenario
We've just completed building a Dockerfile, and we're ready to push it to Docker Hub. We need to build our image with the VERSION build argument that is set to 1.5, then tag the image to latest, and finally push both images to Docker Hub.

First make sure that you have a Docker Hub account. We'll need it.

# Log in to Docker Hub
Login to Docker Hub:
```sh
docker login -u [DOCKER_HUB_USERNAME]
```

![image](https://user-images.githubusercontent.com/44756128/115964377-d69ed200-a4e9-11eb-8b6e-92b3f7536a06.png)

# Get the Git Commit Hash
Use Git commit hash as the image tag:
```sh
cd weather-app
cd src
git log -1 --pretty=%H
```

![image](https://user-images.githubusercontent.com/44756128/115964387-e61e1b00-a4e9-11eb-90a5-206f49fab958.png)

The next line will be the has we need.

# Build the weather-app Image
Now let's move up a directory and build the image:
```sh
cd ../
docker image build -t [USERNAME]/weather-app:[HASH] --build-arg VERSION=1.5 .
```

![image](https://user-images.githubusercontent.com/44756128/115964392-edddbf80-a4e9-11eb-9d69-402b13db4954.png)

# Tag the weather-app Image as Latest
```sh
[cloud_user@host]$ docker image tag [USERNAME]/weather-app:[HASH] [USERNAME]/weather-app:latest
```

![image](https://user-images.githubusercontent.com/44756128/115964449-2ed5d400-a4ea-11eb-835a-0db3b5e0a94e.png)

# Push Both Images to Docker Hub
```sh
docker image push [USERNAME]/weather-app:[HASH]
docker image push [USERNAME]/weather-app:latest
```

![image](https://user-images.githubusercontent.com/44756128/115964470-42813a80-a4ea-11eb-9af1-c7b292e40885.png)

![image](https://user-images.githubusercontent.com/44756128/115964500-66dd1700-a4ea-11eb-956f-8c815461d0a9.png)
