## Resource Quotas in Kubernetes

### Task 1: Creating a Namespace
Create a namespace
```
kubectl create namespace quotas
``` 
Verify namespace creation
```
kubectl get ns
```

### Task 2: Creating a resourcequota
Create a file quota.yaml
```
vi rq-quotas.yaml
```
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: quota
  namespace: quotas
spec:
  hard:
    requests.cpu: "1"
    requests.memory: 1Gi
    limits.cpu: "2"
    limits.memory: 2Gi
```
Save the file using "ESCAPE + :wq!" and Create the resourcequota from the yaml
```
kubectl create -f rq-quotas.yaml
``` 
Verify resourcequota creation
```
kubectl get resourcequota -n quotas
```

### Task 3: Verify resourcequota Functionality
Create a pod yaml called quota-pod.yaml 
```
vi rq-pod.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: quota-pod
  namespace: quotas
spec:
  containers:
  - name: quota-ctr
    image: nginx
    resources:
      limits:
        memory: "800Mi"
        cpu: "1000m"
      requests:
        memory: "600Mi"
        cpu: "350m"
    ports:
      - containerPort: 80
```
Save the file using "ESCAPE + :wq!" and Create the pod using the yaml created in the previous step
```
kubectl create -f rq-pod.yaml
```
Once the pod is created, view the resources used by the pod
```
kubectl get resourcequota -n quotas -o yaml
```
Create the same pod again and see that it does not get created due to the set resourcequotas. Two pods together will request 1.2Gi of memory while quota is set at 1Gi
```
kubectl create -f rq-pod.yaml
```

### Task 4: Limiting Number of Pods
Edit the existing resourcequota and include a limit for the number of pods
```
kubectl edit resourcequotas quota -n quotas
```
Append pods: “1” to the limits in the file. If the editing is successful, you will get "resourcequota/quota edited" output. Now Modify the pod memory limit to ensure that pod creation is not affected by the memory limit
```
vi rq-pod.yaml
```
```
apiVersion: v1
kind: Pod
metadata:
  name: quota-pod
  namespace: quotas
spec:
  containers:
  - name: quota-ctr
    image: nginx
    resources:
      limits:
        memory: "800Mi"
        cpu: "1000m"
      requests:
        memory: "375Mi"            #ensure the memory size
        cpu: "350m"
    ports:
      - containerPort: 80
```
Save the file using "ESCAPE + :wq!" and Now, try to create a pod and note that. It will not be created due to the restriction on the number of pods
```
kubectl create -f rq-pod.yaml
``` 

### Task 5: Clean-up
Delete the quota to clean up.
```
kubectl delete ns quotas
```
