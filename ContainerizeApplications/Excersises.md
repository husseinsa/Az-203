# Azure 203 - Create Containerized Solutions

## Overview



## Prerequisites

1. Tools and Setup
   1. Install Visual Studio Code - [Download](https://code.visualstudio.com/)
   2. Install Docker 
2. Azure Cli [Download](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)
3. Kubectl 
4. Provisioned AKS instance
5. Provisioned ACR instance

##  LAB 01: Prepare and Dockerize your Application Locally

### 1 - Get Application Code

1. Make sure you have you checked the pre-requites section
2. Clone the source code
   ```
   git clone https://github.com/Azure-Samples/azure-voting-app-redis.git
   ```
3. Change into the cloned directory
    ```
    cd azure-voting-app-redis
    ```

### 2 - Create container images

1. Start the application by running the following commadn in CMD or bash 
    ```
    docker-compose up -d
    ```
2. To see the three created images, run the command `docker images` 
    ```
    $ docker images

    REPOSITORY                   TAG        IMAGE ID            CREATED             SIZE
    azure-vote-front             latest     9cc914e25834        40 seconds ago      694MB
    redis                        latest     a1b99da73d05        7 days ago          106MB
    tiangolo/uwsgi-nginx-flask   flask      788ca94b2313        9 months ago        694MB
    ```     

3. Run the `docker ps` command to see the running containers.
```
    $ docker ps
    CONTAINER ID        IMAGE             COMMAND                  CREATED             STATUS              PORTS                           NAMES
    82411933e8f9        azure-vote-front  "/usr/bin/supervisord"   57 seconds ago      Up 30 seconds       443/tcp, 0.0.0.0:8080->80/tcp   azure-vote-front
    b68fed4b66b6        redis             "docker-entrypoint..."   57 seconds ago      Up 30 seconds       0.0.0.0:6379->6379/tcp          azure-vote-back
```
### 3 - Test the application localy

1. To see the running application, enter `http://localhost:8080` in a local web browser.

2. Stop and remove the container instances and resources with the `docker-compose down` command


##  LAB 02: Tag and Push Images to ACR

### 1 - Log in to the container registry

1.  Login to you Azure Account
    ```
    az login
    ```
2.  Use the `az acr login` command and provide the unique name given to your container registry 
    ```
    az acr login --name <acrName>
    ```

### 2 - Tag the container image


1. Tag your local images with the `acrloginServer` address of the container registry. To indicate the image version, add :v1 to the end of the image name:
    ```
    docker tag azure-vote-front <acrLoginServer>/azure-vote-front:v1
    ```
2. To verify the tags are applied, run the command `docker images` 
   
### 3 - Push images to ACR

1. Use docker push and provide your own `acrLoginServer address` for the image name as follows:
    ```
    docker push <acrLoginServer>/azure-vote-front:v1
    ```
2. Wait some time for the image to be pushed


### 3 - Verify the images are in Registry
1. Display list of images that were pushed to the registry. Use the `az acr repository list` command. Provide your own `acrName` as follows:
    ```
    az acr repository list --name <acrName> --output table
    ```
2. Or You can go to Azure Portal. Then, Navigate to your Container Registry, Under Services -> Select Repositories. Then, select your repository `azure-vote-front`, to see the the image with the tag specified.




##  LAB 03: Run applicaton on AKS

### 1 - Modify the Yaml file

1.  Navigate to the source code which you downloaded in the first lab. Open the kubernetes manifest file `azure-vote-all-in-one-redis.yaml`

2.  To deploy the application, you must update the image name in the Yaml file to include the ACR login server name.

3. Replace `microsoft` with your ACR login server name. The following example shows the default image name:
    ```
    containers:
    - name: azure-vote-front
      image: microsoft/azure-vote-front:v1
    ```

    The new yaml file will look like:
    ```
    containers:
    - name: azure-vote-front
      image: <acrName>.azurecr.io/azure-vote-front:v1
    ```



### 2 - Deploy and Run Application on Kubernetes

1.  Connect to your Kubernetes cluster, use the `az aks get-credentials` command.
    ```
    az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
    ```
2. To verify the connection to your cluster, run the `kubectl get nodes` command. This will show the nodes in your cluster.

3. Use the  `kubectl apply` commnad to parse the yaml file and create the required Kubernetes object.
    ```
    kubectl apply -f azure-vote-all-in-one-redis.yaml
    ```
4. List existing services 
   ```
   kubectl get services
   ```
5. Show the service details  
   ```
    kubectl describe service azure-vote-front
    ```
6. To see the application in action, open a web browser to the external IP address (`EXTERNAL-IP`) of your service 

7. 4. Check the pods that are created
    ```
    kubectl get pods
    ```

8. Run the desribe command to investigate why you cannot browse the application
   ```
   kubectl describe pod <your-front-end-pod-name>
   ```
9. Check the pod events in the previous step to understand the reason behind the problem :)

10. Fix it, then run the command `kubectl get pods`. Validate your pod is running.

11. Repeat step `6` to browse your application.

12. Use the following kubectl command to create the Kubernetes secret. Replace the parameter in the commnd inside <>
  ```
  kubectl create secret docker-registry acr-auth --docker-server <acr-login-server> --docker-username <service-principal-ID> --docker-password <service-principal-password> --docker-email <email-address>
  ```
13. You can now use the Kubernetes secret in pod deployments by specifying its name (in this case, "acr-auth") in the imagePullSecrets parameter. As in:
```
spec:
      containers:
      - name: <your-container-name>
        image: <your-image>
      imagePullSecrets:
      - name: acr-auth
```
14. Repeat step `3`. Then Repeat step `6`




