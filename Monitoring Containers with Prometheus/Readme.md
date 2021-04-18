![image](https://user-images.githubusercontent.com/44756128/115154555-0d37a100-a041-11eb-9333-4dc304c45b89.png)

# Introduction
We'll walk through how to monitor containers using Docker stats, Prometheus, and cAdvisor.

Log into the server using ssh, and then become the root user with a sudo su -.

![image](https://user-images.githubusercontent.com/44756128/115154703-d615bf80-a041-11eb-89bb-d6d87bfdaefc.png)

# Create prometheus.yml
In root's home directory, create prometheus.yml:
```sh
vi prometheus.yml
```

We've got to stick a few configuration lines in here. When we're done, it should look like this:
```yml
scrape_configs:
- job_name: cadvisor
  scrape_interval: 5s
  static_configs:
  - targets:
    - cadvisor:8080
```

![image](https://user-images.githubusercontent.com/44756128/115154730-f9406f00-a041-11eb-9347-a9a6acaa4706.png)

# Create the Prometheus Services
In order to set up Prometheus services, we've got to create configuration files. We're going to make docker-compose.yml, right here in the same directory:
```sh
vi docker-compose.yml
```

It should look like this when we're finished with it:
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
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
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
      - /var/lib/docker:/var/lib/docker:ro
```

![image](https://user-images.githubusercontent.com/44756128/115154761-1bd28800-a042-11eb-9c83-bc63f2120d90.png)

In order to stand up the environment, we'll run this:
```sh
docker-compose up -d
```

And to see if everything stood up properly, let's run a quick docker ps. The output should show four containers: prometheus, cadvisor, nginx, and redis.

![image](https://user-images.githubusercontent.com/44756128/115154790-48869f80-a042-11eb-9d24-8d29904c1aab.png)

# Viewing the Prometheus Web Interface
Let's so see in a web browser as well. Grab the public IP of the server, and browse to it, using the correct port number: http://<IP_ADDRESS>:9090/graph/

![image](https://user-images.githubusercontent.com/44756128/115154824-70760300-a042-11eb-9914-c1ec68bd1da0.png)

We should see a Prometheus page. Click on Status in the top menu, and pick Targets. Now we should land on a screen showing a link for cadvisor.

![image](https://user-images.githubusercontent.com/44756128/115154834-7f5cb580-a042-11eb-97c7-5f466e66bb91.png)

![image](https://user-images.githubusercontent.com/44756128/115154841-8683c380-a042-11eb-9548-e1d549903d6c.png)

Now let's click on Graph (again, in the top menu). There's an Execute button. In the dropdown next to it, find container_startup_time_second. Then click Execute. Just below, we'll see Graph and Console tabs. They've each got the container startup time information we were hunting for. One tab is just prettier than the other.

![image](https://user-images.githubusercontent.com/44756128/115154874-b763f880-a042-11eb-9aeb-00ccf8676102.png)

![image](https://user-images.githubusercontent.com/44756128/115154882-c0ed6080-a042-11eb-9bf6-713625c9fead.png)

Just above that Execute button and related dropdown is a web form field where we can type different expressions. Let's try that now, sticking rate(container_cpu_usage_seconds_total{name="redis"}[1m]) in there. We can do this instead of using the dropdown if we want. If we hit Execute, the information below changes to reflect our new "query."

![image](https://user-images.githubusercontent.com/44756128/115154933-14f84500-a043-11eb-9d89-378a55a91289.png)

![image](https://user-images.githubusercontent.com/44756128/115154951-28a3ab80-a043-11eb-859f-ac61572ff53e.png)

# Investigating cAdvisor
In a browser, navigate to http://<IP_ADDRESS>:8080/containers/. Take a peek around, then change the URL to one of our container names (like nginx) so we're at http://<IP_ADDRESS>:8080/docker/nginx/.

![image](https://user-images.githubusercontent.com/44756128/115154990-5f79c180-a043-11eb-952a-70f697c02b99.png)

# Stats in Docker
If we run docker stats, we're going to get some output that looks a lot like docker ps, but this stays open and reports what's going on as far as the various aspects (CPU and memory usage, etc.) of our containers.

![image](https://user-images.githubusercontent.com/44756128/115155014-78827280-a043-11eb-886b-3f75fe5355f3.png)

To get out of this, hit Ctrl+C

# Prettier Stats
We can format the output of docker stats differently if we want. And, if we really want to get fancy, we can do it in a shell script. Let's do it.

Create a stats.sh file in /root:
```sh
vi ~/stats.sh
```

Now let's fill it with this command:
```sh
docker stats --format "table {{.Name}} {{.ID}} {{.MemUsage}} {{.CPUPerc}}"
```

![image](https://user-images.githubusercontent.com/44756128/115155060-c13a2b80-a043-11eb-9a1a-fd8a741365cb.png)

Once we've made sure the file is executable (chmod a+x stats.sh) we can run it with this:
```sh
./stats.sh
```

![image](https://user-images.githubusercontent.com/44756128/115155050-aebff200-a043-11eb-8c45-ddd8c47a8262.png)

Just like before, when we get bored watching stats, we can exit by pressing Ctrl+C
