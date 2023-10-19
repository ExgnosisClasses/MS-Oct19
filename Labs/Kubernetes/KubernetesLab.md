# Microservice Architecture

---

## Kubernetes Lab
Last update: October 16, 2023

## Part One - Logging in

Use the login process described in the login document to access the Ubuntu machine that is running kubernetes. Once you are logged in, clone the lab repository to the Ubuntu machine using 

```bash
ubuntu@ip-172-31-15-199:~$ git clone https://github.com/ExgnosisClasses/MS-Labs-Oct18.git
```

## Part Two - Start Minikube

You will be using the minikube environment to simulate using a kubernetes cluster. To start minikube enter the command _minikube start_ at the command line as shown. It may take several minutes for the cluster to boot up.

```bash
ubuntu@ip-172-31-15-199:~$ minikube start
😄  minikube v1.30.1 on Ubuntu 20.04
✨  Using the docker driver based on existing profile
👍  Starting control plane node minikube in cluster minikube
🚜  Pulling base image ...
🤷  docker "minikube" container is missing, will recreate.
🔥  Creating docker container (CPUs=2, Memory=2200MB) ...
🐳  Preparing Kubernetes v1.26.3 on Docker 23.0.2 ...
🔗  Configuring bridge CNI (Container Networking Interface) ...
🔎  Verifying Kubernetes components...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Enabled addons: storage-provisioner, default-storageclass

🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```
Now get some info about the cluster and nodes. First, get the cluster info

```bash
ubuntu@ip-172-31-15-199:~$ kubectl cluster-info
Kubernetes control plane is running at https://192.168.49.2:8443
CoreDNS is running at https://192.168.49.2:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```
Next get the information about the nodes that are running. Because this is minikube, there will only be one node

```bash
buntu@ip-172-31-15-199:~$ kubectl get -o wide nodes
NAME       STATUS   ROLES           AGE   VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION    CONTAINER-RUNTIME
minikube   Ready    control-plane   40m   v1.26.3   192.168.49.2   <none>        Ubuntu 20.04.5 LTS   5.11.0-1027-aws   docker://23.0.2
```

And finally, get the running pods. Notice that if you don't specify the `--all-namespaces` option, kubernetes will only list your pods. The pods that you see are the running Kubernetes services in the control plane.

```bash
ubuntu@ip-172-31-15-199:~$ kubectl get pods -o wide  --all-namespaces
NAMESPACE     NAME                               READY   STATUS    RESTARTS        AGE   IP             NODE       NOMINATED NODE   READINESS GATES
kube-system   coredns-787d4945fb-gb7qb           1/1     Running   2 (3m50s ago)   41m   10.244.0.3     minikube   <none>           <none>
kube-system   etcd-minikube                      1/1     Running   1 (22m ago)     42m   192.168.49.2   minikube   <none>           <none>
kube-system   kube-apiserver-minikube            1/1     Running   1 (22m ago)     42m   192.168.49.2   minikube   <none>           <none>
kube-system   kube-controller-manager-minikube   1/1     Running   1 (22m ago)     42m   192.168.49.2   minikube   <none>           <none>
kube-system   kube-proxy-9rqgp                   1/1     Running   1 (22m ago)     41m   192.168.49.2   minikube   <none>           <none>
kube-system   kube-scheduler-minikube            1/1     Running   1 (22m ago)     42m   192.168.49.2   minikube   <none>           <none>
kube-system   storage-provisioner                1/1     Running   3 (3m33s ago)   42m   192.168.49.2   minikube   <none>           <none>
```

## Part Three - Deploying a Pod

In this section, you will deploy a simple pod using a yaml specification file.

The file in the repo is `mypod.yaml` and looks like this.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: alpine
    image: alpine:3.9
    command:
    - "sleep"
    - "3600"
