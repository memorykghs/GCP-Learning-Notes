# 02 - Kerbernates Engine
## Task 1: Set a default compute zone
Your compute zone is an approximate regional location in which your clusters and their resources live. For example, us-central1-a is a zone in the us-central1 region.

To set your default compute zone to `us-central1-a`, start a new session in Cloud Shell, and run the following command:
```shell
gcloud config set compute/zone us-central1-a
```

Expected output:
```shell
Updated property [compute/zone].
```

## Task 2: Create a GKE cluster
A cluster consists of at least one cluster master machine and multiple worker machines called nodes. Nodes are Compute Engine virtual machine (VM) instances that run the Kubernetes processes necessary to make them part of the cluster.

> Note: Cluster names must start with a letter and end with an alphanumeric, and cannot be longer than 40 characters.

To create a cluster, run the following command, replacing `[CLUSTER-NAME]` with the name you choose for the cluster (for example:my-cluster).
```shell
gcloud container clusters create [CLUSTER-NAME]
```
You can ignore any warnings in the output. It might take several minutes to finish creating the cluster.

Expected output:
```shell
NAME        LOCATION       ...   NODE_VERSION  NUM_NODES  STATUS
my-cluster  us-central1-a  ...   1.16.13-gke.401  3          RUNNING
```

## Task 3: Get authentication credentials for the cluster
After creating your cluster, you need authentication credentials to interact with it.

To authenticate the cluster, run the following command, replacing [CLUSTER-NAME] with the name of your cluster:
```shell
gcloud container clusters get-credentials [CLUSTER-NAME]
```

Expected output:
```shell
Fetching cluster endpoint and auth data.
kubeconfig entry generated for my-cluster.
```

## Task 4: Deploy an application to the cluster
You can now deploy a containerized application to the cluster. For this lab, you'll run hello-app in your cluster.

GKE uses Kubernetes objects to create and manage your cluster's resources. Kubernetes provides the Deployment object for deploying stateless applications like web servers. Service objects define rules and load balancing for accessing your application from the internet.

1. To create a new Deployment hello-server from the hello-app container image, run the following kubectl create command:
     ```shell
    kubectl create deployment hello-server --image=gcr.io/google-samples/hello-app:1.0
    ```

    Expected output:
    ```shell
    deployment.apps/hello-server created
    ```
    This Kubernetes command creates a Deployment object that represents `hello-server`. In this case, `--image` specifies a container image to deploy. The command pulls the example image from a Container Registry bucket. `gcr.io/google-samples/hello-app:1.0` indicates the specific image version to pull. If a version is not specified, the latest version is used.
    <br/>

2. To **create a Kubernetes Service**, which is a Kubernetes resource that lets you expose your application to external traffic, run the following kubectl expose command:
    ```shell
    kubectl expose deployment hello-server --type=LoadBalancer --port 8080
    ```

    In this command:
    * `--port` specifies the port that the container exposes.
    * `type="LoadBalancer"` creates a Compute Engine load balancer for your container.

    Expected output:
    ```shell
    service/hello-server exposed
    ```
    <br/>

3. To inspect the hello-server Service, run kubectl get:
    ```shell
    kubectl get service
    ```
    
    Expected output:
    ```shell
    NAME              TYPE              CLUSTER-IP        EXTERNAL-IP      PORT(S)           AGE
    hello-server      loadBalancer      10.39.244.36      35.202.234.26    8080:31991/TCP    65s
    kubernetes        ClusterIP         10.39.240.1       <none>           433/TCP           5m13s
    ```

    > Note: It might take a minute for an external IP address to be generated. Run the previous command again if the EXTERNAL-IP column status is pending.

    <br/>

4. To view the application from your web browser, open a new tab and enter the following address, replacing [EXTERNAL IP] with the EXTERNAL-IP for hello-server.
    ``` 
    http://[EXTERNAL-IP]:8080
    ```

## Task 5: Deleting the cluster
1. To delete the cluster, run the following command:
    ```shell
    gcloud container clusters delete [CLUSTER-NAME]
    ```
    <br/>

2. When prompted, type `Y` to confirm.
    Deleting the cluster can take a few minutes. For more information on deleted GKE clusters, view the documentation.