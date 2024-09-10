## Helm

### Installing WordPress with Helm

-----------------------------------------------------------------------
Task 1: Helm setup
-----------------------------------------------------------------------
#Download the shell script for the installation of the Helm package manager and run it.
wget  https://s3.ap-south-1.amazonaws.com/files.cloudthat.training/devops/kubernetes-essentials/helm.sh
bash helm.sh
helm version



-----------------------------------------------------------------------
Task 2: Setup WordPress
-----------------------------------------------------------------------
#Add bitnami repository to helm which contains WordPress chart.
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm repo list
 
#Search for WordPress package from the repository and install the WordPress chart with release name wordpress-chart. You can name it as you want.
helm search repo wordpress
helm install wordpress-chart bitnami/wordpress


#Perform helm fetch to get the WordPress compressed file to the working directory. Perform ls and verify that a compressed WordPress file is present in PWD.
helm fetch bitnami/wordpress
ls
 
#Extract the compressed file using the below command.
tar -xvzf wordpress-12.2.1.tgz


#Now, deploy the WordPress using helm install command.  This will set up the pods and services for WordPress.
helm install wordpress --generate-name

 
-----------------------------------------------------------------------
Task 3: Verify that WordPress has been set up 
-----------------------------------------------------------------------

#Verify that the pods are running.
kubectl get pods
 
 

#Now Notice that MariaDB (database) pods are part of statefulset.
kubectl get sts
kubectl describe sts
 
 

#Also Notice that the front end of wordpress are part of deployment.
kubectl get deploy

#View the services using the below commands.
#Make note of the EXTERNAL IP of the LoadBalancer service.
kubectl get svc
kubectl get svc --namespace default -w wordpress-1637763307
 

#Open the browser and paste the service endpoint noted on the previous step. Observe that the WordPress site is up and running.

 
-----------------------------------------------------------------------
Task 4: Cleanup
-----------------------------------------------------------------------

#List the current helm release and delete it
helm ls
helm delete <helm_Release_Name >
helm ls

 
 

==================================================================

**** Helm Chart ****

Installing Helm 3 in Ubuntu Linux and installing wordpress (LAMP) application through Helm.

apt update
wget https://get.helm.sh/helm-v3.11.3-linux-386.tar.gz
ls
tar -xvzf helm-v3.11.3-linux-386.tar.gz 
ls
cd linux-386/
ls
mv helm /bin/
helm list
helm repo list
helm repo add my-repo https://charts.bitnami.com/bitnami
helm repo list
helm install my-word my-repo/wordpress
helm list
kubectl get all
kubectl get pv,pvc
kubectl get sts
kubectl get deploy
kubectl get secrets
kubectl get svc
kubectl edit svc my-word-wordpress
helm list
helm status my-word
 
 
***************************************************************


*****************Delete Cluster*********************************
kops get cluster
kops delete cluster --name <cluster-naame> --state s3://<cluster-naame> --yes



 
 