```
The pod uses the alpine image discussed in the docker section and just has the image running without actually doing anything. Generally, pods are set to restart so we may have to kill the pod to get it to stop.

We can start the pod directly at the command line by using the `create` command, but the preferred way is to provide a configuration file that describes what we want kubernetes to do for us. This is the `mypod.yaml` file above.

Start the pod using the command:

```bash
ubuntu@ip-172-31-15-199:~$ kubectl apply -f mypod.yaml
pod/mypod created

ubuntu@ip-172-31-15-199:~$ kubectl get pods -o wide 
NAME    READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES
mypod   1/1     Running   0          24s   10.244.0.5   minikube   <none>           <none>
```

To see detailed information about the pod you just started use the describe command. Inspect the output noting the IP address and other details about the pod. Note that at the end of listing is a history of events related to that pod.

```bash
ubuntu@ip-172-31-15-199:~$ kubectl describe pod mypod 
## Lots of stuff omitted

Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  46s   default-scheduler  Successfully assigned default/mypod to minikube
  Normal  Pulled     45s   kubelet            Container image "alpine:3.9" already present on machine
  Normal  Created    45s   kubelet            Created container alpine
  Normal  Started    45s   kubelet            Started container alpine

```

To stop the pod, use the `delete` command. After you delete the pod, it will take some time to actually remove it from the kubernetes environment. You can see this process by using the `get pods` command.

```bash
ubuntu@ip-172-31-15-199:~$ kubectl delete pod mypod
pod "mypod" deleted

ubuntu@ip-172-31-15-199:~$ kubectl get pods -o wide 
NAME    READY   STATUS        RESTARTS   AGE     IP           NODE       NOMINATED NODE   READINESS GATES
mypod   1/1     Terminating   0          5m18s   10.244.0.5   minikube   <none>           <none>

ubuntu@ip-172-31-15-199:~$ kubectl get pods -o wide 
No resources found in default namespace.


```

## Part Four - Running a Replica Set Deployment

In this section, you will create a replica set deployment of three pods, scale it up and down and update the running image.

### 1. Create the deployment

The deployment yaml file looks like this:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

Start the deployment by running the specification file. Then check to see that the deployment is running.

```bash
ubuntu@ip-172-31-15-199:~$ kubectl apply -f deployment.yaml
deployment.apps/nginx-deployment created

ubuntu@ip-172-31-15-199:~$ kubectl get -o wide deployments 
NAME               READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES         SELECTOR
nginx-deployment   3/3     3            3           44s   nginx        nginx:1.14.2   app=nginx
```

List the pods. Notice that each replica has a unique name generated by Kubernetes.

```bash
ubuntu@ip-172-31-15-199:~$ kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-85996f8dbd-dhn9x   1/1     Running   0          3m43s
nginx-deployment-85996f8dbd-ghwl8   1/1     Running   0          3m43s
nginx-deployment-85996f8dbd-zsb5q   1/1     Running   0          3m43s

```

Now destroy one of the pods and note that Kubernetes will automatically create a replacement pod to ensure that there are three replicas running. Use one of the pod names from your listing.

```bash
ubuntu@ip-172-31-15-199:~$ kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-85996f8dbd-dhn9x   1/1     Running   0          8m58s
nginx-deployment-85996f8dbd-ghwl8   1/1     Running   0          8m58s
nginx-deployment-85996f8dbd-zsb5q   1/1     Running   0          8m58s

ubuntu@ip-172-31-15-199:~$ kubectl delete pod nginx-deployment-85996f8dbd-dhn9x
pod "nginx-deployment-85996f8dbd-dhn9x" deleted

ubuntu@ip-172-31-15-199:~$ kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-85996f8dbd-ghwl8   1/1     Running   0          9m35s
nginx-deployment-85996f8dbd-j4vp8   1/1     Running   0          11s
nginx-deployment-85996f8dbd-zsb5q   1/1     Running   0          9m35s
ubuntu@ip-172-31-15-199:~$ 
```

In this example, we deleted pod `nginx-deployment-85996f8dbd-dhn9x` and then Kubernetes created a new pod `nginx-deployment-85996f8dbd-j4vp8 `

### 2. Scaling the Deployment

We can scale up or scale down the number of pods being deployed just by changing the specification in the deployment file. For example, if we want to scale up to five, we just modify the spec to look like this:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 5
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.14.2
          ports:
            - containerPort: 80
```

