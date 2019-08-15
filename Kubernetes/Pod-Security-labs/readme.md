# Use kops to create the cluster.

From the bastion host, use kops to create a cluster:

Use the terminal emulator or SSH to gain access to the 'Bastion Host' Cloud Server server instantiated for the lab.

ssh cloud_user@[IP Address of Bastion Host]
Once you have access you should be able to do a ls -l and see the k8s-create.sh.

ls -l
Execute the script to create the cluster configuration files.

. ./k8s-create.sh
Note: Answer any prompts as needed.

Use kops to edit the cluster configuration.

kops edit cluster
Under the spec: for the cluster, add the following lines.

spec:
    kubeAPIServer:
        admissionControl:
        - NamespaceLifecycle
        - LimitRanger
        - ServiceAccount
        - PersistentVolumeLabel
        - DefaultStorageClass
        - ResourceQuota
        - PodSecurityPolicy
        - DefaultTolerationSeconds
The display above has tabs, but you should normally use just two spaces to indent lines. The file should remain the same from the next line to the end.

api:
    dns: {} ... and so on as it was before.
After editing the cluster configuration to add the admission controller, use kops to update the cluster and create the nodes.

kops update cluster --name=$KOPS_CLUSTER_NAME --yes
Copy the command at the bottom to connect to the master node using SSH.

check_circle

# Create namespace, serviceaccount, and rolebinding.

Create the psp-ns namespace.

kubectl create namespace psp-ns
Create the psp-sa serviceaccount within the psp-ns namespace.

kubectl create serviceaccount -n psp-ns psp-sa
Create a rolebinding binding the cluster role verb edit to the service account psp-sa.

kubectl create rolebinding -n psp-ns rb-id --clusterrole=edit --serviceaccount=psp-ns:psp-sa
Now for convenience, create an alias for the psp-admin within the namespace.

alias psp-admin='kubectl -n psp-ns'
And create an alias for the psp-user within the namespace.

alias psp-user='kubectl --as=system:serviceaccount:psp-ns:psp-sa -n psp-ns'
check_circle

# Create the Pod Security Policy.

Use vi or a Linux editor to create the psp-policy YAML file.

vi psp-policy.yaml
Make sure the file is as follows.

apiVersion: policy/v1beta1 
kind: PodSecurityPolicy 
metadata:   
    name: psp-policy
spec:   
    privileged: false  # Don't allow privileged pods!   
    seLinux:     
        rule: RunAsAny   
    supplementalGroups:    
        rule: RunAsAny   
    runAsUser:     
        rule: RunAsAny   
    fsGroup:     
        rule: RunAsAny   
    volumes:   
    - '*'
Then create the policy.

psp-admin create -f psp-policy.yaml
check_circle

# Create a YAML file to deploy a pod, and attempt to create it.

Use an editor to create a YAML file to create a pod.

vi pod-pause.yaml
Edit the file as follows.

apiVersion: v1
kind: Pod
metadata:
    name: pod-pause
spec:
    containers:
    - name: pause
      image: k8s.gcr.io/pause
Now attempt to create the pod.

psp-user create -f pod-pause.yaml
To explain the error, you can use 'can-i' to see that the policy is not being used by the service account psp-sa.

psp-user auth can-i use podsecuritypolicy/psp-policy
check_circle

# Create a rolebinding to allow the psp-sa service account to use the policy and then re-attempt to create the pod.

Create a role to use the psp-policy.

psp-admin create role psp-role --verb=use --resource=podsecuritypolicy --resource-name=psp-policy
Create a rolebinding to bind the role to the serviceaccount.

psp-admin create rolebinding rb-id2 --role=psp-role --serviceaccount=psp-ns:psp-sa
Retry to create the pod as before.

psp-user create -f pod-pause.yaml
The pod should deploy. Check with:

psp-user get pods
check_circle

# Delete the pod from the previous step and attempt to deploy a privileged pod.

Delete the pod previously deployed.

psp-user delete po/pod-pause
Use the editor to create a YAML file for a privileged pod.

vi priv-pod.yaml
The YAML file should contain the following:

apiVersion: v1 
kind: Pod 
metadata:   
    name: privileged 
spec:   
    containers:
    - name:  pause
      image: k8s.gcr.io/pause
      securityContext:         
        privileged: true
Now try to create the pod.

psp-user create -f priv-pod.yaml
The attempt should fail.

check_circle

# Attempt to use a deployment to create the unprivileged pod.

Use an editor to create a YAML file for the deployment.

vi psp-deploy.yaml
The file contents should be:

apiVersion: apps/v1 
kind: Deployment 
metadata:  
    name: psp-deploy 
      labels: 
        app: paused 
spec: 
  replicas: 1 
  selector: 
    matchLabels: 
      app: paused 
  template: 
    metadata: 
      labels:  
        app: paused 
    spec: 
      containers: 
      - name: paused 
        image: k8s.gcr.io/pause 
Now attempt to create the deployment.

psp-user create -f psp-deploy.yaml
See if the pod deployed.

psp-user get pods
It should not have deployed.

Check the events to see what happened.

psp-user get events --sort-by='.metadata.creationTimestamp'
check_circle

# Clean up the failed deployment, add the needed role binding, and re-attempt the deployment.
 
Check if there are any pods running and delete as needed.

psp-user get pods
(delete as needed)

psp-user delete po/[pod name]
Check if the failed deployment exists and delete as needed.

psp-user get deploy 
(delete as needed)

psp-user delete deploy/[deployment name]
Create a role binding linking the role that allows use of the policy with the default service account.

psp-admin create rolebinding rb-id3 --role=psp-role --serviceaccount=psp-ns:default
Now reattempt to create the deployment as before.

psp-user create -f psp-deploy.yaml
Check the events to see what happened.

psp-user get events  --sort-by='.metadata.creationTimestamp'
Check the deployment.

psp-user get deploy
Check the pod.

psp-user get pods
Clean up as needed, and experiment with other namespaces and service accounts until this material is comfortable.

This completes this lab.
