## StatefulSet

### Task 1: Create Stateful Set
Download file with yaml defintion for an nginx Stateful Set 
```
wget https://s3.ap-south-1.amazonaws.com/files.cloudthat.training/devops/kubernetes-essentials/nginx-sts.yaml
```
Create a Stateful Set by applying the yaml
```
kubectl apply -f nginx-sts.yaml
```
Create a headless service
```
kubectl get service nginx-svc
```
```
Validate the stateful set creation
```
kubectl get statefulset nginx-sts
```
Watch the pods getting created in an ordinal index fashion
```
kubectl get pods -w -l app=nginx-sts
```
Go to each of the pods to see the hostname, as a DNS entry is created for each Pod, the hostname inside the Pod should match the pod-name
```
for i in 0 1; do kubectl exec "nginx-sts-$i" -- sh -c 'hostname'; done
```
Create a busybox pod to test the ping to one of the Ngninx pods by using DNS name of the pod
```
kubectl run -i --tty --image busybox:1.28 dns-test --restart=Never --rm
```
```
nslookup nginx-sts-0.nginx-svc
```
```
nslookup nginx-sts-1.nginx-svc
```
```
exit
```
Delete the pods for stateful set 
```
kubectl delete pod -l app=nginx-sts
```
In another window notice the new pods getting created in a proper order
```
kubectl get pod -w -l app=nginx-sts
```
Verify the hostname in each of the newly created pods, it will match the pod name
```
for i in 0 1; do kubectl exec "nginx-sts-$i" -- sh -c 'hostname'; done
```

### Task 3: Scaling a Stateful Set
Scale the Stateful Set to 5 replicas using below.

kubectl scale sts nginx-sts --replicas=5


# Verify the pods getting created in ordinal way

kubectl get pods -w -l app=nginx-sts

# Verify the PV Claim getting created in ordinal fashion
kubectl get pvc -l app=nginx-sts

# Edit the stateful Set yam and reduce replicas to 3 
kubectl edit sts nginx-sts

# Notice that the controller deletes the pods one at a time. It waits for one to completely shut down before going to next

kubectl get pods -w -l app=nginx-sts


# Verify statefulSet’s PersistentVolumeClaims and verify that are not deleted on scaling down. 

kubectl get pvc -l app=nginx-sts


-------------------------------------------------------------------------------
# Task 4 - Delete a Stateful Set - Cleanup
--------------------------------------------------------------------------------

kubectl delete -f nginx-sts.yaml

# List all the PV and PVC’s that has been allocated to Statefulset pods and delete them as below.

kubectl get pvc

kubectl delete pvc --all

