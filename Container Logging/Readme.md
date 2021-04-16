![image](https://user-images.githubusercontent.com/44756128/115050939-e2fbad00-9ea1-11eb-948a-952b324dfd43.png)

# Introduction
We are going to configure syslog on a Docker instance, configure Docker to use syslog instead of the JSON logging driver, and spin up two containers to test our configuration.

Open your terminal application and loginto your server via SSH:
```sh
ssh username@<PUBLIC_IP>
```

Next, elevate privileges to root.
```sh
sudo su -
```

# Configure Docker to Use Syslog
Open the rsyslog.conf file.
```sh
vim /etc/rsyslog.conf
```

![image](https://user-images.githubusercontent.com/44756128/115055239-d9287880-9ea6-11eb-948f-2e824379ef74.png)

In the file editor, uncomment the two lines under `Provides UDP syslog reception` by removing `#`.

![image](https://user-images.githubusercontent.com/44756128/115055185-c7df6c00-9ea6-11eb-9db5-737431c83cff.png)

```sh
$ModLoad imudp
$UDPServerRun 514
```

Then, start the syslog service.
```sh
systemctl start rsyslog
```

Now that syslog is running, let's configure Docker to use syslog as the default logging driver. We'll do this by creating a file called daemon.json.
```sh
sudo mkdir /etc/docker
vi /etc/docker/daemon.json
```

![image](https://user-images.githubusercontent.com/44756128/115055430-083eea00-9ea7-11eb-850e-2dd6db871423.png)

In the vi editor, enter the following, making sure to replace <PRIVATE_IP> with the private IP of your server:
```sh
{
  "log-driver": "syslog",
  "log-opts": {
    "syslog-address": "udp://<PRIVATE_IP>:514"
  }
}
```

![image](https://user-images.githubusercontent.com/44756128/115055538-2b699980-9ea7-11eb-934c-31f1217257bc.png)

![image](https://user-images.githubusercontent.com/44756128/115056327-4be62380-9ea8-11eb-829c-c865bf6999c6.png)

Next, save and quit. Then, start the Docker service.
```sh
systemctl start docker
```

The next step is to see if there are any logs coming in from Docker.
```sh
tail /var/log/messages
```

![image](https://user-images.githubusercontent.com/44756128/115056304-438de880-9ea8-11eb-9b74-a9c16b81544f.png)

The output of this command tells us that there are. Now let's test our setup.

We're going to create two new containers using the httpd image. The first one will be called syslog-logging and will use syslog for the log driver. The second will be called json-logging and will use the JSON file for the log driver.

Let's create our first container.
```sh
docker container run -d --name syslog-logging httpd
```

Then run docker ps to make sure our container is up and running correctly.

Next, execute the docker logs command to see what logs we have.
```sh
docker logs syslog-logging
```

When we run this, we receive an error that says Error response from daemon: configured logging does not support reading. This is because we're using syslog instead of JSON for logging.

To confirm this, we can check the content of /var/log/messages. Verify that the syslog-logging container is sending its logs to syslog by running tail on the message log file.
```sh
tail /var/log/messages
```

The output shows us the logs that are being input to syslog.

Now let's create our second test container. This time, we'll specify the log driver as the JSON file.
```sh
docker container run -d --name json-logging --log-driver json-file httpd
```

Run the docker ps command again. This time, we should see two containers running.

Next, verify that the json-logging container is sending its logs to the JSON file. Execute the docker logs command on the json-logging container.
```sh
docker logs json-logging
```

![image](https://user-images.githubusercontent.com/44756128/115056611-b0a17e00-9ea8-11eb-9c55-825e462335d0.png)

This time, the logs do not appear in /var/log/messages because they are being sent to a JSON file instead.
