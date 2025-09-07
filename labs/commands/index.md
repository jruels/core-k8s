# Commands and Namespaces
## Lab Objectives
This lab has you run a few different commands to familiarize yourself with `kubectl`.

## Lab Structure - Overview 
1. List Kubernetes Pods and Nodes
2. List Kubernetes Services and Namespaces
3. Look at Kubernetes components 



## Execute on Leader node 

The following commands should all be executed in the VS Code window connected to the leader node.

### List Pods and Nodes

1. Enter the following to list Pods in the `default` namespace
```
kubectl get pods 
```
**NOTE: This will return 'no resources found'**

2. List Pods in all namespaces.
```
kubectl get pods --all-namespaces
```

3. List nodes
```
kubectl get nodes 
```

4. List nodes and additional information (labels, name, IP etc.) 
```
kubectl get nodes -o wide 
```

### List Services and Namespaces
1. List Services in `default` namespace
```
kubectl get services
```

2. List Services in all namespaces.
```
kubectl get services --all-namespaces
```
**PROTIP: substitute `svc` for `services`**

3. List Namespaces
```
kubectl get namespaces
```

4. Create a new Namespace..  Use `kubectl create --help` if you get stuck. 

### Explore Kubernetes
Using `kubectl` interact with the Kubernetes cluster. 
1. Look at documentation
```
kubectl --help
```

2. Retrieve information about Kubernetes resources
```
kubectl get <random resources>
```

3. Look at node details
```
kubectl describe node <node from cluster>
```

4. Look at Pod details 
```
kubectl describe pod -n kube-system coredns
```

5. Check out service details
```
kubectl describe svc kubernetes
```

6. Look at deployment details
```
kubectl describe deployment -n kube-system coredns 
```

Now you should have a better understanding of Kubernetes resources and using the `kubectl` client.