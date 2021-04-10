![image](https://user-images.githubusercontent.com/44756128/114278068-9290d500-99f3-11eb-8190-e89bdd0bdff5.png)

# Introduction
Each container should serve a single purpose, such as running one application like a web server. Containers can be powerful by themselves, but when connected together, they are far more useful.

For example, a web server container can be connected to a database container to provide application storage. Docker provides multiple options for networking containers.

We'll explore a few of the common types of networks that Docker supports, and learn how containers within those networks interact.

Log into the server using ssh:
```sh
ssh username@<PUBLIC_IP_ADDRESS>
```

# Explore the Default Network
List the default networks:
```sh
docker network ls
```

Run an httpd container named web1, without specifying a network, and see which network it uses:
```sh
docker run -d --name web1 httpd:2.4
docker inspect web1
```

![image](https://user-images.githubusercontent.com/44756128/114278346-0384bc80-99f5-11eb-89d4-fa35c196924f.png)

Take note of the IPAddress.

![image](https://user-images.githubusercontent.com/44756128/114278326-e94ade80-99f4-11eb-94b8-5fe30e3cd56a.png)

Run a container using the busybox image, and see if you can connect to the web1 server:
```sh
docker run --rm -it busybox
```

Check the container's networking, and verify it is in the same IP range as web1:
```sh
ip addr
```

Ping the web1 container using the IP address:
```sh
ping <WEB1_IP_ADDRESS>
```

Attempt to ping the web1 container by name:
```sh
ping web1
```

Attempt to access web1 using wget:
```sh
wget <WEB1_IP_ADDRESS>
```

Exit the container:
```sh
exit
```

![image](https://user-images.githubusercontent.com/44756128/114278418-6a09da80-99f5-11eb-87aa-1afd61591062.png)

# Explore Bridge Networks
Create a new bridge network named test_application:
```sh
docker network create test_application
```

Run an httpd container named web2, in the test_application network:
```sh
docker run -d --name web2 --network test_application httpd:2.4
```

Check the status of the container:
```sh
docker ps -a
```

Verify that web2 was added to the test_application network:
```sh
docker inspect web2
```

![image](https://user-images.githubusercontent.com/44756128/114278487-b228fd00-99f5-11eb-82a6-4771399a1985.png)

![image](https://user-images.githubusercontent.com/44756128/114278509-d258bc00-99f5-11eb-99e3-d29b3137b58a.png)

Run a container using the busybox image, and see if you can connect to the web2 server, within the test_application network:
```sh
docker run --rm -it --network test_application busybox
```

Check the container's networking, and verify it is in the same IP range as web2:
```sh
ip addr
```

Ping the web2 container using the IP address:
```sh
ping <WEB2_IP_ADDRESS>
```

Attempt to ping the web2 container by name:
```sh
ping web2
```

Using wget, attempt to access web2 with the hostname:
```sh
wget web2
```

Attempt to ping web1:
```sh
ping <WEB1_IP_ADDRESS>
```

Attempt to access web1 using wget:
```sh
wget <WEB1_IP_ADDRESS>
```

Exit the container:
```sh
exit
```

![image](https://user-images.githubusercontent.com/44756128/114278581-1ea3fc00-99f6-11eb-9c15-5950836cf1df.png)

# Explore the Host Network
Run an httpd container named web3 on the host network:
```sh
docker run -d --name web3 --network host httpd:2.4
```

Check the status of the container:
```sh
docker ps -a
```

Attempt to connect to web3 directly from the server:
```sh
wget localhost
```

Stop web3:
```sh
docker stop web3
```

Attempt to connect to web3 directly from the server again:
```sh
wget localhost
```

Start web3:
```sh
docker start web3
```

Run a container using the busybox image, and see if you can connect to the web3 server:
```sh
docker run --rm -it --network host busybox
ping web3
```

Using wget, attempt to access localhost within the busybox image:
```sh
wget localhost
```

Attempt to ping web2:
```sh
ping <WEB2_IP_ADDRESS>
```

Attempt to ping web1:
```sh
ping <WEB1_IP_ADDRESS>
```

![image](https://user-images.githubusercontent.com/44756128/114278667-7fcbcf80-99f6-11eb-86f7-1dc627d423a0.png)
