# Services and Networking
## Lab Objectives
In this lab you will learn how to expose Pods using Kubernetes Services. You will create ClusterIP and NodePort services, explore service discovery with DNS, and practice both declarative and imperative approaches to creating services. By the end of this lab you will understand how traffic reaches your applications inside a Kubernetes cluster.

## Lab Structure - Overview
1. Create a Pod to expose
2. Expose a Pod with ClusterIP
3. Expose a Pod with NodePort
4. Service Discovery with DNS
5. Using kubectl expose
6. Cleanup

## Execute on the Leader node

The following commands should all be executed in the VS Code window connected to the leader node.

### Create a Pod to expose

1. Create and enter the working directory for this lab.
```bash
mkdir -p $HOME/labs/services && cd $HOME/labs/services
```

2. Create a manifest file for an nginx Pod. Create a file called `web-pod.yaml` with the following content:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-server
  labels:
    app: web-server
spec:
  containers:
  - name: nginx
    image: nginx:alpine
    ports:
    - containerPort: 80
```

3. Deploy the Pod.
```
kubectl apply -f web-pod.yaml
```

4. Confirm the Pod is running.
```
kubectl get pods -o wide
```

You should see something like this:
```
NAME         READY   STATUS    RESTARTS   AGE   IP            NODE
web-server   1/1     Running   0          10s   10.244.1.5    worker-node
```

**Note**: The Pod IP shown is an internal cluster IP. You cannot reach it from outside the cluster. This is why we need Services.

### Expose a Pod with ClusterIP

A ClusterIP service gives your Pod a stable internal IP address and DNS name that other Pods in the cluster can use. This is the default service type.

1. Create a file called `clusterip-svc.yaml` with the following content:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-clusterip
spec:
  type: ClusterIP
  selector:
    app: web-server
  ports:
  - port: 80
    targetPort: 80
```

**Note**: The `selector` field matches the `app: web-server` label on the Pod. This is how the Service knows which Pods to send traffic to.

2. Deploy the service.
```
kubectl apply -f clusterip-svc.yaml
```

3. View the service.
```
kubectl get svc web-clusterip
```

You should see something like this:
```
NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
web-clusterip   ClusterIP   10.96.142.53    <none>        80/TCP    5s
```

Notice the `CLUSTER-IP` column. This is a virtual IP address that only works inside the cluster. The `EXTERNAL-IP` is `<none>` because ClusterIP services are not accessible from outside the cluster.

4. Get more details about the service.
```
kubectl describe svc web-clusterip
```

Look for the `Endpoints` field in the output. It should show the Pod IP and port. This confirms the Service has found the Pod using the label selector.

5. Test the service from inside the cluster by running a temporary Pod with `curl`.
```
kubectl run curl-test --image=curlimages/curl --rm -it --restart=Never -- curl http://web-clusterip
```

You should see the default nginx welcome page HTML:
```
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...
```

**Note**: The `--rm` flag deletes the Pod after it exits. The `-it` flags attach your terminal to the Pod so you can see the output. The `--restart=Never` flag creates a Pod instead of a Deployment.

6. Confirm the temporary Pod was removed.
```
kubectl get pods
```

You should only see the `web-server` Pod.

**PROTIP: ClusterIP is the right choice when your service only needs to be reached by other applications running inside the cluster, such as a database or internal API.**

### Expose a Pod with NodePort

A NodePort service exposes your application on a port on every node in the cluster. This allows external traffic to reach your Pod.

1. Create a file called `nodeport-svc.yaml` with the following content:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-nodeport
spec:
  type: NodePort
  selector:
    app: web-server
  ports:
  - port: 80
    targetPort: 80
