![image](https://user-images.githubusercontent.com/44756128/115281683-e0ea5600-a10e-11eb-927b-76cb1c759096.png)

# Introduction
We will learn how to set up Prometheus and Grafana for alerting and monitoring.

Open your terminal application and log into the server using ssh:
```sh
ssh username@PUBLIC_IP_ADDRESS
```

Then elevate privileges to root.
```sh
sudo su -
```

![image](https://user-images.githubusercontent.com/44756128/115282248-9ddcb280-a10f-11eb-9847-06b9ff23858f.png)

# Using Grafana with Prometheus
Before we get started, let's take a look at our root directory to see what we have.
```sh
ls
```

![image](https://user-images.githubusercontent.com/44756128/115282360-c369bc00-a10f-11eb-9912-c2f4ba1a0fe8.png)

Currently, we have a dashboard, the docker-compose file that was used to set up the environment, and a Prometheus configuration file. Run clear to clear your screen.

# Configure Docker
The first thing we need to do is create a daemon.json file for Docker.
```sh
vi /etc/docker/daemon.json
```


Once /etc/docker/daemon.json is open in the vi text editor, add the following:
```json
{
  "metrics-addr" : "0.0.0.0:9323",
  "experimental" : true
}
```

Then, save and exit the vi editor.
```sh
:wq
```

Restart the Docker service.
```sh
systemctl restart docker
```

Open up port 9323 in the firewall.
```sh
firewall-cmd --zone=public --add-port=9323/tcp
```

![image](https://user-images.githubusercontent.com/44756128/115282642-0b88de80-a110-11eb-8609-75120fb72be4.png)

# Update Prometheus
Next, we're going to edit the prometheus.yml file in the /root directory.
```sh
vi prometheus.yml
```

Change the contents of the file to the following: (Be sure to provide the private IP address of your instance)
```yml
scrape_configs:
  - job_name: prometheus
    scrape_interval: 5s
    static_configs:
    - targets:
      - prometheus:9090
      - node-exporter:9100
      - pushgateway:9091
      - cadvisor:8080

  - job_name: docker
    scrape_interval: 5s
    static_configs:
    - targets:
      - <PRIVATE_IP_ADDRESS>:9323
```

Then, save and quit.
```sh
:wq
```

![image](https://user-images.githubusercontent.com/44756128/115282828-43902180-a110-11eb-8798-5d9e2b44849f.png)

# Update Docker Compose
The next step is to open our docker-compose file and add three new services. Open docker-compose.yml.
```sh
vi ~/docker-compose.yml
```

Change the contents of the file to the following:
```yml
version: '3'
services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
      - 9090:9090
    command:
      - --config.file=/etc/prometheus/prometheus.yml
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml:ro
    depends_on:
      - cadvisor
  cadvisor:
    image: google/cadvisor:latest
    container_name: cadvisor
    ports:
      - 8080:8080
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:rw
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
  pushgateway:
    image: prom/pushgateway
    container_name: pushgateway
    ports:
      - 9091:9091
  node-exporter:
    image: prom/node-exporter:latest
    container_name: node-exporter
    restart: unless-stopped
    expose:
      - 9100
  grafana:
    image: grafana/grafana
    container_name: grafana
    ports:
      - 3000:3000
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=password
    depends_on:
      - prometheus
      - cadvisor
```

Then, save and quit.
```sh
:wq
```

![image](https://user-images.githubusercontent.com/44756128/115283028-7b976480-a110-11eb-9d66-9c91fe36e149.png)

Next, let's apply our changes and rebuild the environment.
```sh
docker-compose up -d
```

Then, let's make sure everything is running.
```sh
docker ps
```

![image](https://user-images.githubusercontent.com/44756128/115283179-a4b7f500-a110-11eb-9016-eb29ad458556.png)

Lastly, let's verify that all of our Prometheus targets are healthy. Open a web browser tab, and go to http://PUBLIC_IP_ADDRESS:9090. Click Status, and select Targets. Everything should be up and running correctly.

![image](https://user-images.githubusercontent.com/44756128/115283283-c5804a80-a110-11eb-9962-b8a89cfefb2c.png)

# Create a New Grafana Data Source
Navigate to Grafana by entering http://PUBLIC_IP_ADDRESS:3000 in the address bar of your browser.

![image](https://user-images.githubusercontent.com/44756128/115283369-e183ec00-a110-11eb-9a48-4993bccb93c2.png)

Type "admin" for username and "password" for password. Click Log In.

![image](https://user-images.githubusercontent.com/44756128/115283432-f496bc00-a110-11eb-9e86-be1d00088dcb.png)

In the Grafana Home Dashboard, click the Add data source icon. For Name, type "Prometheus". Click into the Type field, and select Prometheus from the dropdown. Under URL, select http://localhost:9090. (But we're going to change this in a moment.) Replace "localhost" in the URL with the private IP address.

![image](https://user-images.githubusercontent.com/44756128/115283508-11cb8a80-a111-11eb-92db-1fd3d1b0fb75.png)

![image](https://user-images.githubusercontent.com/44756128/115283646-44758300-a111-11eb-9c42-9d8ceb77fa2f.png)

![image](https://user-images.githubusercontent.com/44756128/115283704-548d6280-a111-11eb-9580-9d1660f1daa9.png)

Click Save & Test.

![image](https://user-images.githubusercontent.com/44756128/115283720-5c4d0700-a111-11eb-8c71-37937b51360d.png)

# Add the Docker Dashboard to Grafana
Click the plus sign (+) on the left side of the Grafana interface, and click Import. Open the provided JSON file. Copy the contents of the file to your clipboard.

![image](https://user-images.githubusercontent.com/44756128/115283769-6a9b2300-a111-11eb-9f76-62d984249cdd.png)

Switch back to Grafana, and paste the text we just copied into the Or paste JSON field. Click Load. On the Import screen, click the dropdown menu for Prometheus, and select the Prometheus data source that we created earlier. Click Import.

We now have our Grafana visualization. In the upper right corner, click on Refresh every 5m and select Last 5 minutes.

Add an Email Notification Channel
In the left sidebar, click the bell icon, and select Notification channels. Click Add channel. For Name, type "Email". In Email addresses, enter your email address. Click Save.

Create an Alert for CPU Usage
Click the dashboard icon in the left sidebar, and select Home. Select the Docker and system monitoring dashboard. Click the CPU Usage graph, and select Edit from the dropdown.

In the Metrics tab, click Add Query. Enter the following:

sum(rate(process_cpu_seconds_total[1m])) * 100
Click the eye icon on the right side of Row E to hide the new rule from view.

Click the Alert tab, then Create Alert. Under Conditions, click into the query field and select E** from the dropdown. For IS ABOVE, type "75". Click **Test Rule.

Click Notifications in the left sidebar, then click the + icon next to Sent to. Select Email Alerts. Next, click into the Message text box, and type "CPU usage is over 75%." Click the save button at the top of the page. In the save popup that opens, check the box next to Save current variables. In the description box, type "Add alerting". Then click Save.
