# Pods
## Lab Objectives
In this lab we will configure a Kubernetes Pod hosting a mysql database from the command-line. We’ll also log into the container we deploy into this lab using the mysql-client. Additionally, we’ll explore pod logs, observe pod lifecycle events and failure states, and configure resource requests and limits. We’ll expose the pod in a follow-up lab.

## Lab Structure - Overview
1. Run a command to deploy a pod from the command-line
2. Run a command to deploy a pod sourced from a YAML file
3. Run a command to exec a shell into a Kubernetes hosted container
4. Run a command to exec directly into the mysql-client from the command-line
5. View and stream pod logs, including logs from specific containers
6. Observe pod lifecycle phases and cluster events
7. Configure resource requests and limits and observe OOMKilled behavior

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

### Pod Logs

1. First, confirm the `mysql-demo` pod from the previous section is still running:
```
kubectl get pods
```

2. View the logs of the `mysql-demo` pod:
```
kubectl logs mysql-demo
```

3. Stream the logs in real time using the `-f` flag:
```
kubectl logs -f mysql-demo
```

Press `Ctrl+C` to stop streaming.

4. If the `two-containers` pod is still running, view the logs of a specific container using the `-c` flag:
```
kubectl logs two-containers -c nginx-container
```

5. View logs from the other container in the same pod:
```
kubectl logs two-containers -c debian-container
```

**Note**: When a pod has multiple containers, you must specify which container's logs to view using the `-c` flag.

6. Now deploy a pod that generates continuous log output:
```
kubectl run log-generator --image=busybox --restart=Never -- /bin/sh -c "i=0; while true; do echo \"$(date) - Log entry $i\"; i=$((i+1)); sleep 2; done"
```

7. Wait a few seconds, then view the logs:
```
kubectl logs log-generator
```

8. Stream the logs and watch new entries appear:
```
kubectl logs -f log-generator
```

Press `Ctrl+C` to stop streaming.

9. View only the last 5 log lines using `--tail`:
```
kubectl logs --tail=5 log-generator
```

10. The `--previous` flag lets you view logs from a previous instance of a container (useful when a container has crashed and restarted):
```
kubectl logs --previous log-generator
```

**Note**: This will show an error since the container has not restarted. You will use `--previous` in real-world scenarios when debugging pods that have crashed and been restarted by Kubernetes.

### Pod Lifecycle and Events

1. Use `kubectl describe` to look at the Events section of a running pod:
```
kubectl describe pod log-generator
```

Scroll to the bottom of the output and look at the `Events:` section. You should see events like `Scheduled`, `Pulling`, `Pulled`, `Created`, and `Started`.

Sample output:
```
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  2m    default-scheduler  Successfully assigned default/log-generator to node1
  Normal  Pulling    2m    kubelet            Pulling image "busybox"
  Normal  Pulled     2m    kubelet            Successfully pulled image "busybox"
  Normal  Created    2m    kubelet            Created container log-generator
  Normal  Started    2m    kubelet            Started container log-generator
```

2. View all recent cluster events:
```
kubectl get events --sort-by='.lastTimestamp'
```

**Note**: Events are a valuable troubleshooting tool. They show you what Kubernetes is doing behind the scenes when scheduling and running your pods.

3. Now create a pod with a bad image name to observe failure behavior:
```
kubectl run broken-pod --image=nginx:doesnotexist --restart=Never
```

4. Watch the pod status:
```
kubectl get pods broken-pod
```

You should see something like this:
```
NAME         READY   STATUS             RESTARTS   AGE
broken-pod   0/1     ErrImagePull       0          10s
```

5. Wait about 30 seconds and check again:
```
kubectl get pods broken-pod
```

The status should change to `ImagePullBackOff`:
```
NAME         READY   STATUS              RESTARTS   AGE
broken-pod   0/1     ImagePullBackOff    0          45s
```

**Note**: `ImagePullBackOff` means Kubernetes tried to pull the image, failed, and is now waiting with an exponential backoff before retrying.

