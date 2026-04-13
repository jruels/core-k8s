# Labels, Annotations, and Scheduling
## Lab Objectives
In this lab you will learn how to use labels and annotations to organize and select Kubernetes resources. You will see how labels drive service selection, pod scheduling, and resource management. These concepts are foundational for understanding Deployments and Services in upcoming labs.

## Lab Structure - Overview
1. Working with Labels
2. Adding and Modifying Labels
3. Labels and Services
4. Working with Annotations
5. Node Labels and Scheduling
6. Cleanup

## Execute on the Leader node

The following commands should all be executed in the VS Code window connected to the leader node.

### Working with Labels

1. Create a working directory for this lab and change into it.
```bash
mkdir -p $HOME/labs/labels && cd $HOME/labs/labels
```

2. Create a file called `frontend-prod.yaml` with the following content:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend-prod
  labels:
    app: frontend
    env: production
spec:
  containers:
  - name: nginx
    image: nginx:alpine
    ports:
    - containerPort: 80
```

3. Create a file called `frontend-staging.yaml` with the following content:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend-staging
  labels:
    app: frontend
    env: staging
spec:
  containers:
  - name: nginx
    image: nginx:alpine
    ports:
    - containerPort: 80
```

4. Create a file called `backend-prod.yaml` with the following content:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: backend-prod
  labels:
    app: backend
    env: production
spec:
  containers:
  - name: nginx
    image: nginx:alpine
    ports:
    - containerPort: 80
```

5. Deploy all three pods.
```
kubectl apply -f frontend-prod.yaml
kubectl apply -f frontend-staging.yaml
kubectl apply -f backend-prod.yaml
```

6. Confirm all pods are running.
```
kubectl get pods
```

Sample output:
```
NAME               READY   STATUS    RESTARTS   AGE
backend-prod       1/1     Running   0          5s
frontend-prod      1/1     Running   0          10s
frontend-staging   1/1     Running   0          7s
```

7. List all pods and show their labels.
```
kubectl get pods --show-labels
```

Sample output:
```
NAME               READY   STATUS    RESTARTS   AGE   LABELS
backend-prod       1/1     Running   0          30s   app=backend,env=production
frontend-prod      1/1     Running   0          35s   app=frontend,env=production
frontend-staging   1/1     Running   0          32s   app=frontend,env=staging
```

8. Filter pods by a single label. Show only pods with `app=frontend`.
```
kubectl get pods -l app=frontend
```

You should see `frontend-prod` and `frontend-staging` but not `backend-prod`.

9. Filter with multiple selectors. Show pods that are both `app=frontend` AND `env=production`.
```
kubectl get pods -l app=frontend,env=production
```

You should see only `frontend-prod`.

10. Use a set-based selector to match multiple values. Show pods where `env` is either `production` or `staging`.
```
kubectl get pods -l 'env in (production,staging)'
```

All three pods should appear.

11. Use an inequality selector. Show all pods that are NOT `backend`.
```
kubectl get pods -l app!=backend
```

You should see the two `frontend` pods.

**PROTIP: Labels are key-value pairs attached to objects. Selectors let you filter objects by their labels. Equality-based selectors use `=`, `==`, and `!=`. Set-based selectors use `in`, `notin`, and `exists`.**

### Adding and Modifying Labels

1. Add a new label `team=alpha` to the `frontend-prod` pod.
```
kubectl label pod frontend-prod team=alpha
```

2. Verify the label was added.
```
kubectl get pod frontend-prod --show-labels
```

Sample output:
```
NAME            READY   STATUS    RESTARTS   AGE   LABELS
frontend-prod   1/1     Running   0          2m    app=frontend,env=production,team=alpha
```

3. Try to update an existing label. Change `env` from `production` to `development`.
```
kubectl label pod frontend-prod env=development
```

This will fail with an error because the label already exists:
```
error: 'env' already has a value (production), and --overwrite is false
```

4. Use the `--overwrite` flag to update the existing label.
```
kubectl label pod frontend-prod env=development --overwrite
```

5. Verify the update.
```
kubectl get pod frontend-prod --show-labels
```

You should see `env=development` instead of `env=production`.

6. Remove the `team` label from the pod by appending a minus sign (`-`) to the label key.
```
kubectl label pod frontend-prod team-
```

7. Verify the label was removed.
```
kubectl get pod frontend-prod --show-labels
```

The `team` label should no longer appear.

8. Restore the original label so this pod is ready for the next section.
```
kubectl label pod frontend-prod env=production --overwrite
```

### Labels and Services

Labels are how Services know which Pods to route traffic to. In this section you will see that connection firsthand.

1. Create a file called `frontend-svc.yaml` with the following content:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-svc
spec:
  selector:
    app: frontend
    env: production
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

**Note**: The `selector` field means this Service will only send traffic to Pods that have BOTH `app: frontend` AND `env: production`.

2. Deploy the Service.
```
kubectl apply -f frontend-svc.yaml
```

3. Check which Pods back this Service by looking at the Endpoints.
```
kubectl get endpoints frontend-svc
```

Sample output:
```
NAME           ENDPOINTS        AGE
frontend-svc   10.244.1.5:80    5s
```

You should see only one IP address, corresponding to the `frontend-prod` pod.

4. Verify by checking the Pod IPs.
```
kubectl get pods -l app=frontend -o wide
```

Notice that the endpoint IP matches the `frontend-prod` pod, not `frontend-staging`. This is because `frontend-staging` has `env: staging`, which does not match the Service selector.

5. Now change the `frontend-staging` pod's `env` label to `production` so it matches the Service selector.
```
kubectl label pod frontend-staging env=production --overwrite
```

6. Check the endpoints again.
```
kubectl get endpoints frontend-svc
```

Sample output:
```
NAME           ENDPOINTS                     AGE
frontend-svc   10.244.1.5:80,10.244.2.3:80   30s
```

You should now see two IP addresses. The Service immediately picked up the newly matching Pod.

7. Change the label back to `staging`.
```
kubectl label pod frontend-staging env=staging --overwrite
```

8. Confirm the endpoints are back to one.
```
kubectl get endpoints frontend-svc
```

**Note**: This is exactly how Deployments work behind the scenes. A Deployment creates Pods with specific labels, and a Service selects those Pods by matching labels. Understanding this relationship is critical for Day 2 labs.

### Working with Annotations

Annotations are similar to labels but are NOT used for selection. They store arbitrary metadata such as descriptions, tool configuration, or build information.

1. Add an annotation to the `frontend-prod` pod.
```
kubectl annotate pod frontend-prod description="Primary frontend web server"
```

2. View the annotation using `kubectl describe`.
```
kubectl describe pod frontend-prod
```

Look for the `Annotations` section in the output:
```
Annotations:  description: Primary frontend web server
```

3. Add another annotation.
```
kubectl annotate pod frontend-prod owner="platform-team"
```

4. View annotations in a compact way using JSON output.
```
kubectl get pod frontend-prod -o jsonpath='{.metadata.annotations}' | python3 -m json.tool
```

Sample output:
```json
{
    "description": "Primary frontend web server",
    "owner": "platform-team"
}
```

5. Remove an annotation by appending a minus sign, just like labels.
```
kubectl annotate pod frontend-prod owner-
```

**PROTIP: Use labels when you need to select or group resources. Use annotations for metadata that tools or humans need to read but that Kubernetes does not use for selection.**

### Node Labels and Scheduling

Kubernetes uses node labels and `nodeSelector` to control which node a Pod runs on. This is useful when you need specific hardware, regions, or configurations.

1. View the existing labels on your nodes.
```
kubectl get nodes --show-labels
```

You will see many built-in labels like `kubernetes.io/hostname`, `kubernetes.io/os`, and `kubernetes.io/arch`.

2. Pick one of your worker nodes and add a custom label. Replace `<node-name>` with an actual node name from the previous command.
```
kubectl label node <node-name> disktype=ssd
```

**Note**: Use `kubectl get nodes` to find the name of a worker node (not the control plane node).

3. Verify the label was added.
```
kubectl get nodes --show-labels | grep disktype
```

4. Create a file called `ssd-pod.yaml` with the following content:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ssd-pod
  labels:
    app: database
spec:
  containers:
  - name: nginx
    image: nginx:alpine
  nodeSelector:
    disktype: ssd
```

