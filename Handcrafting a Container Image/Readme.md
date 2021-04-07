![image](https://user-images.githubusercontent.com/44756128/113885099-2d37ac80-9785-11eb-8e1a-e03eaab1ab44.png)

# Introduction
If you run your website from a pre-built base image, it will require a manual process to set up the container each time it runs. For repeatability and scalability, the container, and your website code should be made into an image.

We will start with a base webserver image, modify settings in the container for our website, and then create images from the container. We'll demonstrate the importance of small changes to our container, and how they affect our image. Lastly, we will use our new images to create containers to see our hard work in action.

Log in to the server using ssh:
```sh
ssh username@<PUBLIC_IP_ADDRESS>
```

# Get and Run the Base Image
Retrieve the httpd image:
```sh
docker pull httpd:2.4
```

Run the image:
```sh
docker run --name webtemplate -d httpd:2.4
```

Check the status of the webtemplate container:
```sh
docker ps
```

![image](https://user-images.githubusercontent.com/44756128/113881712-4b4fdd80-9782-11eb-8e50-7f62fabcf78d.png)

# Install Tools and Code in the Container
Log in to the container:
```sh
docker exec -it webtemplate bash
```

Run apt update and install git
```sh
apt update && apt install git -y
```

![image](https://user-images.githubusercontent.com/44756128/113882098-a386df80-9782-11eb-94fd-03152fec02ee.png)

Clone the website code from GitHub:
```sh
git clone  https://github.com/linuxacademy/content-widget-factory-inc.git /tmp/widget-factory-inc
```

Verify that the code was cloned successfully:
```sh
ls -l /tmp/widget-factory-inc/
```

List the files in the htdocs/ directory:
```sh
ls -l htdocs/
```

Remove the index.html file:
```sh
rm htdocs/index.html
```

Copy the webcode from /tmp/ to the htdocs/ folder:
```sh
cp -r /tmp/widget-factory-inc/web/* htdocs/
```

Verify that they were copied over successfully:
```sh
ls -l htdocs/
```

Exit the container:
```sh
exit
```

![image](https://user-images.githubusercontent.com/44756128/113882487-f3fe3d00-9782-11eb-8b1a-d9793c0f9a19.png)

# Create an Image from the Container
Copy the Container ID:
```sh
docker ps
```

Create an image from the container:
```sh
docker commit <CONTAINER_ID> example/widgetfactory:v1
```

Verify that the image was created successfully:
```sh
docker images
```

![image](https://user-images.githubusercontent.com/44756128/113882782-34f65180-9783-11eb-8a40-0ce598104696.png)

Take note of the image size.

# Clean up the Template for a Second Version
Log in to the container:
```sh
docker exec -it webtemplate bash
```

Remove the cloned code from the /tmp/ directory:
```sh
rm -rf /tmp/widget-factory-inc/
```

Use apt to uninstall git and clean the cache:
```sh
apt remove git -y && apt autoremove -y && apt clean 
```

Exit the container:
```sh
exit
```

Check the status of the container:
```sh
docker ps
```

Create an image from the updated container:
```sh
docker commit <CONTAINER_ID> example/widgetfactory:v2
```

Verify that both images are now running:
```sh
docker images
```

Delete the v1 image:
```sh
docker rmi example/widgetfactory:v1
```

![image](https://user-images.githubusercontent.com/44756128/113883303-a46c4100-9783-11eb-8b28-1a3ac13e6936.png)

# Run Multiple Containers from the Image
Run multiple containers using the new image:
```sh
docker run -d --name web1 -p 8081:80 example/widgetfactory:v2
docker run -d --name web2 -p 8082:80 example/widgetfactory:v2
docker run -d --name web3 -p 8083:80 example/widgetfactory:v2
```
![image](https://user-images.githubusercontent.com/44756128/113884249-6f142300-9784-11eb-9dba-b152f6b0fad2.png)

![image](https://user-images.githubusercontent.com/44756128/113884302-7a674e80-9784-11eb-92b6-20bf18b2ced6.png)

![image](https://user-images.githubusercontent.com/44756128/113884367-85ba7a00-9784-11eb-858e-bcea5fe349fd.png)

Check the status of the containers:
```sh
docker ps
```

![image](https://user-images.githubusercontent.com/44756128/113884417-90750f00-9784-11eb-966a-98b95acc153b.png)

Stop the base webtemplate image:
```sh
docker stop webtemplate
```

Verify that only the created containers are running:
```sh
docker ps
```

![image](https://user-images.githubusercontent.com/44756128/113884594-b5698200-9784-11eb-9884-438a4424b6fb.png)

Using a web browser, verify that the containers are running successfully:
```sh
<SERVER_PUBLIC_IP_ADDRESS>:8081
<SERVER_PUBLIC_IP_ADDRESS>:8082
<SERVER_PUBLIC_IP_ADDRESS>:8083
```

![image](https://user-images.githubusercontent.com/44756128/113884752-dd58e580-9784-11eb-9606-d4d1c461a0b8.png)

![image](https://user-images.githubusercontent.com/44756128/113884822-eba70180-9784-11eb-8e8f-ddcc006f9dbe.png)

![image](https://user-images.githubusercontent.com/44756128/113884902-fd88a480-9784-11eb-8832-1ed0bb9003c8.png)