This in the lab repo as `deployment-scaled.yaml`

Run the new deployment specification and check to see that the deployment now has five pods running. Note the age of each pod which will tell you which are newly added ones.

```bash
buntu@ip-172-31-15-199:~$ kubectl apply -f deployment-scaled.yaml
deployment.apps/nginx-deployment configured

ubuntu@ip-172-31-15-199:~$ kubectl get deployment nginx-deployment
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   5/5     5            5           20m

ubuntu@ip-172-31-15-199:~$ kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-85996f8dbd-67fw5   1/1     Running   0          64s
nginx-deployment-85996f8dbd-ghwl8   1/1     Running   0          21m
nginx-deployment-85996f8dbd-j4vp8   1/1     Running   0          11m
nginx-deployment-85996f8dbd-rj2jv   1/1     Running   0          64s
nginx-deployment-85996f8dbd-zsb5q   1/1     Running   0          21m
```

## 3. Updating the Images

Now that we have a deployment, we can do a rolling update to change the containers that are running in the pods. In the file `deployment-update.yaml` we change the version of nginx from 1.14.2 to 1.25.2. While doing the update, Kubernetes will spin up a new pod with the new version, wait until it is operational, then spin down one of the older pods. This ensures that there are at least five pods running at all times, which is what the replica set specifices.

The modified yaml file looks like this:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 5
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.25.2
          ports:
            - containerPort: 80
```

First check the version being run by the current deployment.

```bash
ubuntu@ip-172-31-15-199:~$ kubectl get deployment nginx-deployment -o wide
NAME               READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES         SELECTOR
nginx-deployment   5/5     5            5           50m   nginx        nginx:1.14.2   app=nginx
```
Now apply the modified specification. Check the pods to see that they are all recent and look at the version of nginx that is being run by the deployment

```bash
uubuntu@ip-172-31-15-199:~$ kubectl apply -f deployment-update.yaml
deployment.apps/nginx-deployment configured

ubuntu@ip-172-31-15-199:~$ kubectl get pods -o wide
NAME                                READY   STATUS        RESTARTS   AGE     IP            NODE       NOMINATED NODE   READINESS GATES
nginx-deployment-74dbb4766d-5sntk   1/1     Running       0          5s      10.244.0.26   minikube   <none>           <none>
nginx-deployment-74dbb4766d-96cnz   1/1     Running       0          8s      10.244.0.23   minikube   <none>           <none>
nginx-deployment-74dbb4766d-bjxnl   1/1     Running       0          5s      10.244.0.27   minikube   <none>           <none>
nginx-deployment-74dbb4766d-tmj5w   1/1     Running       0          7s      10.244.0.25   minikube   <none>           <none>
nginx-deployment-74dbb4766d-tw78c   1/1     Running       0          8s      10.244.0.24   minikube   <none>           <none>
nginx-deployment-85996f8dbd-zw7xg   0/1     Terminating   0          4m17s   10.244.0.19   minikube   <none>           <none>

ubuntu@ip-172-31-15-199:~$ kubectl get deployment nginx-deployment -o wide
NAME               READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES         SELECTOR
nginx-deployment   5/5     5            5           56m   nginx        nginx:1.25.2   app=nginx

```

### 4. Delete the Deployment
 
Delete the deployment and check to see that there are no pods running.

```nashorn js
ubuntu@ip-172-31-15-199:~$ kubectl delete deployment nginx-deployment
deployment.apps "nginx-deployment" deleted

ubuntu@ip-172-31-15-199:~$ kubectl get pods -o wide
No resources found in default namespace.

```