```

**Note**: We are not specifying a `nodePort` value, so Kubernetes will automatically assign one from the range 30000-32767.

2. Deploy the service.
```
kubectl apply -f nodeport-svc.yaml
```

3. View the service.
```
kubectl get svc web-nodeport
```

You should see something like this:
```
NAME           TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
web-nodeport   NodePort   10.96.58.190   <none>        80:31245/TCP   5s
```

Notice the `PORT(S)` column shows `80:31245/TCP`. The number after the colon (`31245` in this example) is the NodePort. Your number will be different.

4. Get the NodePort value.
```
kubectl get svc web-nodeport -o jsonpath='{.spec.ports[0].nodePort}'
```

5. Now access nginx from your browser. Open a browser and navigate to:
```
http://<LEADER_IP>:<NodePort>
```

Replace `<LEADER_IP>` with the public IP of your leader node and `<NodePort>` with the port number from step 4. You should see the nginx welcome page.

**Note**: NodePort services expose the application on the same port on ALL nodes in the cluster. Traffic to any node's IP on that port will be routed to the Pod, regardless of which node the Pod is actually running on.

6. You can also test with `curl` directly from the leader node.
```
curl http://localhost:$(kubectl get svc web-nodeport -o jsonpath='{.spec.ports[0].nodePort}')
```

### Service Discovery with DNS

Kubernetes runs an internal DNS server (CoreDNS) that automatically creates DNS records for every Service. This means Pods can reach Services by name instead of IP address.

1. Run a temporary busybox Pod and look up the ClusterIP service by name.
```
kubectl run dns-test --image=busybox:1.36 --rm -it --restart=Never -- nslookup web-clusterip.default.svc.cluster.local
```

You should see output similar to:
```
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Name:      web-clusterip.default.svc.cluster.local
Address 1: 10.96.142.53 web-clusterip.default.svc.cluster.local
```

This shows that `web-clusterip` resolves to the ClusterIP address. The DNS server at `10.96.0.10` is CoreDNS.

**Note**: If you use just the short name (`nslookup web-clusterip`), you may see `NXDOMAIN` errors as busybox tries several search domains before finding the correct one. This is normal — the resolution still succeeds. Using the FQDN avoids these intermediate errors.

2. Services can also be reached using their fully qualified domain name (FQDN). The format is:
```
<service-name>.<namespace>.svc.cluster.local
```

3. Test the FQDN from a temporary Pod.
```
kubectl run dns-test2 --image=curlimages/curl --rm -it --restart=Never -- curl http://web-clusterip.default.svc.cluster.local
```

You should see the nginx welcome page, confirming the FQDN works.

4. Now test cross-namespace service discovery. Create a new namespace.
```
kubectl create namespace test-ns
```

5. Create a Pod and Service in the new namespace. Run the following commands:
```
kubectl run web-test --image=nginx:alpine --port=80 --restart=Never -n test-ns
```

```
kubectl expose pod web-test --port=80 --name=web-test-svc -n test-ns
```

6. From the `default` namespace, resolve the service in `test-ns`.
```
kubectl run dns-cross --image=busybox:1.36 --rm -it --restart=Never -- nslookup web-test-svc.test-ns.svc.cluster.local
```

You should see the service resolve successfully:
```
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Name:      web-test-svc.test-ns.svc.cluster.local
Address 1: 10.96.xxx.xxx web-test-svc.test-ns.svc.cluster.local
```

**PROTIP: Within the same namespace, you can use just the service name (e.g., `web-clusterip`). To reach a service in a different namespace, use `<service-name>.<namespace>` or the full FQDN.**

### Using kubectl expose

So far you have created Services using YAML manifests (the declarative approach). Kubernetes also supports creating Services imperatively using `kubectl expose`.

1. First, delete the existing services so we can recreate one.
```
kubectl delete svc web-clusterip web-nodeport
```

2. Create a NodePort service using `kubectl expose`.
```
kubectl expose pod web-server --type=NodePort --port=80 --name=web-quick
```

3. View the new service.
```
kubectl get svc web-quick
```

You should see something like this:
```
NAME        TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
web-quick   NodePort   10.96.33.100   <none>        80:30812/TCP   3s
```

4. Verify it works.
```
kubectl run curl-verify --image=curlimages/curl --rm -it --restart=Never -- curl http://web-quick
```

5. Compare the two approaches by looking at what Kubernetes generated.
```
kubectl get svc web-quick -o yaml
```

**Note**: The imperative approach (`kubectl expose`) is quick and useful for testing. The declarative approach (YAML manifests) is better for production because you can version control your files, review changes, and reproduce environments reliably. You will use the declarative approach throughout the rest of this course.

### Cleanup

Run the following commands to delete all resources created in this lab.

1. Delete resources in the default namespace.
```
kubectl delete pod web-server
kubectl delete svc web-quick
```

2. Delete the test namespace and all its resources.
```
kubectl delete namespace test-ns
```

3. Confirm everything is cleaned up.
```
kubectl get pods
kubectl get svc
```

You should only see the default `kubernetes` ClusterIP service:
```
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   1d
```

Now you understand how to expose Pods using ClusterIP and NodePort services, how Kubernetes DNS enables service discovery by name, and how to create services both declaratively and imperatively. In the next lab you will learn about Deployments, which manage multiple replicas of your Pods and work together with Services to provide reliable, scalable applications.

## Lab complete
