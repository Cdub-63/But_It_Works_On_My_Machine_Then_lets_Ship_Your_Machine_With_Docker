![image](https://user-images.githubusercontent.com/44756128/114838316-3682d380-9d9a-11eb-8294-6d261b1673b1.png)

# Introduction
Creating a container image by hand is possible, but it requires manual processes. There has to be a more automatic way to build images. Manual processes do not scale and are not easily version controlled. Docker provides a solution to this problem - the Dockerfile. We will create a Dockerfile to build an image, and host a static website.

Log in to the docker server using SSH:
```sh
ssh username@<PUBLIC_IP_ADDRESS>
```

# Build a First Version
Change to the widget-factory-inc directory:
```sh
cd widget-factory-inc
```

Create a Dockerfile that uses httpd:2.4 as the base image:
```sh
vim Dockerfile
```

In the new file, insert the following:
```sh
FROM httpd:2.4
RUN apt update -y && apt upgrade -y && apt autoremove -y && apt clean && rm -rf /var/lib/apt/lists*
```

Save the file:
```sh
ESC
:wq
```

Verify that the file was saved successfully:
```sh
cat Dockerfile
```

Build the 0.1 version of the widgetfactory image using the Dockerfile:
```sh
docker build -t widgetfactory:0.1 .
```

Set variables to examine the image's size and layers:
```sh
export showLayers='{{ range .RootFS.Layers }}{{ println . }}{{end}}'
export showSize='{{ .Size }}'
```

Compare the httpd and widgetfactory images:
```sh
docker images
```

![image](https://user-images.githubusercontent.com/44756128/114840860-d9d4e800-9d9c-11eb-8f4e-67d3a0a2b7c8.png)

Show the widgetfactory image's size:
```sh
docker inspect -f "$showSize" widgetfactory:0.1
```

Show the layers:
```sh
docker inspect -f "$showLayers" widgetfactory:0.1
```

Show the layers of the httpd:2.4 image:
```sh
docker inspect -f "$showLayers" httpd:2.4
```

![image](https://user-images.githubusercontent.com/44756128/114840980-fb35d400-9d9c-11eb-96b9-2f802e9651c1.png)

Compare the layers. Are they the same?

# Load the Website into the Container
Open the Dockerfile:
```sh
vim Dockerfile
```

Remove the Apache welcome page from the image by adding the following:
```sh
RUN rm -f /usr/local/apache2/htdocs/index.html
```

Save the file:
```sh
ESC
:wq
```

![image](https://user-images.githubusercontent.com/44756128/114841275-451eba00-9d9d-11eb-8cc8-2cddac2ab25a.png)

Build version 0.2 of the widgetfactory image:
```sh
docker build -t widgetfactory:0.2 .
```

Inspect both versions of the widgetfactory image to see the differences in size and layers:
```sh
docker images
```

Show the widgetfactory:0.1 image's size:
```sh
docker inspect -f "$showSize" widgetfactory:0.1
```
 
Compare it to the image size for widgetfactory:0.2:
```sh
docker inspect -f "$showSize" widgetfactory:0.2
```

Using an interactive terminal, check the htdocs folder for widgetfactory:0.2. Are the website files in the folder?:
```sh
docker run --rm -it widgetfactory:0.2 bash
ls htdocs
```

Exit the container:
```sh
exit
```

Show the layers for the widgetfactory:0.1 image:
```sh
docker inspect -f "$showLayers" widgetfactory:0.1
```

Show the layers for the widgetfactory:0.2 image and compare the two:
```sh
docker inspect -f "$showLayers" widgetfactory:0.2
```

Open the Dockerfile:
```sh
vim Dockerfile
```

Add the website data to the container by adding the following to the end of the file:
```sh
WORKDIR /usr/local/apache2/htdocs
COPY ./web .
```

Save the file:
```sh
ESC
:wq
```

![image](https://user-images.githubusercontent.com/44756128/114841636-b1012280-9d9d-11eb-8d7a-81f54b0e6053.png)

Build version 0.3 of the widgetfactory image:
```sh
docker build -t widgetfactory:0.3 .
```

Inspect versions 0.2 and 0.3 to see the differences in size and layers:
```sh
docker images
```

Show the widgetfactory:0.2 image's size:
```sh
docker inspect -f "$showSize" widgetfactory:0.1
```

Compare it to the image size for widgetfactory:0.3:
```sh
docker inspect -f "$showSize" widgetfactory:0.2
```

Show the layers for the widgetfactory:0.2 image:
```sh
docker inspect -f "$showLayers" widgetfactory:0.1
```

Show the layers for the widgetfactory:0.3 image and compare the two:
```sh
docker inspect -f "$showLayers" widgetfactory:0.2
```

Using an interactive terminal, check the htdocs folder for widgetfactory:0.3:
```sh
docker run --rm -it widgetfactory:0.3 bash
```

Are the website files in the folder?:
```sh
ls -l
```

![image](https://user-images.githubusercontent.com/44756128/114841970-050c0700-9d9e-11eb-8c39-556f4284a45e.png)

# Run a Container from the Image
Run a container from the widgetfactory:0.3 image. What commmand does it use to run the server? Remember to publish the web server port:
```sh
docker run --name web1 -p 80:80 widgetfactory:0.3
```

Exit the container:
```sh
CTRL+C
```

Check the status of the container:
```sh
docker ps -a
```

Start the container:
```sh
docker start web1
```

Using docker exec connect to the web1 container:
```sh
docker exec -it web1 bash
```

View the website files in the container:
```sh
ls -l
```

Exit the container:
```sh
exit
```

Retrieve the main website page from the container:
```sh
wget localhost
```

Compare it to the copy on the server:
```sh
diff index.html web/index.html
```

![image](https://user-images.githubusercontent.com/44756128/114842308-5916eb80-9d9e-11eb-925d-ccadc8494f11.png)

# Read Me
Note: Many of the things done here are intentionally inefficient, These tasks are done to demonstrate certain aspects of Docker, and highlight common pitfalls. These do not represent best practices.
