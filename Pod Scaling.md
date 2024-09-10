## Pod Scaling

### Task 1: Manual Scaling
Create the deployment Yaml File
```
 kubectl create deployment dep1 --image nginx --replicas 3 --dry-run=client -o yaml > dep1.yaml
```
Apply the yaml file
```
 kubectl apply -f dep1.yaml
```
Check the obejcts create. You would notice Deployemnt, Replicaset and Pods getting created
```
 kubectl get all
```
![image](https://github.com/user-attachments/assets/99c66d04-e948-475e-a17d-7218d92e978c)

Describe the deployment
```
 kubectl describe deploy dep1
```
Manually scale out the deployment using the imperative method
```
 kubectl scale deployment dep1 --replicas 5
```
```
 kubectl get all
```
![image](https://github.com/user-attachments/assets/d0f79e9e-9b82-445d-a4ca-8030986057b0)

Now we will scale the deployment by editing the yaml file and increasing the replicas to 6
```
 vi dep1.yaml
```
```
 kubectl apply -f dep1.yaml
```
```
 kubectl get all
```
![image](https://github.com/user-attachments/assets/2c806739-51be-4009-8bc7-96535caf65ae)

To decrease the numebr of repliacs, we again can do it by the imperative method or by editing the yaml file.
```
 kubectl scale deployment dep1 --replicas 3
```
```
 kubectl get all
```
![image](https://github.com/user-attachments/assets/a05544cf-e63c-4ee2-ab28-524bd706a266)


### Task 2: Horizantal Pod Autoscaler(HPA)

Install Metrics Server 
```
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```
Check the status of the metrics server
```
kubectl -n kube-system get po
```
Download the patch for the Mtrics Server
```
wget https://gist.githubusercontent.com/initcron/1a2bd25353e1faa22a0ad41ad1c01b62/raw/008e23f9fbf4d7e2cf79df1dd008de2f1db62a10/k8s-metrics-server.patch.yaml
```
Apply the patch
```
kubectl patch deploy metrics-server -p "$(cat k8s-metrics-server.patch.yaml)" -n kube-system
```
```
kubectl -n kube-system get po
```
Let’s deploy a sample application. 
```
vi nginx-deployment.yaml
```
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 1
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
        image: nginx
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: "100m"
            memory: "128Mi"
          limits:
            cpu: "250m"
            memory: "256Mi"
```
Apply the Deployment:
```
kubectl apply -f nginx-deployment.yaml
```
Verify the Deployment:
```
kubectl get deployment nginx-deployment
```
Expose the deployment
```
kubectl expose deployment nginx-deployment --port=80 --target-port=80
```
![image](https://github.com/user-attachments/assets/2de8938b-4d12-4f8d-bc9a-70ff8c47111d)

Now, create an HPA resource to scale the Nginx deployment based on CPU utilization.
```
vi hpa.yaml
```
```
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-deployment
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50  # Replaces targetCPUUtilizationPercentage
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 60  # Time to wait before scaling down (in seconds)
      policies:
      - type: Percent
        value: 100                     # Scale down by 100% within 60 seconds if needed
        periodSeconds: 60
    scaleUp:
      policies:
      - type: Percent
        value: 100                     # Scale up by 100% within 60 seconds if needed
        periodSeconds: 60
```
Apply the HPA
```
kubectl apply -f hpa.yaml
```
Verify the HPA
```
kubectl get hpa
```
![image](https://github.com/user-attachments/assets/8bec4bba-6c57-4ef3-b7b5-2b981c3bac0d)

Generate Load to Test HPA: To see HPA in action, we need to generate load on the Nginx deployment.
```
vi load-generator.yaml
```
```
apiVersion: v1
kind: Pod
metadata:
  name: load-generator
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["/bin/sh"]
    args: ["-c", "while true; do wget -q -O- http://nginx-deployment.default.svc.cluster.local; done"]
```
Apply the Load Generator:
```
kubectl apply -f load-generator.yaml
```
Monitor the HPA to see how it scales the Nginx deployment based on the load.
```
kubectl get hpa
```
![image](https://github.com/user-attachments/assets/35d6f103-e097-46f9-955b-63c6c227d544)

You should see the HPA scaling up the number of pods as the CPU utilization increases due to the load generated.

After some time you will also see the pods scaling down as the cpu utilization has reduced.
![image](https://github.com/user-attachments/assets/9ac3265c-4b00-4b0d-b2e5-931d3d22fab3)





