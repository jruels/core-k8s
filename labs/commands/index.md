# Commands and Namespaces
## Lab Objectives
This lab has you run a few different commands to familiarize yourself with `kubectl`.

## Lab Structure - Overview
1. List Kubernetes Pods and Nodes
2. List Kubernetes Services and Namespaces
3. Look at Kubernetes components
4. Output Formatting
5. Working with Namespaces
6. Exploring the API
7. Cleanup



## Execute on the Leader node

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

### Output Formatting

`kubectl` supports several output formats that let you control how much detail you see.

1. You already used `-o wide` earlier. Run it again on Pods to see additional columns like Node and IP.
```
kubectl get pods -n kube-system -o wide
```

2. View the full YAML definition of a resource. This shows every field Kubernetes knows about.
```
kubectl get svc kubernetes -o yaml
```

3. View the same resource as JSON.
```
kubectl get svc kubernetes -o json
```

**PROTIP: YAML output is great for creating manifest files from existing resources.**

4. Use `-o jsonpath` to extract specific fields. For example, get the internal IP addresses of all nodes.
```
kubectl get nodes -o jsonpath='{.items[*].status.addresses[?(@.type=="InternalIP")].address}'
```

5. Get just the names of all Pods in `kube-system`.
```
kubectl get pods -n kube-system -o jsonpath='{.items[*].metadata.name}'
```

6. Use `--sort-by` to sort output by a field. Sort Pods by creation time.
```
kubectl get pods -n kube-system --sort-by=.metadata.creationTimestamp
```

7. Sort nodes by name.
```
kubectl get nodes --sort-by=.metadata.name
```

8. Combine flags. List all Pods in `kube-system` sorted by node name with wide output.
```
kubectl get pods -n kube-system --sort-by=.spec.nodeName -o wide
```

### Working with Namespaces

Namespaces let you organize and isolate resources within a cluster.

1. Create a new namespace called `lab-test`.
```
kubectl create namespace lab-test
```

2. Confirm the namespace exists.
```
kubectl get namespaces
```

3. Deploy a simple Pod into the `lab-test` namespace.
```
kubectl run nginx --image=nginx --namespace=lab-test
```

4. Verify the Pod is running in `lab-test`.
```
kubectl get pods -n lab-test
```

**NOTE: Without `-n lab-test`, this Pod will not appear because `kubectl` defaults to the `default` namespace.**

5. Try listing Pods without specifying a namespace.
```
kubectl get pods
```
You should see `No resources found in default namespace.`

6. List Pods across all namespaces to see everything.
```
kubectl get pods -A
```

**PROTIP: `-A` is the short form of `--all-namespaces`.**

7. Set `lab-test` as your default namespace so you don't have to type `-n lab-test` every time.
```
kubectl config set-context --current --namespace=lab-test
```

8. Now list Pods without specifying a namespace. You should see the nginx Pod.
```
kubectl get pods
```

9. Verify your current context and default namespace.
```
kubectl config get-contexts
```

10. Set the default namespace back to `default`.
```
kubectl config set-context --current --namespace=default
```

11. Delete the `lab-test` namespace. This will delete the namespace and all resources inside it.
```
kubectl delete namespace lab-test
```

12. Watch the cascading deletion. The namespace will show a `Terminating` status while resources are being cleaned up.
```
kubectl get namespaces
```

13. Confirm the nginx Pod is gone.
```
kubectl get pods -A | grep lab-test
```

**NOTE: Deleting a namespace removes everything inside it. Be careful in production environments.**

### Exploring the API

Kubernetes has many resource types. Use these commands to discover and understand them.

1. List all resource types available in the cluster.
```
kubectl api-resources
```

2. List only namespaced resources (resources that live inside a namespace).
```
kubectl api-resources --namespaced=true
```

3. List only cluster-scoped resources (resources that are NOT inside a namespace).
```
kubectl api-resources --namespaced=false
```

**PROTIP: Nodes, Namespaces, and PersistentVolumes are cluster-scoped. Pods, Services, and Deployments are namespaced.**

4. Notice the `SHORTNAMES` column in the output. These are shortcuts you can use with `kubectl`. Some common ones:

| Short Name | Full Name    |
|------------|-------------|
| po         | pods         |
| svc        | services     |
| deploy     | deployments  |
| ns         | namespaces   |
| no         | nodes        |
| ds         | daemonsets   |
| rs         | replicasets  |

5. Try using short names.
```
kubectl get no
kubectl get ns
kubectl get po -A
```

6. Use `kubectl explain` to view documentation for a resource type.
```
kubectl explain pod
```

7. Drill into nested fields using dot notation.
```
kubectl explain pod.spec.containers
```

8. Go even deeper. Look at the `ports` field on a container.
```
kubectl explain pod.spec.containers.ports
```

**PROTIP: `kubectl explain` is very useful when writing YAML manifests and you can't remember the correct field names.**

9. Use `--recursive` to see all fields for a resource at once.
```
kubectl explain pod.spec --recursive
```

### Cleanup

There is nothing to clean up. The `lab-test` namespace and its resources were already deleted in the Working with Namespaces section.

Confirm your default namespace is set back to `default`.
```
kubectl config get-contexts
```

If the default namespace is not `default`, reset it.
```
kubectl config set-context --current --namespace=default
```

Now you should have a better understanding of Kubernetes resources, output formatting, namespaces, the API, and using the `kubectl` client.
