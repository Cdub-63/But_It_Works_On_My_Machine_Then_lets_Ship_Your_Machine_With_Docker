![image](https://user-images.githubusercontent.com/44756128/113729301-7e2f9e00-96bc-11eb-92a9-38e5b958f54a.png)

# Introduction
After installation, the best way to familiarize yourself with Docker, is to run containers from a few prebuilt images.

We will explore Docker Hub for images that will run a website. Once we find suitable images, we will get them into our development environment and begin experimenting. We will run, stop, and delete containers from those images. We will also learn how to use existing data in containers.


Log into your server using ssh:
```sh
ssh username@<PUBLIC_IP_ADDRESS>
```

# Explore Docker Hub
  - Sign in to Docker Hub (https://hub.docker.com/).
  - At the top of the page, search for "httpd".
  - In the left-hand menu, filter for Application Infrastructure, and Official Images.
  - Select the httpd project.
  ![image](https://user-images.githubusercontent.com/44756128/113729496-af0fd300-96bc-11eb-8d5e-942624c27f99.png)

  - At the top of the page, click the Tags tab.
  - Under latest, select linux/amd64.
  ![image](https://user-images.githubusercontent.com/44756128/113729730-e41c2580-96bc-11eb-9ea3-c20f6d01520a.png)

  - Back in the list of available images, select nginx
  ![image](https://user-images.githubusercontent.com/44756128/113730141-3d845480-96bd-11eb-93fd-c3918c1a3d7b.png)

  - Review the How to use this image section.
  - ![image](https://user-images.githubusercontent.com/44756128/113730200-4bd27080-96bd-11eb-9b42-80d48a0c580d.png)

# Get and View httpd
In the Docker Instance, verify that docker is installed:
```sh
docker ps
```

Using docker, pull the httpd:2.4 image:
```sh
docker pull httpd:2.4
```

Run the image:
```sh
docker run --name httpd -p 8080:80 -d httpd:2.4
```

Check the status of the container:
```sh
docker ps
```

![image](https://user-images.githubusercontent.com/44756128/113730513-95bb5680-96bd-11eb-90e1-2bd279bd87dd.png)

In a web browser, test connectivity to the container:
```sh
<PUBLIC_IP_ADDRESS>:8080
```

![image](https://user-images.githubusercontent.com/44756128/113730976-fc407480-96bd-11eb-814e-41d000613922.png)

# Run a Copy of the Website in httpd
Clone the Widget Factory Inc repository:
```sh
git clone https://github.com/linuxacademy/content-widget-factory-inc
```

Change to the content-widget-factory-inc directory:
```sh
cd content-widget-factory-inc
```

Check the files:
```sh
ll
```

Move to the web directory:
```sh
cd web
```

Check the files:
```sh
ll
```

![image](https://user-images.githubusercontent.com/44756128/113731435-6822dd00-96be-11eb-83ca-448759e3c2e9.png)

Stop the httpd container:
```sh
docker stop httpd
```

Remove the httpd container:
```sh
docker rm httpd
```

Verify that the container has been removed:
```sh
docker ps -a
```

Run the container with the website data:
```sh
docker run --name httpd -p 8080:80 -v $(pwd):/usr/local/apache2/htdocs:ro -d httpd:2.4
```

Check the status of the container:
```sh
docker ps
```

![image](https://user-images.githubusercontent.com/44756128/113731595-85f04200-96be-11eb-904b-1e5dc2dadb74.png)

In a web browser, check connectivity to the container:
```sh
<PUBLIC_IP_ADDRESS>:8080
```

![image](https://user-images.githubusercontent.com/44756128/113731761-a4563d80-96be-11eb-90b9-5c0f37c8bc44.png)

# Get and View Nginx
Using docker, pull the latest version of nginx:
```sh
docker pull nginx
```

Verify that the image was pulled successfully:
```sh
docker images
```

![image](https://user-images.githubusercontent.com/44756128/113732042-dd8ead80-96be-11eb-97f1-99aca7aea1ae.png)

Run the container using the nginx image:
```sh
docker run --name nginx -p 8081:80 -d nginx 
```

Check the status of the container:
```sh
docker ps
```

![image](https://user-images.githubusercontent.com/44756128/113732186-fd25d600-96be-11eb-85d7-d3473e5a86d6.png)

Verify connectivity to the nginx container:
```sh
<PUBLIC_IP_ADDRESS>:8081
```

![image](https://user-images.githubusercontent.com/44756128/113732281-13cc2d00-96bf-11eb-868c-2b9608fd72ce.png)

# Run a Copy of the Website in Nginx
Stop the nginx container:
```sh
docker stop nginx
```

Remove the nginx container:
```sh
docker rm nginx
```

Verify that the container has been removed:
```sh
docker ps -a
```

![image](https://user-images.githubusercontent.com/44756128/113732629-5a218c00-96bf-11eb-8bf7-7e44b2b354de.png)

Run the nginx container, and mount the website data:
```sh
docker run --name nginx -v $(pwd):/usr/share/nginx/html:ro -p 8081:80 -d nginx
```

Check the status of the container:
```sh
docker ps
```

![image](https://user-images.githubusercontent.com/44756128/113732686-66a5e480-96bf-11eb-9948-7ad82adc238a.png)

In a web browser, verify connectivity to the container:
```sh
<PUBLIC_IP_ADDRESS>:8081
```

![image](https://user-images.githubusercontent.com/44756128/113732769-7f15ff00-96bf-11eb-9dbd-76aa2790c2ef.png)

Stop the nginx container:
```sh
docker stop nginx
```

Remove the nginx container:
```sh
docker rm nginx
```

Verify that the container has been removed:
```sh
docker ps -a
```

![image](https://user-images.githubusercontent.com/44756128/113732926-9ce36400-96bf-11eb-9853-6ae701f73b76.png)
