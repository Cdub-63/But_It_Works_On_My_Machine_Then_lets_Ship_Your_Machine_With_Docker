![image](https://user-images.githubusercontent.com/44756128/115486557-87704d00-a21c-11eb-8b56-e3fc0daac9af.png)


Introduction
Docker works well for running single containers or small groups of containers on a single host. But what if you have more than a few containers, or want those containers to be spread across multiple servers, or even made highly available?

For these scenarios, you'll need a container orchestrator. Kubernetes is a popular container orchestration framework that easily works with Docker. We'll learn to set up a Kubernetes cluster with Docker.

# Configure the Kubernetes Cluster
On all three nodes, add the Kubernetes repo to /etc/yum.repos.d:
```sh
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
```

Install the repo:
```sh
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kube*
EOF
```

Disable SELinux:
```sh
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
```

Install Kubernetes:
```sh
sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
```

![image](https://user-images.githubusercontent.com/44756128/115487202-d23e9480-a21d-11eb-9ced-2f69bdac3bcb.png)

Enable and start kubelet:
```sh
sudo systemctl enable --now kubelet
```

![image](https://user-images.githubusercontent.com/44756128/115487268-f601da80-a21d-11eb-96fb-4a807a557c1e.png)

From Node 1, initialize the controller node, and set the code network CIDR to 10.244.0.0/16:
```sh
kubeadm init --pod-network-cidr=10.244.0.0/16
```

![image](https://user-images.githubusercontent.com/44756128/115487466-5db82580-a21e-11eb-8ff7-56e35d18bfa7.png)

From Node 1, check the status of your cluster:
```sh
docker ps -a
```

![image](https://user-images.githubusercontent.com/44756128/115487488-69a3e780-a21e-11eb-9e2d-aa465949b6b5.png)

Repeat this step on the worker nodes. Can the worker nodes see the cluster?

![image](https://user-images.githubusercontent.com/44756128/115487516-79233080-a21e-11eb-833f-3420dbe70b82.png)

Run the following commands to start using the cluster:
```sh
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

![image](https://user-images.githubusercontent.com/44756128/115487563-93f5a500-a21e-11eb-9a52-b1fed93c9f0e.png)

Copy the kubeadm join command, then paste and run it in your Node 2, and Node 3 terminal windows.

![image](https://user-images.githubusercontent.com/44756128/115487601-a5d74800-a21e-11eb-971b-690f6e38d180.png)

From the worker nodes, verify that they can see the cluster:
```sh
docker ps -a
```

![image](https://user-images.githubusercontent.com/44756128/115487613-b1c30a00-a21e-11eb-8e2c-0978fa92ed88.png)

From Node 1, check the status of the nodes:
```sh
kubectl get nodes
```

Install Flannel:
```sh
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

# Create a Pod
Create the pod.yml file:
```sh
vim pod.yml
```

![image](https://user-images.githubusercontent.com/44756128/115487738-e8992000-a21e-11eb-9c2d-880431af3922.png)

In the file, paste the following:
```yml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod-demo
  labels:
    app: nginx-demo
spec:
  containers:
  - image: nginx:latest
    name: nginx-demo
    ports:
    -  containerPort: 80
    imagePullPolicy: Always
```

![image](https://user-images.githubusercontent.com/44756128/115487767-f5b60f00-a21e-11eb-8d98-2354431c25e5.png)

Save the file:
```sh
ESC
:wq
```

Create the pod:
```sh
kubectl create -f pod.yml
```

Check the status of the pod:
```sh
kubectl get pods
```

![image](https://user-images.githubusercontent.com/44756128/115487821-14b4a100-a21f-11eb-8394-e8d22734caf0.png)

# Create the Service
Create the service.yml file:
```sh
vim service.yml
```

![image](https://user-images.githubusercontent.com/44756128/115487888-31e96f80-a21f-11eb-88a6-604622cc22f0.png)

In the file, paste the following:
```yml
apiVersion: v1
kind: Service
metadata:
  name: service-demo
spec:
  selector:
    app: nginx-demo
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: NodePort
```

![image](https://user-images.githubusercontent.com/44756128/115487936-3f9ef500-a21f-11eb-92f7-0a17a20e6c89.png)

Save the file:
```sh
ESC
:wq
```

Create the service:
```sh
kubectl apply -f service.yml
```

Run the following command to view the service:
```sh
kubectl get services
```

![image](https://user-images.githubusercontent.com/44756128/115487967-58a7a600-a21f-11eb-8fad-df4e3127d3fd.png)

Take note of the service-demo port number.

In a web browser, navigate to the public IP address for a server in the cluster, and verify connectivity:
```sh
<PUBLIC_IP_ADDRESS>:<SERVICE_DEMO_PORT_NUMBER>
```

![image](https://user-images.githubusercontent.com/44756128/115488048-7f65dc80-a21f-11eb-9e99-9c70bbcb024c.png)
