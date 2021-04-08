![image](https://user-images.githubusercontent.com/44756128/114060727-49a61880-985b-11eb-89fa-9f20014a8b27.png)

# Introduction
Using Docker volumes is the preferred method of storing container data locally. Volume support is built directly into Docker, making it an easy tool to use for storage, as well as more portable. However, storing container data in Docker volumes still requires you to back up the data in those volumes on your own.

There is another option - storing your container data in the cloud. It's not a solution for every problem, but after this lab, you'll have another tool at your disposal.

This lab will show you how to mount an S3 bucket onto your local system as a directory. You will then mount that directory into your Docker container. We will use an httpd container to serve the contents of that bucket as a webpage, but you can use it to share any common data between containers.

This will demonstrate how flexible Docker can be. You can make changes to your bucket and all of your containers using the S3 bucket will near-instantly have access to the content.

Log in to the server using ssh:
```sh
ssh username@<PUBLIC_IP_ADDRESS>
```

# Configuration and Installation
Install the awscli, while checking if there are any versions currently installed, and not stopping any user processes:
```sh
pip install --upgrade --user awscli
```

![image](https://user-images.githubusercontent.com/44756128/114061775-6b53cf80-985c-11eb-8f50-89511f02977c.png)

Configure the CLI:
```sh
aws configure
```

Enter the following:
  - AWS Access Key ID:
  - AWS Secret Access Key:
  - Default region name: us-east-1
  - Default output format: json

![image](https://user-images.githubusercontent.com/44756128/114062022-a81fc680-985c-11eb-934e-f0581c1e30d9.png)

Copy the CLI configuration to the root user:
```sh
sudo cp -r ~/.aws /root
```

Install the s3fs package:
```sh
sudo yum install s3fs-fuse -y
```

![image](https://user-images.githubusercontent.com/44756128/114062198-ddc4af80-985c-11eb-9731-72b298a38619.png)

# Prepare the Bucket
Create a mount point for the s3 bucket:
```sh
sudo mkdir /mnt/widget-factory
```

Export the bucket name:
```sh
export BUCKET=<S3_BUCKET_NAME>
```

Mount the S3 bucket:
```sh
sudo s3fs $BUCKET /mnt/widget-factory -o allow_other -o default_acl=public-read -o use_cache=/tmp/s3fs
```

Verify that the bucket was mounted successfully:
```sh
ll /mnt/widget-factory
```

Copy the website files to the s3 bucket:
```sh
cp -r ~/widget-factory-inc/web/* /mnt/widget-factory
```

Verify the files are in the folder:
```sh
ll /mnt/widget-factory
```

Verify the files are in the s3 bucket:
```sh
aws s3 ls s3://$BUCKET
```

![image](https://user-images.githubusercontent.com/44756128/114062547-41e77380-985d-11eb-8fba-37dbe8d6f6a0.png)

# Use the S3 Bucket Files in a Docker Container
Run an httpd container using the S3 bucket:
```sh
docker run -d --name web1 -p 80:80 --mount type=bind,source=/mnt/widget-factory,target=/usr/local/apache2/htdocs,readonly httpd:2.4
```

![image](https://user-images.githubusercontent.com/44756128/114063195-f84b5880-985d-11eb-8d9a-19b1faacf43f.png)

In a web browser, verify connectivity to the container:
```sh
<SERVER_PUBLIC_IP_ADDRESS>
```

![image](https://user-images.githubusercontent.com/44756128/114062719-70fde500-985d-11eb-9bd4-a4968203ff03.png)

Check the bucket cache:
```sh
ll /tmp/s3fs/<S3_BUCKET_NAME>
```

Change to the /mnt/widget-factory/ directory:
```sh
cd /mnt/widget-factory
```

Create a new page within the bucket:
```sh
cp index.html newPage.html
```

![image](https://user-images.githubusercontent.com/44756128/114063151-ec5f9680-985d-11eb-8f58-41c06fa7c27a.png)

In a web browser, verify that the new page is accessible:
```sh
<SERVER_PUBLIC_IP_ADDRESS>/newPage.html
```

![image](https://user-images.githubusercontent.com/44756128/114062957-b7ebda80-985d-11eb-84b1-38088078c790.png)

Verify that the page was added to the bucket:
```sh
aws s3 ls $BUCKET
```

![image](https://user-images.githubusercontent.com/44756128/114063091-d8b43000-985d-11eb-9a94-81f4f7da9ebb.png)
