![image](https://user-images.githubusercontent.com/44756128/113725885-63a7f580-96b9-11eb-84b0-856ea0f5523d.png)

# Introduction
Docker is the leading containerization platform. If you are using containers, you are likely using Docker. In order to work with Docker however, you must have the Docker daemon, and CLI available.

The pupose here is to set up your environment, so you can get started working with Docker today!

Log into your server using ssh:
```sh
ssh username@<PUBLIC_IP_ADDRESS>
```

# Installing Docker
Install the Docker prerequisites:
```sh
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```

![image](https://user-images.githubusercontent.com/44756128/113726313-bd102480-96b9-11eb-8d57-a7986b9834d7.png)

Using yum-config-manager, add the CentOS-specific Docker repo:
```sh
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

![image](https://user-images.githubusercontent.com/44756128/113726368-cef1c780-96b9-11eb-91ab-b115f137c222.png)

Install Docker:
```sh
sudo yum -y install docker-ce
```

![image](https://user-images.githubusercontent.com/44756128/113726642-052f4700-96ba-11eb-8d86-c5054dd3436f.png)

# Enable the Docker Daemon
Enable the Docker daemon:
```sh
sudo systemctl enable --now docker
```

![image](https://user-images.githubusercontent.com/44756128/113726767-2132e880-96ba-11eb-9680-4c8a61b7cc9f.png)

# Configure User Permissions
Add the lab user to the docker group:
```sh
sudo usermod -aG docker cloud_user
```

![image](https://user-images.githubusercontent.com/44756128/113726875-37d93f80-96ba-11eb-9298-6964bf266809.png)

Note: You will need to exit the server for the change to take effect.

# Run a Test Image
Using docker, run the hello-world image to verify that the environment is set up properly:
```sh
docker run hello-world
```

![image](https://user-images.githubusercontent.com/44756128/113727109-71aa4600-96ba-11eb-9c85-0a2940ca4c5b.png)
