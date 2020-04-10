# Let's Kube


![Kube architecture][logo]

[logo]: https://github.com/ramiljoaquin/HelloKubernetes_AKS/blob/master/assets/KubeArchitecture.png "Kubernetes architecture"



## Getting Started

### Kubernetes

- [Install kubectl](https://github.com/ramiljoaquin/HelloKubernetes/tree/master/docs/Kubernetes/tools/install-kubectl.md)
- [Install minikube](https://github.com/ramiljoaquin/HelloKubernetes/tree/master/docs/Kubernetes/tools/install-minikube.md)
- [Running Kubernetes locally via minikube](https://github.com/ramiljoaquin/HelloKubernetes/tree/master/docs/Kubernetes/tools/running-kubernetes-locally-via-minikube.md)

- [Kubernetes 101](https://medium.com/google-cloud/kubernetes-101-pods-nodes-containers-and-clusters-c1509e409e16)

### 1. Building Docker Image

1. Build the Docker Image

```bash
$ ls
Dockerfile              Pages                   Startup.cs              letskubedeploy.yml
LetsKube.csproj         Program.cs              appsettings.json
$ dotnet restore
  Restore completed in 4.48 sec for /Users/ramiljoaquin/Source/hello_kube/LetsKube.csproj.
$ docker build . -t letskube:local
Sending build context to Docker daemon  1.455MB
Step 1/10 : FROM microsoft/aspnetcore-build AS build-env
 ---> 06a6525397c2
Step 2/10 : WORKDIR /app
 ---> Using cache
 ---> e203dbfc0ab7
Step 3/10 : COPY *.csproj ./
 ---> Using cache
 ---> a8bd573338d5
Step 4/10 : RUN dotnet restore
 ---> Using cache
 ---> 3d3045b79a25
Step 5/10 : COPY . ./
 ---> c5f459c99132
Step 6/10 : RUN dotnet publish -c Release -o output
 ---> Running in fc63c46e28cd
Microsoft (R) Build Engine version 15.7.179.6572 for .NET Core
Copyright (C) Microsoft Corporation. All rights reserved.

  Restoring packages for /app/LetsKube.csproj...
  Generating MSBuild file /app/obj/LetsKube.csproj.nuget.g.props.
  Generating MSBuild file /app/obj/LetsKube.csproj.nuget.g.targets.
  Restore completed in 4.35 sec for /app/LetsKube.csproj.
  LetsKube -> /app/bin/Release/netcoreapp2.0/LetsKube.dll
  LetsKube -> /app/output/
Removing intermediate container fc63c46e28cd
 ---> adfce1d4b8ca
Step 7/10 : FROM microsoft/aspnetcore
 ---> db030c19e94b
Step 8/10 : WORKDIR /app
 ---> Using cache
 ---> d57649b16c2f
Step 9/10 : COPY --from=build-env /app/output .
 ---> c7865032d65c
Step 10/10 : ENTRYPOINT ["dotnet", "LetsKube.dll"]
 ---> Running in d0d831bf4b8b
Removing intermediate container d0d831bf4b8b
 ---> 0278d5f061c0
Successfully built 0278d5f061c0
Successfully tagged letskube:local
$
```

2. Now that you created your docker image, go ahead and list all your images.

```bash
$ docker image ls
REPOSITORY                              TAG                  IMAGE ID            CREATED             SIZE
letskube                                local                0278d5f061c0        2 minutes ago      347MB
mcr.microsoft.com/dotnet/core/runtime   3.0-alpine-arm64v8   13dfac7ab6b4        5 days ago          84.9MB
k8s.gcr.io/kube-proxy                   v1.14.3              004666307c5b        2 weeks ago         82.1MB
...
$
```

3. You can see that your brand new docker images called "letskube" and it's on the top of the list.

Now to test this image, go ahead and run a container based on this image.

```bash
$ docker run -d -p 5000:80 letskube:local
23e472ee089d90aad6d8c0cf6969578ff774d49da228628a47f3c38b97f0bc1d
$
```

4. To see the container up and running, run the command docker ps

```bash
$ docker ps
CONTAINER ID        IMAGE               COMMAND                 CREATED             STATUS              PORTS                  NAMES
23e472ee089d        letskube:local      "dotnet LetsKube.dll"   3 minutes ago       Up 3 minutes        0.0.0.0:5000->80/tcp   focused_hypatia
$
```

Oh yeah, since it's now up and running, why not try to resolve this url in your browser: http://localhost:5000/

### 2. Deploying the Application to a Local Kubernetes Cluster

1. First things first, please install https://www.virtualbox.org/wiki/Downloads then after the installation, start your minikube.

```bash
$ minikube start
😄  minikube v1.1.1 on darwin (amd64)
💡  Tip: Use 'minikube start -p <name>' to create a new cluster, or 'minikube delete' to delete this one.
🔄  Restarting existing virtualbox VM for "minikube" ...
⌛  Waiting for SSH access ...
🐳  Configuring environment for Kubernetes v1.14.3 on Docker 18.09.6
🔄  Relaunching Kubernetes v1.14.3 using kubeadm ...
⌛  Verifying: apiserver proxy etcd scheduler controller dns
🏄  Done! kubectl is now configured to use "minikube"
$
```

NOTE:
So to use an image without uploading it, you can follow these steps:
a. Set the environment variables with eval $(minikube docker-env)
b. Build the image with the Docker daemon of Minikube (eg docker build -t my-image .)
c. Set the image in the pod spec like the build tag (eg my-image)
d. Set the imagePullPolicy to Never, otherwise Kubernetes will try to download the image.
Important note: You have to run eval $(minikube docker-env) on each terminal you want to use, since it only sets the environment variables for the current shell session.

```bash
$ docker build . -t letskube:local
Sending build context to Docker daemon  1.595MB
Step 1/10 : FROM microsoft/aspnetcore-build AS build-env
latest: Pulling from microsoft/aspnetcore-build
55cbf04beb70: Pull complete
1607093a898c: Pull complete
9a8ea045c926: Pull complete
d4eee24d4dac: Pull complete
9996a3dbf48f: Pull complete
e626b8d73a89: Pull complete
4d833e0fa218: Pull complete
1eebc8a262e7: Pull complete
4867c32ba582: Pull complete
c0ece037099c: Pull complete
Digest: sha256:ab861527a8485e7df91069e80cd7a94237c22995f13494c2cccc071b76e347f0
Status: Downloaded newer image for microsoft/aspnetcore-build:latest
 ---> 06a6525397c2
Step 2/10 : WORKDIR /app
 ---> Running in fe7415163748
Removing intermediate container fe7415163748
 ---> 08b7375df928
Step 3/10 : COPY *.csproj ./
 ---> 9e28e1dd0add
Step 4/10 : RUN dotnet restore
 ---> Running in 0904ee031933
  Restoring packages for /app/LetsKube.csproj...
  Installing Microsoft.AspNetCore.Identity 2.0.3.
  Installing Microsoft.Extensions.Identity.Core 2.0.3.
  Installing Microsoft.AspNetCore.Server.Kestrel.Https 2.0.3.
  Installing Microsoft.AspNetCore.Server.Kestrel.Transport.Libuv 2.0.3.
  Installing Microsoft.Extensions.Identity.Stores 2.0.3.
  Installing Microsoft.AspNetCore.Server.Kestrel.Transport.Abstractions 2.0.3.
  Installing Microsoft.AspNetCore.Server.Kestrel.Core 2.0.3.
  Installing Microsoft.AspNetCore.Server.Kestrel 2.0.3.
  Installing Microsoft.AspNetCore.Identity.EntityFrameworkCore 2.0.3.
  Installing Microsoft.AspNetCore.AzureAppServicesIntegration 2.0.3.
  Installing Microsoft.AspNetCore.AzureAppServices.HostingStartup 2.0.3.
  Installing Microsoft.AspNetCore.ApplicationInsights.HostingStartup 2.0.3.
  Installing Microsoft.AspNetCore 2.0.3.
  Installing Microsoft.AspNetCore.All 2.0.8.
  Generating MSBuild file /app/obj/LetsKube.csproj.nuget.g.props.
  Generating MSBuild file /app/obj/LetsKube.csproj.nuget.g.targets.
  Restore completed in 4.43 sec for /app/LetsKube.csproj.
Removing intermediate container 0904ee031933
 ---> ef2c281ed8a3
Step 5/10 : COPY . ./
 ---> 0f77ef283ecc
Step 6/10 : RUN dotnet publish -c Release -o output
 ---> Running in beb8141363aa
Microsoft (R) Build Engine version 15.7.179.6572 for .NET Core
Copyright (C) Microsoft Corporation. All rights reserved.

  Restoring packages for /app/LetsKube.csproj...
  Generating MSBuild file /app/obj/LetsKube.csproj.nuget.g.props.
  Generating MSBuild file /app/obj/LetsKube.csproj.nuget.g.targets.
  Restore completed in 953.26 ms for /app/LetsKube.csproj.
  LetsKube -> /app/bin/Release/netcoreapp2.0/LetsKube.dll
  LetsKube -> /app/output/
Removing intermediate container beb8141363aa
 ---> f7d750ea22ac
Step 7/10 : FROM microsoft/aspnetcore
latest: Pulling from microsoft/aspnetcore
55cbf04beb70: Already exists
2ad9fb8b9d3d: Pull complete
7debd121b442: Pull complete
76fbad01d0bd: Pull complete
1cba46fa0c00: Pull complete
Digest: sha256:8bd7808981568a92b7f1d0aa4bab07ef811e6417898847d9caf698a4a788ab11
Status: Downloaded newer image for microsoft/aspnetcore:latest
 ---> db030c19e94b
Step 8/10 : WORKDIR /app
 ---> Running in cea1b174ea5c
Removing intermediate container cea1b174ea5c
 ---> b7a2794848d3
Step 9/10 : COPY --from=build-env /app/output .
 ---> 2ec179730c31
Step 10/10 : ENTRYPOINT ["dotnet", "LetsKube.dll"]
 ---> Running in f298442e50a3
Removing intermediate container f298442e50a3
 ---> 04492c32c00c
Successfully built 04492c32c00c
Successfully tagged letskube:local
$
```

2. So to create a deployment, you can run the kubectl run command.

The --replica switch is to specify that you need three replicas of our application. So when you execute this command, the following things happen. A deployment is created where the desire state is set to three replicas.

Next, a ReplicaSet is created that ensures that there are always three replicas of the application running. The scheduler then schdules the pod deployment on the worker node, which then instructs the Docker engine running on the worker node to pull the image and then run them in the pods. Let's see these steps in action.

```bash
$ kubectl run hello-letskube --image=letskube:local --port=80 --replicas=3
kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version.
Use kubectl run --generator=run-pod/v1 or kubectl create instead.
deployment.apps/hello-letskube created
$
```

3. Let's run kubectl get deployments and you can the that the hello-letskube's desired stated of replicas is set to 3, and we currently have 3 pods running.

```bash
$ kubectl get deployments
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
hello-letskube        0/3     3            0           23s
$
```

4. Let's check the ReplicaSets using kubectl get rs command

```bash
$ kubectl get rs
NAME                             DESIRED   CURRENT   READY   AGE
hello-letskube-57848bf5c7        3         3         0       1m
$
```

5. Let's run the kubectl get pods to see the status of the pods deployed by the deployment

```bash
$ kubectl get pods
NAME                                   READY   STATUS             RESTARTS   AGE
hello-letskube-57848bf5c7-4mv64        0/1     ImagePullBackOff   0          50m
hello-letskube-57848bf5c7-6q47s        0/1     ImagePullBackOff   0          50m
hello-letskube-57848bf5c7-fgjh7        0/1     ImagePullBackOff   0          50m
$
```

6. To access the hello-minikube Deployment, expose it as a Service:

```bash
$ kubectl expose deployment hello-letskube --type=NodePort
service/hello-letskube exposed
$
```

7. This creates the service, which we can verify by running kubectl get service or svc.

```bash
$ kubectl get service
NAME             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
hello-letskube   NodePort    10.103.98.199   <none>        80:32613/TCP   4m46s
kubernetes       ClusterIP   10.96.0.1       <none>        443/TCP        4d
$
```

Another approach is to run the container using .yml file. Let's run the following command

```bash
$ kubectl apply -f letskubedeploy.yml
deployment.apps/hello-letskube created
service/letskube-service created
$
```

To test your deployment, run curl command, please replace the local IP and port with your minkube cluster loca IP and port.

```bash
curl -X GET http://192.168.99.100:32565
```

# Push image to Azure Container registry

## Prerequisites

Before taking this course, you need to do the hello_terraform exercise first.

You must also change the image repository from your letskubedeploy.yml. It must be point to your Azure Container Registry Login server. From image: letskube:local to image: devlabcontainerregistry.azurecr.io/letskube:v1

```json
spec:
      containers:
      - name: letskube
        image: devlabcontainerregistry.azurecr.io/letskube:v1
        imagePullPolicy: IfNotPresent
      restartPolicy: Always
```

# Getting Started

## Log in to registry

Before pushing and pulling container images, you must log in to the registry. To do so, use the az acr login command.

```bash
$ az acr login --name devlabContainerRegistry
Login Succeeded
$
```

1. To push an image to an Azure Container registry, you must first have an image. If you don't yet have any local container images, run the following docker build command to build an image to your local machine. For this example, build the letskube image.

```bash
$ docker build . -t letskube:latest
Sending build context to Docker daemon   1.72MB
Step 1/11 : FROM microsoft/aspnetcore-build AS build-env
 ---> 06a6525397c2
Step 2/11 : WORKDIR /app
 ---> Using cache
 ---> e203dbfc0ab7
Step 3/11 : COPY *.csproj ./
 ---> Using cache
 ---> a8bd573338d5
Step 4/11 : RUN dotnet restore
 ---> Using cache
 ---> 3d3045b79a25
Step 5/11 : COPY . ./
 ---> 988982a0cddf
Step 6/11 : RUN dotnet publish -c Release -o output
 ---> Running in b4d313a1a6e3
Microsoft (R) Build Engine version 15.7.179.6572 for .NET Core
Copyright (C) Microsoft Corporation. All rights reserved.

  Restoring packages for /app/LetsKube.csproj...
  Generating MSBuild file /app/obj/LetsKube.csproj.nuget.g.props.
  Generating MSBuild file /app/obj/LetsKube.csproj.nuget.g.targets.
  Restore completed in 2.57 sec for /app/LetsKube.csproj.
  LetsKube -> /app/bin/Release/netcoreapp2.0/LetsKube.dll
  LetsKube -> /app/output/
Removing intermediate container b4d313a1a6e3
 ---> aa9679c1d8d5
Step 7/11 : FROM microsoft/aspnetcore
 ---> db030c19e94b
Step 8/11 : WORKDIR /app
 ---> Using cache
 ---> d57649b16c2f
Step 9/11 : COPY --from=build-env /app/output .
 ---> ca7ad4456df4
Step 10/11 : EXPOSE 80 5000
 ---> Running in 7842b15607b0
Removing intermediate container 7842b15607b0
 ---> d62485db0442
Step 11/11 : ENTRYPOINT ["dotnet", "LetsKube.dll"]
 ---> Running in a0e2d8fbd9b5
Removing intermediate container a0e2d8fbd9b5
 ---> 073eaf562bcb
Successfully built 073eaf562bcb
Successfully tagged letskube:latest
$
```

2. Check your local images by running this command

```bash
$ docker images
REPOSITORY                              TAG                  IMAGE ID            CREATED             SIZE
letskube                                latest               073eaf562bcb        23 seconds ago      347MB
<none>                                  <none>               aa9679c1d8d5        24 seconds ago      2.03GB
$
```

3. Before you can push an image to your registry, you must tag it with the fully qualified name of your ACR login server. The login server name is in the format <registry-name>.azurecr.io (all lowercase), for example, mycontainerregistry007.azurecr.io.

Tag the image using the docker tag command. Replace <acrLoginServer> with the login server name of your ACR instance.

```bash
$ docker tag letskube devlabcontainerregistry.azurecr.io/letskube:v1
$
```

4. Finally, use docker push to push the image to the ACR instance. Replace <acrLoginServer> with the login server name of your ACR instance. This example creates the letskube repository, containing the letskube:v1 image.

```bash
$ docker push devlabcontainerregistry.azurecr.io/letskube:v1
The push refers to repository [devlabcontainerregistry.azurecr.io/letskube]
3b3fe6247a28: Pushed
57473a034f37: Pushed
94a7e5001357: Pushed
fea4f503ccf8: Pushed
3172a1c8308a: Pushed
264a7fdea008: Pushed
3b10514a95be: Pushed
v1: digest: sha256:02d2b312f9fe00cdb85347a8b331f4a7f5990a06f349b61af7559b017d1b5a69 size: 1790
$
```

5. After pushing the image to your container registry, remove the letskube:v1 image from your local Docker environment. (Note that this docker rmi command does not remove the image from the letskube repository in your Azure container registry.)

```bash
$ docker rmi devlabcontainerregistry.azurecr.io/letskube:v1
Untagged: devlabcontainerregistry.azurecr.io/letskube:v1
Untagged: devlabcontainerregistry.azurecr.io/letskube@sha256:02d2b312f9fe00cdb85347a8b331f4a7f5990a06f349b61af7559b017d1b5a69
$
```

# List container images

The following example lists the repositories in your registry:

```bash
$ az acr repository list --name devlabContainerRegistry --output table
Result
--------
letskube
$
```

The following example lists the tags on the hello-world repository.

```bash
$ az acr repository show-tags --name devlabContainerRegistry --repository letskube --output table
Result
--------
v1
$
```

# Run image from registry

Now, you can pull and run the letskube:v1 container image from your container registry by using docker run:

```bash
$ docker run devlabcontainerregistry.azurecr.io/letskube:v1
Unable to find image 'devlabcontainerregistry.azurecr.io/letskube:v1' locally
v1: Pulling from letskube
55cbf04beb70: Already exists
2ad9fb8b9d3d: Already exists
7debd121b442: Already exists
76fbad01d0bd: Already exists
1cba46fa0c00: Already exists
7d6e971060cf: Pull complete
77eb167548be: Pull complete
Digest: sha256:02d2b312f9fe00cdb85347a8b331f4a7f5990a06f349b61af7559b017d1b5a69
Status: Downloaded newer image for devlabcontainerregistry.azurecr.io/letskube:v1
info: Microsoft.AspNetCore.DataProtection.KeyManagement.XmlKeyManager[0]
      User profile is available. Using '/root/.aspnet/DataProtection-Keys' as key repository; keys will not be encrypted at rest.
info: Microsoft.AspNetCore.DataProtection.KeyManagement.XmlKeyManager[58]
      Creating key {a7604f72-0e42-4069-82db-c910c7e00655} with creation date 2019-07-17 05:57:32Z, activation date 2019-07-17 05:57:32Z, and expiration date 2019-10-15 05:57:32Z.
warn: Microsoft.AspNetCore.DataProtection.KeyManagement.XmlKeyManager[35]
      No XML encryptor configured. Key {a7604f72-0e42-4069-82db-c910c7e00655} may be persisted to storage in unencrypted form.
info: Microsoft.AspNetCore.DataProtection.Repositories.FileSystemXmlRepository[39]
      Writing data to file '/root/.aspnet/DataProtection-Keys/key-a7604f72-0e42-4069-82db-c910c7e00655.xml'.
Hosting environment: Production
Content root path: /app
Now listening on: http://[::]:80
Application started. Press Ctrl+C to shut down.
$
```

# Deploying the Application to a Azure Kubernetes Services Cluster

1. Create shell file authenticate_aks_to_acr.sh and copy and paste the code below:

```bash
AKS_RESOURCE_GROUP=IT.Kubernetes.SoutheastAsia.DevLab
AKS_CLUSTER_NAME=company-kube-devlab
ACR_RESOURCE_GROUP=IT.Kubernetes.SoutheastAsia.DevLab
ACR_NAME=devlabContainerRegistry

# Get the id of the service principal configured for AKS
CLIENT_ID=$(az aks show --resource-group $AKS_RESOURCE_GROUP --name $AKS_CLUSTER_NAME --query "servicePrincipalProfile.clientId" --output tsv)

# Get the ACR registry resource id
ACR_ID=$(az acr show --name $ACR_NAME --resource-group $ACR_RESOURCE_GROUP --query "id" --output tsv)

# Create role assignment
az role assignment create --assignee $CLIENT_ID --role acrpull --scope $ACR_ID
```

Then run this command

```bash
$ ./authenticate_aks_to_acr.sh
$
```

2. With our AKS Cluster ready, let's now deploy our application to it. But before we do so, we need to make one very important change. Fire up the letskubedeploy.yml YAML manifest. As of now the image is referring to a local Docker image. We need to change it to refer to the image we pushed to Azure Container Registry.

Let's grab the login server details using the az acr list command and use the query to extract the value of login which happen to be devlabcontainerregistry.azurecr.io.

```bash
$ az acr list --resource-group IT.Kubernetes.SoutheastAsia.De
ab --query "[].{acrLoginServer:loginServer}" --output table
AcrLoginServer
----------------------------------
devlabcontainerregistry.azurecr.io
$
```

Now let's change our image in our deployment manifest to point to this login server and make sure we provide the correct image name and tag which is letskube:v1

```json
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-letskube
  labels:
    app: letskube
spec:
  replicas: 3
  template:
    metadata:
      name: letskube
      labels:
        app: letskube
    spec:
      containers:
      - name: letskube
        image: devlabcontainerregistry.azurecr.io/letskube:v1
        imagePullPolicy: Always
      restartPolicy: Always
  selector:
    matchLabels:
      app: letskube


---

apiVersion: v1
kind: Service
metadata:
  name: letskube-service
spec:
  selector:
    app: letskube
  ports:
    - port: 80
  type: LoadBalancer


```

3. Also if you recollect, we deployed this application to the local cluster. We use the service type NodePort. Now that we're deploying those to a cloude service like Azure, we can change the type to LoadBalancer. This will deploy an Azure load balancer that will load balance the external request hitting our application. Let's save changes, and we are all set deploying our application to the Azure Kubernetes cluster.

```bash
$ kubectl apply -f letskubedeploy.yml
deployment.apps/hello-letskube unchanged
service/letskube-service configured
$
```

4. This will create a defined Kubernetes objects, which include a deployment and a service. This service created exposes the application to the internet. To monitor the progress of the deployment, we can run the following command.

```bash
$ kubectl get service --watch
NAME               TYPE           CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
kubernetes         ClusterIP      10.0.0.1      <none>        443/TCP        22h
letskube-service   LoadBalancer   10.0.180.19   <pending>     80:32370/TCP   17h
$
```

5. Initially, the EXTERNAL-IP of the letskube deployment service appears as pending. When the EXTERNAL-IP address changes from pending to an actual public IP address, use CTRL-C to stop the kubectl watch process. The following example output shows a valid public IP address assigned to the service:

```bash
$ kubectl get service --watch
NAME               TYPE           CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
kubernetes         ClusterIP      10.0.0.1      <none>        443/TCP        22h
letskube-service   LoadBalancer   10.0.180.19   <pending>     80:32370/TCP   17h
letskube-service   LoadBalancer   10.0.180.19   52.187.3.245   80:32370/TCP   17h
$
```

6. Run the kubectl get pods command

```bash
$ kubectl get pods
NAME                              READY   STATUS              RESTARTS   AGE
hello-letskube-5756ccf694-7f4nf   0/1     ContainerCreating   0          5s
hello-letskube-5756ccf694-q89vs   0/1     ContainerCreating   0          5s
hello-letskube-5756ccf694-x9zjn   0/1     ContainerCreating   0          5s
$
```

Watch it

```bash
$ kubectl get pods -w
NAME                              READY   STATUS              RESTARTS   AGE
hello-letskube-5756ccf694-7f4nf   0/1     ContainerCreating   0          8s
hello-letskube-5756ccf694-q89vs   0/1     ContainerCreating   0          8s
hello-letskube-5756ccf694-x9zjn   0/1     ContainerCreating   0          8s
hello-letskube-5756ccf694-x9zjn   1/1     Running             0          28s
hello-letskube-5756ccf694-7f4nf   1/1     Running             0          29s
hello-letskube-5756ccf694-q89vs   1/1     Running             0          29s
$

```

7. Once the status has changed to Running. Try to resolve the external IP (e.g http://52.187.3.245/) in your browser.

# Access the Kubernetes web dashboard in Azure Kubernetes Service (AKS)

Kubernetes includes a web dashboard that can be used for basic management operations. This dashboard lets you view basic health status and metrics for your applications, create and deploy services, and edit existing applications. This article shows you how to access the Kubernetes dashboard using the Azure CLI, then guides you through some basic dashboard operations.

For more information on the Kubernetes dashboard, see [Kubernetes Web UI Dashboard](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/).

1. For RBAC-enabled clusters
   If your AKS cluster uses RBAC, a ClusterRoleBinding must be created before you can correctly access the dashboard. By default, the Kubernetes dashboard is deployed with minimal read access and displays RBAC access errors. The Kubernetes dashboard does not currently support user-provided credentials to determine the level of access, rather it uses the roles granted to the service account. A cluster administrator can choose to grant additional access to the kubernetes-dashboard service account, however this can be a vector for privilege escalation. You can also integrate Azure Active Directory authentication to provide a more granular level of access.

Create a file called "role-kubernetesdashboard-binding.yml under /kubernetes folder

```json
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: kubernetes-dashboard
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: kubernetes-dashboard
  namespace: kube-system
```

2. Run the following command

```bash
$ kubectl apply -f role-kubernetesdashboard-binding.yml
clusterrolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
$
```

3. To start the Kubernetes dashboard, use the az aks browse command. The following example opens the dashboard for the cluster named company-kube-devlab in the resource group named IT.Kubernetes.SoutheastAsia.DevLab:

```bash
$ az aks browse --resource-group IT.Kubernetes.SoutheastAsia.DevLab --name company-kube-devlab
Merged "company-kube-devlab" as current context in /var/folders/c1/hpcxpvbj1q1c1yhp8djvkhlc0000gn/T/tmpzsk06g8a
Proxy running on http://127.0.0.1:8001/
Press CTRL+C to close the tunnel...
$
```

This command creates a proxy between your development system and the Kubernetes API, and opens a web browser to the Kubernetes dashboard. If a web browser doesn't open to the Kubernetes dashboard, copy and paste the URL address noted in the Azure CLI, typically http://127.0.0.1:8001.

[Kubernetes web dashboard in Azure Kubernetes Service (AKS)](https://docs.microsoft.com/en-us/azure/aks/kubernetes-dashboard)

# Install applications with Helm in Azure Kubernetes Service (AKS)

Helm is an open-source packaging tool that helps you install and manage the lifecycle of Kubernetes applications. Similar to Linux package managers such as APT and Yum, Helm is used to manage Kubernetes charts, which are packages of preconfigured Kubernetes resources.
This article shows you how to configure and use Helm in a Kubernetes cluster on AKS.

Before you begin
This article assumes that you have an existing AKS cluster. If you need an AKS cluster, see the AKS quickstart using the Azure CLI or using the Azure portal.

You also need the Helm CLI installed, which is the client that runs on your development system. It allows you to start, stop, and manage applications with Helm. If you use the Azure Cloud Shell, the Helm CLI is already installed. For installation instructions on your local platform see, [Installing Helm](https://docs.helm.sh/using_helm/#installing-helm).

Create a service account
Before you can deploy Helm in an RBAC-enabled AKS cluster, you need a service account and role binding for the Tiller service. For more information on securing Helm / Tiller in an RBAC enabled cluster, see Tiller, Namespaces, and RBAC. If your AKS cluster is not RBAC enabled, skip this step.

1. Create a file named helm-rbac.yaml and copy in the following YAML:

```json
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: tiller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: tiller
    namespace: kube-system
```

2. Create the service account and role binding with the kubectl apply command:

```bash
$ kubectl apply -f helm-rbac.yaml
serviceaccount/tiller created
clusterrolebinding.rbac.authorization.k8s.io/tiller created
$
```

3. Generate a Certificate Authority
   The simplest way to generate a certificate authority is to run two commands:
   Step 1
   ```bash
   $ openssl genrsa -out ./ca.key.pem 4096
   Generating RSA private key, 4096 bit long modulus
   ..................++
   ................................................................................................++
   e is 65537 (0x10001)
   $
   ```
   Step 2

```bash
$ openssl req -key ca.key.pem -new -x509 -days 7300 -sha256 -out ca.cert.pem -extensions v3_ca
# Answer the questions with your client user's info
$
```

4. If you are getting the error “Error Loading extension section v3_ca” using macOS on step 2, add the following to your /etc/ssl/openssl.cnf

```bash
[ v3_ca ]
basicConstraints = critical,CA:TRUE
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always,issuer:always
```

Note that the data input above is sample data. You should customize to your own specifications.

The above will generate both a secret key and a CA. Note that these two files are very important. The key in particular should be handled with particular care.

Often, you will want to generate an intermediate signing key. For the sake of brevity, we will be signing keys with our root CA.

5. Generating Certificates
   We will be generating two certificates, each representing a type of certificate:

- One certificate is for Tiller. You will want one of these per tiller host that you run.
- One certificate is for the user. You will want one of these per helm user.

Since the commands to generate these are the same, we’ll be creating both at the same time. The names will indicate their target.

First, the Tiller key:

```bash
$ openssl genrsa -out ./tiller.key.pem 4096
Generating RSA private key, 4096 bit long modulus
.......................................................................................................................................................++
.....................................................................................++
e is 65537 (0x10001)
$
```

6.vNext, generate the Helm client’s key:

```bash
$ openssl genrsa -out ./helm.key.pem 4096
Generating RSA private key, 4096 bit long modulus
........................................................++
..............................................................................++
e is 65537 (0x10001)
$
```

7. Again, for production use you will generate one client certificate for each user.

Next we need to create certificates from these keys. For each certificate, this is a two-step process of creating a CSR, and then creating the certificate.

```bash
$ openssl req -key tiller.key.pem -new -sha256 -out tiller.csr.pem
# Answer the questions with your client user's info

```

8. And we repeat this step for the Helm client certificate:

```bash
$ openssl req -key helm.key.pem -new -sha256 -out helm.csr.pem
# Answer the questions with your client user's info
$

```

9. (In rare cases, we’ve had to add the -nodes flag when generating the request.)

Now we sign each of these CSRs with the CA certificate we created (adjust the days parameter to suit your requirements):

```bash
$ openssl x509 -req -CA ca.cert.pem -CAkey ca.key.pem -CAcreateserial -in tiller.csr.pem -out tiller.cert.pem -days 365
Signature ok
subject=/C=PH/ST=Pampanga/L=Clark/O=Company/OU=DevOps/CN=devlab/emailAddress=ramil.joaquin@com
Getting CA Private Key
$
```

10. And again for the client certificate:

```bash
$ openssl x509 -req -CA ca.cert.pem -CAkey ca.key.pem -CAcreateserial -in helm.csr.pem -out helm.cert.pem  -days 365
Signature ok
subject=/C=PH/ST=Pampanga/L=Clark/O=Company/OU=DevOps/CN=devlab/emailAddress=ramil.joaquin@com
Getting CA Private Key
$
```

At this point, the important files for us are these:

```bash
# The CA. Make sure the key is kept secret.
ca.cert.pem
ca.key.pem
# The Helm client files
helm.cert.pem
helm.key.pem
# The Tiller server files.
tiller.cert.pem
tiller.key.pem
```

11. Now we’re ready to move on to the next steps.

```bash
$ helm init \
>     --tiller-tls \
>     --tiller-tls-cert tiller.cert.pem \
>     --tiller-tls-key tiller.key.pem \
>     --tiller-tls-verify \
>     --tls-ca-cert ca.cert.pem \
>     --service-account tiller \
>     --node-selectors "beta.kubernetes.io/os"="linux"
$HELM_HOME has been configured at /Users/ramiljoaquin/.helm.
Warning: Tiller is already installed in the cluster.
(Use --client-only to suppress this message, or --upgrade to upgrade Tiller to the current version.)
```

## Run Helm charts

To install charts with Helm, use the helm install command and specify the name of the chart to install. To see installing a Helm chart in action, let's install a basic nginx deployment using a Helm chart. If you configured TLS/SSL, add the --tls parameter to use your Helm client certificate.

```bash
$ helm install stable/nginx-ingress --set controller.nodeSelector."beta\.kubernetes\.io/os"=linux --set defaultBackend.nodeSelector."beta\.kubernetes\.io/os"=linux
NAME:   manageable-meerkat
LAST DEPLOYED: Wed Jul 17 22:43:50 2019
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/Pod(related)
NAME                                                             READY  STATUS             RESTARTS  AGE
manageable-meerkat-nginx-ingress-controller-7646cc8945-vpr4m     0/1    ContainerCreating  0         1s
manageable-meerkat-nginx-ingress-default-backend-f96ff85f64lsmz  0/1    ContainerCreating  0         1s

==> v1/Service
NAME                                              TYPE          CLUSTER-IP  EXTERNAL-IP  PORT(S)                     AGE
manageable-meerkat-nginx-ingress-controller       LoadBalancer  10.0.21.13  <pending>    80:30724/TCP,443:32118/TCP  1s
manageable-meerkat-nginx-ingress-default-backend  ClusterIP     10.0.51.50  <none>       80/TCP                      1s

==> v1/ServiceAccount
NAME                              SECRETS  AGE
manageable-meerkat-nginx-ingress  1        2s

==> v1beta1/ClusterRole
NAME                              AGE
manageable-meerkat-nginx-ingress  2s

==> v1beta1/ClusterRoleBinding
NAME                              AGE
manageable-meerkat-nginx-ingress  2s

==> v1beta1/Deployment
NAME                                              READY  UP-TO-DATE  AVAILABLE  AGE
manageable-meerkat-nginx-ingress-controller       0/1    1           0          1s
manageable-meerkat-nginx-ingress-default-backend  0/1    1           0          1s

==> v1beta1/Role
NAME                              AGE
manageable-meerkat-nginx-ingress  2s

==> v1beta1/RoleBinding
NAME                              AGE
manageable-meerkat-nginx-ingress  2s


NOTES:
The nginx-ingress controller has been installed.
It may take a few minutes for the LoadBalancer IP to be available.
You can watch the status by running 'kubectl --namespace default get services -o wide -w manageable-meerkat-nginx-ingress-controller'

An example Ingress that makes use of the controller:

  apiVersion: extensions/v1beta1
  kind: Ingress
  metadata:
    annotations:
      kubernetes.io/ingress.class: nginx
    name: example
    namespace: foo
  spec:
    rules:
      - host: www.example.com
        http:
          paths:
            - backend:
                serviceName: exampleService
                servicePort: 80
              path: /
    # This section is only required if TLS is to be enabled for the Ingress
    tls:
        - hosts:
            - www.example.com
          secretName: example-tls

If TLS is enabled for the Ingress, a Secret containing the certificate and key must also be provided:

  apiVersion: v1
  kind: Secret
  metadata:
    name: example-tls
    namespace: foo
  data:
    tls.crt: <base64 encoded cert>
    tls.key: <base64 encoded key>
  type: kubernetes.io/tls
  $
```

It takes a minute or two for the EXTERNAL-IP address of the nginx-ingress-controller service to be populated and allow you to access it with a web browser.

If you encounter this issue: Error: no available release name found

Run this commands:

```bash
kubectl create serviceaccount --namespace kube-system tiller
kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller
kubectl patch deploy --namespace kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}'
```

Then run again the kubectl apply command:

```bash
kubectl apply -f helm-rbac.yaml
```
