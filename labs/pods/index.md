# Pods
## Lab Objectives 
In this lab we will configure a Kubernetes Pod hosting a mysql database from the command-line. We’ll also log into the container we deploy into this lab using the mysql-client. We’ll expose the pod in a follow-up lab.

## Lab Structure - Overview 
1. Run a command to deploy a pod from the command-line
2. Run a command to deploy a pod sourced from a YAML file
3. Run a command to exec a shell into a Kubernetes hosted container
4. Run a command to exec directly into the mysql-client from the command-line

## Execute on the Leader node 

The following commands should all be executed in the VS Code window connected to the leader node.

### Deploy a Pod from the command-line 

1. Show the nodes of the cluster 
```
kubectl get nodes 
```

2. Show information about the cluster
```
kubectl cluster-info
```

3. Dump information about the cluster
```
kubectl cluster-info dump
```

4. Show all Pods in the `kube-system` namespace 
```
kubectl get pods -n kube-system
```

5. Deploy `nginx` Pod
```
kubectl run nginx-pod-lab --image=nginx:alpine --port=80 --restart=Never
```
**Note**: The `--restart=Never` flag ensures this creates a Pod (not a Deployment)

6. Confirm Pod is running'
```
kubectl get pods 
```

7. Get information about Pod in `json` format 
```
kubectl get pod nginx-pod-lab -o json 
```

8. Now get info about it in `YAML` syntax
```
kubectl get pod nginx-pod-lab -o yaml
```

9. Delete the pod. 
```
kubectl delete pod nginx-pod-lab
```

### Create Pod from manifest
1. Enter the lab directory.

```bash
cd $HOME/core-k8s/labs/pods
```

2. In the manifests directory, you will find  `nginx-kube.yml` . This file will launch a simple nginx server. Deploy it with the following:

```
kubectl apply -f manifests/nginx-kube.yml
```

3. Show the deployed Pods

```
kubectl get pods 
```

You should see something like this: 
```
NAME                            READY   STATUS    RESTARTS   AGE
nginx-pod-lab                   1/1     Running   0          18m
nginx-web                       1/1     Running   0          15s
```

4. Cleanup everything 

```
kubectl delete pods --all
```

### Create a Pod with 2 containers 
1. In the manifests directory you will find `two-containers.yml`. Review the manifest and see if you can figure out what it is doing, then deploy it: 
```
kubectl apply -f manifests/two-containers.yml
```

2. Show the deployed Pods
```
kubectl get pods 
```

3. Look at the details of the Pod 
```
kubectl describe pod two-containers 
```

Sample output: 
```
Name:         two-containers
Namespace:    default
Priority:     0
Node:         ip-192-168-29-163.us-west-2.compute.internal/192.168.29.163
Start Time:   Tue, 27 Aug 2019 22:38:37 -0700
Labels:       <none>
Annotations:  kubectl.kubernetes.io/last-applied-configuration:
                {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"name":"two-containers","namespace":"default"},"spec":{"containers":[{"image"...
              kubernetes.io/psp: eks.privileged
Status:       Running
IP:           192.168.22.118
Containers:
  nginx-container:
    Container ID:   docker://d5f34b906d03eddc2521c3860558d14174d2e63b786a22471a3c818cb5e5ad73
<snip>...
```

4. Log into the `nginx` container of the `two-containers` Pod:
```
kubectl exec -it two-containers -c nginx-container -- bash
```

5. Validate `nginx` is running in the container: 
```
apt-get update && apt-get install -y procps && ps aux
```

Sample output:
```
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.1  10624  5412 ?        Ss   05:38   0:00 nginx: master process nginx -g daemon off;
nginx        7  0.0  0.0  11096  2604 ?        S    05:38   0:00 nginx: worker process
root         8  0.0  0.0   3868  3244 pts/0    Ss   05:49   0:00 bash
root       338  0.0  0.0   7640  2672 pts/0    R+   05:49   0:00 ps aux
```

6. Install `curl` inside the container
```
apt install -y curl 
```

7. Check the default nginx page is loading using curl. 
```
curl localhost 
```

Output: 
```
Hello from the debian container
```

8. Exit the `nginx` container 
```
exit
```

### Deploy MySQLDB and connect to the client
1. Deploy a MySQL DB image
```
kubectl run mysql-demo --image=mysql:5.7 --port=3306 --env="MYSQL_ROOT_PASSWORD=password" --restart=Never
```
2. Show all of the running Pods, and note the name of the Pod you just created
```
kubectl get pods 
```

Sample output:
```
NAME             READY   STATUS     RESTARTS   AGE
mysql-demo       1/1     Running    0          3m24s
two-containers   1/2     NotReady   0          4m54s
```

3. Log into the new MySQL container: 
```
kubectl exec -it mysql-demo -- mysql -ppassword
```

4. Look around 
```bash
SHOW DATABASES;
```

```bash
SELECT * FROM mysql.user LIMIT 1\G
```

5. Exit MySQL: 

```bash
quit
```

### Clean up 

Run the following to delete resources created in this lab: 

```bash
kubectl delete --ignore-not-found=true manifests 
```



## Lab complete