The `nodeSelector` field tells Kubernetes to only schedule this Pod on nodes that have the label `disktype=ssd`.

5. Deploy the pod.
```
kubectl apply -f ssd-pod.yaml
```

6. Verify the pod is running on the correct node.
```
kubectl get pod ssd-pod -o wide
```

The `NODE` column should show the node you labeled with `disktype=ssd`.

7. Now create a pod that targets a label that does not exist on any node. Create a file called `nvme-pod.yaml`:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nvme-pod
  labels:
    app: database
spec:
  containers:
  - name: nginx
    image: nginx:alpine
  nodeSelector:
    disktype: nvme
```

8. Deploy the pod.
```
kubectl apply -f nvme-pod.yaml
```

9. Check the pod status.
```
kubectl get pod nvme-pod
```

Sample output:
```
NAME       READY   STATUS    RESTARTS   AGE
nvme-pod   0/1     Pending   0          10s
```

The pod stays `Pending` because no node matches the `disktype=nvme` selector.

10. Use `kubectl describe` to see why the pod is pending.
```
kubectl describe pod nvme-pod
```

Look at the `Events` section at the bottom. You should see a message like:
```
Warning  FailedScheduling  0/3 nodes are available: 3 node(s) didn't match Pod's node affinity/selector.
```

11. Delete the pending pod.
```
kubectl delete pod nvme-pod
```

12. Clean up the node label. Replace `<node-name>` with the same node name you used earlier.
```
kubectl label node <node-name> disktype-
```

### Cleanup

Delete all resources created in this lab.

```
kubectl delete pod frontend-prod frontend-staging backend-prod ssd-pod --ignore-not-found=true
kubectl delete svc frontend-svc --ignore-not-found=true
```

Confirm everything is cleaned up.
```
kubectl get pods
kubectl get svc
```

You should see no pods and only the default `kubernetes` service.

## Lab complete
