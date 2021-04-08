![image](https://user-images.githubusercontent.com/44756128/114068760-d94fc500-9863-11eb-84b2-470b9f53f056.png)

# Introduction
Using Docker volumes is the preferred method of locally storing container data. Volume support is built directly into Docker, making it an easy tool to use for storage, as well as more portable. However, storing container data in Docker volumes still requires us to back up the data in those volumes on our own. There is another option: storing our container data in the Cloud. It's not a solution for every problem, but this demonstrates another tool at our disposal.

This shows how to mount a Blob Storage container onto our local system as a directory. We will then mount that directory into our Docker container. We will use an httpd container to serve the contents of that bucket as a webpage, but we can use it to share any common data between containers. This will demonstrate how flexible Docker can be. We can make changes to our bucket, and all our containers using the Blob Storage container will near-instantly have access to the content.

# Note
While this method works for static content or content that changes occasionally, this should not be used for frequently written files, such as a database. That would drastically increase latency and drive up your cloud storage costs. You should enable local caching and mount the data as read-only in the container to help mitigate these issues. However, you can use this method for data output if the container processes a job that emits uniquely-named output.

Log in to the server using ssh:
```sh
ssh username@<PUBLIC_IP_ADDRESS>
```

# Configuration and Installation
Obtain the Azure login credentials:
```sh
az login
```

Copy the code provided by the command.

Open a browser and navigate to https://microsoft.com/devicelogin.

Enter the code copied in a previous step and click Next.

Use your Azure login credentials to finish logging in.

Switch back to the terminal and wait for the confirmation.

![image](https://user-images.githubusercontent.com/44756128/114069268-6c88fa80-9864-11eb-9478-d5ab58d16d9b.png)

![image](https://user-images.githubusercontent.com/44756128/114070689-deae0f00-9865-11eb-8e35-6cfbc9b0da8c.png)

## Prepare the Storage
Find the name of the Storage account:
```sh
az storage account list | grep name | head -1
```

![image](https://user-images.githubusercontent.com/44756128/114070776-fb4a4700-9865-11eb-9969-21ccc67c6dec.png)

Copy the name of the Storage account to the clipboard.

Export the Storage account name:
```sh
export AZURE_STORAGE_ACCOUNT=<COPIED_STORAGE_ACCOUNT_NAME>
```

Retrieve the Storage access key:
```sh
az storage account keys list --account-name=$AZURE_STORAGE_ACCOUNT
```

Copy the key1 "value" for later use.

Export the key value:
```sh
export AZURE_STORAGE_ACCESS_KEY=<KEY1_VALUE>
```

![image](https://user-images.githubusercontent.com/44756128/114071100-5c721a80-9866-11eb-9a68-db2c4713c927.png)

Install blobfuse:
```sh
sudo rpm -Uvh https://packages.microsoft.com/config/rhel/7/packages-microsoft-prod.rpm
sudo yum install blobfuse fuse -y
```

![image](https://user-images.githubusercontent.com/44756128/114071282-8deae600-9866-11eb-8907-54a971072ad0.png)

Modify the fuse.conf configuration file:
```sh
sudo sed -ri 's/# user_allow_other/user_allow_other/' /etc/fuse.conf
```

![image](https://user-images.githubusercontent.com/44756128/114071354-a2c77980-9866-11eb-81c6-fc1327c8c04c.png)

## Use the Azure Blob Storage Container
Create necessary directories:
```sh
sudo mkdir -p /mnt/widget-factory /mnt/blobfusetmp
```

Change ownership of the directories:
```sh
sudo chown cloud_user /mnt/widget-factory/ /mnt/blobfusetmp/
```

Mount the Blob Storage from Azure:
```sh
blobfuse /mnt/widget-factory --container-name=website --tmp-path=/mnt/blobfusetmp -o allow_other
```

Copy website files into the Blob Storage container:
```sh
cp -r ~/widget-factory-inc/web/* /mnt/widget-factory/
```

Verify the copy worked:
```sh
ll /mnt/widget-factory/
```

Verify the files made it to Azure Blob Storage:
```sh
az storage blob list -c website --output table
```

Run a Docker container:
```sh
docker run -d --name web1 -p 80:80 --mount type=bind,source=/mnt/widget-factory,target=/usr/local/apache2/htdocs,readonly httpd:2.4
```

![image](https://user-images.githubusercontent.com/44756128/114071542-d904f900-9866-11eb-9be5-f97822d8064c.png)

Once the command is complete, open a web browser and navigate to the public IP address of the server.

Verify the website is up and running.

![image](https://user-images.githubusercontent.com/44756128/114071674-ff2a9900-9866-11eb-9ea9-1c777d0fd5b1.png)
