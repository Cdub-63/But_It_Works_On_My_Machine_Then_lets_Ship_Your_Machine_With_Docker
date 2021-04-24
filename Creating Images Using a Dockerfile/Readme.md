![image](https://user-images.githubusercontent.com/44756128/115963457-f4b60380-a4e4-11eb-8f05-8432585f7178.png)

# Introduction
Here we'll build an image and then test it by creating a container that uses it.

# Creating Images Using a Dockerfile
## Create a Dockerfile.
Change directory to weather-app.
```sh
cd weather-app/
```

Open the editor:
```sh
vi Dockerfile
```

Enter the Dockerfile contents:
```sh
FROM node AS source
RUN mkdir -p /node/weather-app
ADD src/ /node/weather-app
WORKDIR /node/weather-app
RUN npm install

FROM node:alpine
ARG APP_VERSION=V1.1
LABEL org.label-schema.version=$APP_VERSION
ENV NODE_ENV="production"
COPY --from=source /node/weather-app /node/weather-app
WORKDIR /node/weather-app
EXPOSE 3000
ENTRYPOINT ["./bin/www"]
```

![image](https://user-images.githubusercontent.com/44756128/115963623-ba009b00-a4e5-11eb-8b64-8b9d046544c9.png)

## Build the image.
Change to the source directory:
```sh
cd src
```

Enter the following to get the Git commit hash:
```sh
git log -1 --pretty=%H
```

Drop back to the weather-app directory:
```sh
cd ../
```

Build the image:
```sh
sudo docker image build -t linuxacademy/weather-app:<HASH> --build-arg APP_VERSION=2.0 .
```

![image](https://user-images.githubusercontent.com/44756128/115963698-28455d80-a4e6-11eb-99b4-b55ee702129b.png)

# Deploy a test container using the image just created.
Create the weather-app container:
```sh
sudo docker container run -d --name weather-app -p 8080:3000 linuxacademy/weather-app:<HASH>
```

Make sure the container is running.
```sh
docker container ls
```

Verify everything is working properly.
```sh
curl localhost:8080
```

![image](https://user-images.githubusercontent.com/44756128/115963781-7f4b3280-a4e6-11eb-9f22-ccf86f83d183.png)
