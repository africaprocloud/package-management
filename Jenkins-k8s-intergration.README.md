Deployment to kubernetes
-
**create service account in master node**

      apiVersion: v1
      kind: ServiceAccount
      metadata:
        name: jenkins
        namespace: webapps

Vi into svc.yml and paste the above content 

first create the name space (webapps)

    kubectl create ns webapps  

    kubectl apply -f svc.yml   

**The next thing is to create a role with some permissions**

       apiVersion: rbac.authorization.k8s.io/v1
       kind: Role
       metadata:
         name: app-role
         namespace: webapps
       rules:
         - apiGroups:
               - ""
               - apps
               - autoscaling
               - batch
               - extensions
               - policy
               - rbac.authorization.k8s.io
           resources:
             - pods
             - secrets
             - componentstatuses
             - configmaps
             - daemonsets
             - deployments
             - events
             - endpoints
             - horizontalpodautoscalers
             - ingress
             - jobs
             - limitranges
             - namespaces
             - nodes
             - pods
             - persistentvolumes
             - persistentvolumeclaims
             - resourcequotas
             - replicasets
             - replicationcontrollers
             - serviceaccounts
             - services
           verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]

vi into role.yml and paste the above yml file for roles settings.

     kubectl apply -f role.yml 

**Then assign or bind the role to the service account by using the below yml file.**

       apiVersion: rbac.authorization.k8s.io/v1
       kind: RoleBinding
       metadata:
         name: app-rolebinding
         namespace: webapps 
       roleRef:
         apiGroup: rbac.authorization.k8s.io
         kind: Role
         name: app-role 
       subjects:
       - namespace: webapps 
         kind: ServiceAccount
         name: jenkins      

vi into bind.yml , paste the file and execute.

   kubectl apply -f rolebinding.yml

**For Jenkins to be able to connect to kubernetes, you need to create a token that will be use for authentication. Use the below file for that.**

       apiVersion: v1
       kind: Secret
       type: kubernetes.io/service-account-token
       metadata:
         name: mysecretname
         annotations:
           kubernetes.io/service-account.name: jenkins

vi secret.yml, paste the file and execute.

   kubectl apply -f secret.yml -n webapps

When this is done, run the below command to copy the token that you will use in jenkins for authentication.

     kubectl describe secret mysecretname -n webapp  

**Carefully copy the token**

In Jenkins, go to dashboard.
manage jenkins.
credentials.
Add credentials.
select the secret text.

Enter the token as secret

**Go to your project, click on pipeline synthax.
select;**

  withkubeCredentials:Configure kubernetes CLI

**For the  kubernetes server endpoint;**

ssh to your masternode.

     cd ~/.kube
     ls
     cat congif

**copy the server endpoint and the cluster name from there. 
Paste it in the kubernetes server endpoint block in the pipeline synthax respectively.**

namespace = webapps

Generate the block
copy and include in your pipeline
use command;

    kubectl apply -f (k8s-manifestfile.yml name)

**Make sure kubectl is installed in jenkins.** 

So execute the below scripe in the jenkins server.

       curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
       chmod +x ./kubectl
       sudo mv ./kubectl /usr/local/bin
       kubectl version --short --client
