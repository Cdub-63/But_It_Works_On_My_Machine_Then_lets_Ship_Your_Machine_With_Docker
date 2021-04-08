![image](https://user-images.githubusercontent.com/44756128/114066136-ffc03100-9860-11eb-94d4-abfbdd33bfaa.png)

# Introduction
Docker volumes is the preferred method of storing container data locally. Volume support is built directly into Docker, making it an easy tool to use for storage, as well as more portable. However, storing container data in Docker volumes still requires you to back up the data in those volumes on your own.

There is another option - storing your container data in the cloud. It's not a solution for every problem, but after this lab, you'll have another tool at your disposal.

This lab will show you how to mount a Cloud Storage bucket onto your local system as a directory. You will then mount that directory into your Docker container. We will use an httpd container, to serve the contents of that bucket as a webpage, but you can use it to share any common data between containers.

This hands-on lab will demonstrate how flexible Docker can be. You can make changes to your bucket and all of your containers using the Cloud Storage bucket will near-instantly have access to the content.

Log in to the server using ssh:
```sh
ssh cloud_user@<PUBLIC_IP_ADDRESS>
```

Note: You do not need to elevate to root user privileges to complete this task.

# Configuration and Installation
Export the projnum variable:
```sh
export projnum=$(curl http://metadata.google.internal/computeMetadata/v1/project/numeric-project-id -sH "Metadata-Flavor: Google")
```

Verify that the variable was set successfully:
```sh
echo $projnum
```

Export the BUCKET variable:
```sh
export BUCKET="widgetfactory-${projnum}"
```

![image](https://user-images.githubusercontent.com/44756128/114066252-22eae080-9861-11eb-9398-ee151252fd5a.png)

# Prepare the Cloud Storage Bucket
Using gsutil, create a new bucket:
```sh
gsutil mb -l us-central1 -c standard gs://$BUCKET
```

Verify that the gcsfuse repo is available on the server:
```sh
cat /etc/yum.repos.d/gcsfuse.repo
```

Install gcsfuse:
```sh
sudo yum install -y gcsfuse
```

![image](https://user-images.githubusercontent.com/44756128/114066482-647b8b80-9861-11eb-8e1f-224b7279eab4.png)

Update the fuse.conf file to allow the user to mount the bucket properly:
```sh
sudo sed -ri 's/# user_allow_other/user_allow_other/' /etc/fuse.conf
```

Configure the directories needed to mount the bucket:
```sh
sudo mkdir /mnt/widget-factory /tmp/gcs
```

Change ownership of the directories to the cloud_user:
```sh
sudo chown cloud_user: /mnt/widget-factory/ /tmp/gcs
```

Mount the bucket:
```sh
gcsfuse -o allow_other --temp-dir=/tmp/gcs $BUCKET /mnt/widget-factory/
```

Copy the website files into the bucket:
```sh
cp -r /home/cloud_user/widget-factory-inc/web/* /mnt/widget-factory/
```

List the contents of the bucket:
```sh
gsutil ls gs://$BUCKET
```

![image](https://user-images.githubusercontent.com/44756128/114066757-ae647180-9861-11eb-862c-c7214a2d0497.png)

# Use the GCS Bucket in a Container
Mount the directory into the Docker container:
```sh
docker run -d --name web1 --mount type=bind,source=/mnt/widget-factory,target=/usr/local/apache2/htdocs,readonly -p 80:80 httpd:2.4
```

![image](https://user-images.githubusercontent.com/44756128/114066846-c6d48c00-9861-11eb-8dcf-de5b53c8694b.png)

Using a web browser, verify connectivity to the container:
```sh
<SERVER_PUBLIC_IP_ADDRESS>
```

![image](https://user-images.githubusercontent.com/44756128/114066940-dbb11f80-9861-11eb-8ae4-4cf12378651a.png)