6. Use `kubectl describe` to see the detailed error:
```
kubectl describe pod broken-pod
```

Look at the Events section at the bottom. You should see `Failed` events with messages about the image not being found.

7. Check the events for this specific pod:
```
kubectl get events --field-selector involvedObject.name=broken-pod
```

8. Delete the broken pod:
```
kubectl delete pod broken-pod
```

9. Now recreate it with the correct image name to see it recover:
```
kubectl run broken-pod --image=nginx:alpine --restart=Never
```

10. Confirm the pod is now running:
```
kubectl get pods broken-pod
```

You should see:
```
NAME         READY   STATUS    RESTARTS   AGE
broken-pod   1/1     Running   0          10s
```

### Resource Requests and Limits

Resource requests and limits let you control how much CPU and memory a container can use. **Requests** tell the scheduler how much a container needs (used for scheduling decisions). **Limits** set the maximum a container can consume (enforced at runtime).

1. Create a manifest file for a pod with resource requests and limits:
```bash
cat <<'EOF' > $HOME/core-k8s/labs/pods/resource-pod.yml
apiVersion: v1
kind: Pod
metadata:
  name: resource-pod
spec:
  containers:
  - name: resource-container
    image: nginx:alpine
    resources:
      requests:
        memory: "64Mi"
        cpu: "100m"
      limits:
        memory: "128Mi"
        cpu: "250m"
EOF
```

**Note**: CPU is measured in millicores (`100m` = 0.1 CPU). Memory is measured in bytes (`64Mi` = 64 mebibytes).

2. Deploy the pod:
```
kubectl apply -f $HOME/core-k8s/labs/pods/resource-pod.yml
```

3. Confirm the pod is running:
```
kubectl get pods resource-pod
```

4. Use `kubectl describe` to verify the resource settings:
```
kubectl describe pod resource-pod
```

Look for the `Requests` and `Limits` sections in the output:
```
    Limits:
      cpu:     250m
      memory:  128Mi
    Requests:
      cpu:     100m
      memory:  64Mi
```

5. Now create a pod with a memory limit lower than what the application needs, to observe what happens when a container exceeds its limit:
```bash
cat <<'EOF' > $HOME/core-k8s/labs/pods/memory-limit-pod.yml
apiVersion: v1
kind: Pod
metadata:
  name: memory-hog
spec:
  containers:
  - name: memory-hog
    image: polinux/stress
    command: ["stress"]
    args: ["--vm", "1", "--vm-bytes", "150M", "--vm-hang", "1"]
    resources:
      requests:
        memory: "50Mi"
      limits:
        memory: "50Mi"
EOF
```

**Note**: This pod is limited to 50Mi of memory but the `stress` tool will try to allocate 150Mi, which exceeds the limit.

6. Deploy the pod:
```
kubectl apply -f $HOME/core-k8s/labs/pods/memory-limit-pod.yml
```

7. Watch the pod status:
```
kubectl get pods memory-hog --watch
```

After a few seconds you should see the status change to `OOMKilled`:
```
NAME         READY   STATUS      RESTARTS   AGE
memory-hog   0/1     OOMKilled   0          5s
```

Press `Ctrl+C` to stop watching.

**PROTIP**: `OOMKilled` means the container was terminated because it tried to use more memory than its limit allows. Always set memory limits high enough for your application's actual needs, but low enough to protect the cluster from runaway processes.

8. Check the pod details to confirm the OOM event:
```
kubectl describe pod memory-hog
```

Look for `OOMKilled` in the `Last State` section of the output.

### Clean up

Run the following to delete all resources created in this lab:

```bash
kubectl delete --ignore-not-found=true -f manifests
kubectl delete pod --ignore-not-found=true log-generator broken-pod mysql-demo resource-pod memory-hog
kubectl delete --ignore-not-found=true -f $HOME/core-k8s/labs/pods/resource-pod.yml -f $HOME/core-k8s/labs/pods/memory-limit-pod.yml
```

## Lab complete
