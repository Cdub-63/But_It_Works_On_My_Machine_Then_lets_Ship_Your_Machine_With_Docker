![image](https://user-images.githubusercontent.com/44756128/115561673-69860500-a27b-11eb-9174-f5539efd4de1.png)

# Complete the Kubernetes cluster
Initialize the cluster:
```sh
kubeadm init --pod-network-cidr=10.244.0.0/16 --kubernetes-version=v1.11.3

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

![image](https://user-images.githubusercontent.com/44756128/115563230-ee255300-a27c-11eb-9149-f389b3cdd558.png)

![image](https://user-images.githubusercontent.com/44756128/115563280-faa9ab80-a27c-11eb-855e-9ce79f92023b.png)

![image](https://user-images.githubusercontent.com/44756128/115563339-0a28f480-a27d-11eb-9fe5-35cfc71e3378.png)

![image](https://user-images.githubusercontent.com/44756128/115563410-20cf4b80-a27d-11eb-8402-8255c48048ac.png)

![image](https://user-images.githubusercontent.com/44756128/115563459-30e72b00-a27d-11eb-9b70-c025164c4a55.png)

Install Flannel on the master:
```sh
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/v0.9.1/Documentation/kube-flannel.yml
```

![image](https://user-images.githubusercontent.com/44756128/115563680-6ab83180-a27d-11eb-98d5-b0ed085f61e6.png)

# Create the deployment
```sh
vi deployment.yml
```

Add the following to deployment.yml:
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpd-deployment
  labels:
    app: httpd
spec:
  replicas: 3
  selector:
    matchLabels:
      app: httpd
  template:
    metadata:
      labels:
        app: httpd
    spec:
      containers:
      - name: httpd
        image: httpd:latest
        ports:
        - containerPort: 80
```

![image](https://user-images.githubusercontent.com/44756128/115563864-91766800-a27d-11eb-9474-80c7bcdc5851.png)

Spin up the deployment:
```sh
kubectl create -f deployment.yml
```

![image](https://user-images.githubusercontent.com/44756128/115564116-c84c7e00-a27d-11eb-8bd1-a1d32312d827.png)

# Create the service
```sh
vi service.yml
```

Add the following to service.yml:
```yml
kind: Service
apiVersion: v1
metadata:
  name: service-deployment
spec:
  selector:
    app: httpd
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: NodePort
```

![image](https://user-images.githubusercontent.com/44756128/115564216-e5814c80-a27d-11eb-8cea-d9938db75662.png)

Create the service:
```sh
kubectl create -f service.yml
```

![image](https://user-images.githubusercontent.com/44756128/115564441-13669100-a27e-11eb-8711-b53ce2fdaf2d.png)

![image](https://user-images.githubusercontent.com/44756128/115564560-2da06f00-a27e-11eb-8d3c-efff01b5e4bf.png)

# Scale the deployment up to 5 replicas
```sh
vi deployment.yml
```

Change the number of replicas to 5:
```yml
spec:
  replicas: 5
```

![image](https://user-images.githubusercontent.com/44756128/115564663-4b6dd400-a27e-11eb-9cf5-24e7e57c94a7.png)

Apply the changes:
```sh
kubectl apply -f deployment.yml
```

![image](https://user-images.githubusercontent.com/44756128/115564767-66404880-a27e-11eb-87f5-089a07e7746b.png)

# Scale the deployment down to 2 replicas
```sh
vi deployment.yml
```

Change the number of replicas to 2:
```yml
spec:
  replicas: 2
```

![image](https://user-images.githubusercontent.com/44756128/115564850-7a844580-a27e-11eb-8d52-c68f9f4aa3cc.png)

Apply the changes:
```sh
kubectl apply -f deployment.yml
```
![image](https://user-images.githubusercontent.com/44756128/115564960-91c33300-a27e-11eb-9cbf-62a722a7515a.png)
