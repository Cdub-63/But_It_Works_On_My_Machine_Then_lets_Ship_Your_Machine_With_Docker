![image](https://user-images.githubusercontent.com/44756128/116006820-851e4200-a5d2-11eb-94f6-741ecb548f61.png)

# The Scenario
We manage containers for clients on several Docker servers. But managing the hosts has become a bit of a pain, so we've been looking for an all-in-one tool. We discovered Portainer, and have decided to test it out by deploying it to one of our hosts.

# Create a Volume
```sh
docker volume create portainer_data
```

![image](https://user-images.githubusercontent.com/44756128/116006894-ca427400-a5d2-11eb-8f4c-d88c04f407e5.png)

# Create Portainer
```sh
docker container run -d --name portainer -p 8080:9000 \
--restart=always \
-v /var/run/docker.sock:/var/run/docker.sock \
-v portainer_data:/data portainer/portainer
```

We can run docker container ls to make sure it's running.

![image](https://user-images.githubusercontent.com/44756128/116006926-e9410600-a5d2-11eb-80f5-ba768391712c.png)

# Log into Portainer and Create a Container
Portainer is running at our server's IP address on port 8080, so head there in a browser (http://:8080).

![image](https://user-images.githubusercontent.com/44756128/116006960-1b526800-a5d3-11eb-8316-803cba8097c4.png)

Create your user account and password, and on the next screen click on Local, then Connect. Click on the local in this screen, then Container in the next one.

![image](https://user-images.githubusercontent.com/44756128/116006972-2a391a80-a5d3-11eb-8bf2-a38f85e3226d.png)

![image](https://user-images.githubusercontent.com/44756128/116006989-3b822700-a5d3-11eb-8b39-98f9d7f497ae.png)

Now we can click the Add container button. In the form that follows, use these settings:

Name: lab_nginx

Image nginx:latest

Click the map additional port button, then map port 8081 to 80 on the container. Now click Deploy the container.

![image](https://user-images.githubusercontent.com/44756128/116007062-9025a200-a5d3-11eb-8c1e-3be89327600f.png)

![image](https://user-images.githubusercontent.com/44756128/116007079-9ddb2780-a5d3-11eb-8ec4-d992c0cb815e.png)

# Test Things
If everything is working, we should be able to look at http://:8081 in a web browser.

![image](https://user-images.githubusercontent.com/44756128/116007127-d418a700-a5d3-11eb-9d35-5e99cbec463a.png)
