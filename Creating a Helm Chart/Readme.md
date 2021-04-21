![image](https://user-images.githubusercontent.com/44756128/115565810-5117e980-a27f-11eb-8eb0-a276cb6060a9.png)

# Install Helm
Use curl to create a local copy of the Helm install script:
```sh
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get > /tmp/get_helm.sh
```

Verify this worked by running cat on the file we just downloaded:
```sh
cat /tmp/get_helm.sh
```

![image](https://user-images.githubusercontent.com/44756128/115566778-2c704180-a280-11eb-8c32-3d9859798568.png)

You should see a long output that indicates the file was downloaded correctly.

Use chmod to modify access permissions for the install script:
```sh
chmod 700 /tmp/get_helm.sh
```

Set the version to v2.8.2:
```sh
DESIRED_VERSION=v2.8.2 /tmp/get_helm.sh
```

Ensure Helm uses the correct stable chart repo (the default one used by Helm has been decommissioned):
```sh
helm init --stable-repo-url https://charts.helm.sh/stable
```

Initialize Helm:
```sh
helm init --wait
```

Give Helm the permissions it needs to work with Kubernetes:
```sh
kubectl --namespace=kube-system create clusterrolebinding add-on-cluster-admin --clusterrole=cluster-admin --serviceaccount=kube-system:default
```

Make sure our configuration is working properly:
```sh
helm ls
```

![image](https://user-images.githubusercontent.com/44756128/115566999-66d9de80-a280-11eb-8f40-79c662c478c6.png)

We should get no error response, indicating there are no issues.

# Create a Helm Chart
Create a directory called charts:
```sh
mkdir charts
```

Change directory to charts:
```sh
cd charts
```

Create the chart for httpd:
```sh
helm create httpd
```

Verify our directory was created correctly:
```sh
ls
```

Navigate to the httpd directory:
```sh
cd httpd/
```

View the files in the directory:
```sh
ls
```

This directory contains two files: Chart.yaml and values.yaml. We need to edit the values.yaml file.

![image](https://user-images.githubusercontent.com/44756128/115567449-e36cbd00-a280-11eb-83ae-a8b08a35d328.png)

Open values.yaml:
```sh
vi values.yaml
```

Under image, change the repository to httpd.

Change the tag to latest.

Under service, change type to NodePort.

When you're finished, httpd/values.yaml should look like this:
```yml
replicaCount: 1

image:
  repository: httpd
  tag: latest
  pullPolicy: IfNotPresent

service:
  type: NodePort
  port: 80

ingress:
  enabled: false
  annotations: {}
  path: /
  hosts:
    - chart-example.local
  tls: []

resources: {}

nodeSelector: {}

tolerations: []

affinity: {}
```

![image](https://user-images.githubusercontent.com/44756128/115567507-f1bad900-a280-11eb-885a-b6978a5d0d1b.png)

Save and exit the file by pressing Escape followed by :wq.

# Create Your Application Using Helm
Back out of the httpd directory:
```sh
cd ../
```

Back in the charts directory, install our application:
```sh
helm install --name my-httpd ./httpd/
```

![image](https://user-images.githubusercontent.com/44756128/115567633-1616b580-a281-11eb-8747-daada623380a.png)

Copy the commands listed under the NOTES section of the output, and then paste and run them. It should return the private IP address and port number of our application.

![image](https://user-images.githubusercontent.com/44756128/115567701-27f85880-a281-11eb-9539-74da1ec2fe26.png)

Let's check to see if our pods have come online:
```sh
kubectl get pods
```

The output shows us the pod that was created by Helm.

List the services:
```sh
kubectl get services
```

![image](https://user-images.githubusercontent.com/44756128/115567840-4eb68f00-a281-11eb-8c5c-128e7c78a434.png)

Copy the public IP address of the server.

In a new browser tab, paste in the public IP address. Add a colon (:) after it. Copy the port number of my-httpd from your terminal output (the number after 80: and before the /TCP), and paste it after the colon in the address bar of your browser tab.

Press Enter to load the page. If everything is working correctly, you should see a success message that says: "It works!"

![image](https://user-images.githubusercontent.com/44756128/115567991-773e8900-a281-11eb-9c70-2d45e573bc10.png)
