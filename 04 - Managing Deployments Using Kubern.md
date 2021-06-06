# 04 - Managing Deployments Using Kubernetes Engine
## Overview
Dev Ops practices will regularly make use of multiple deployments to manage application deployment scenarios such as "Continuous Deployment", "Blue-Green Deployments", "Canary Deployments" and more. This lab is to provide practice in scaling and managing containers so you can accomplish these common scenarios where multiple heterogeneous deployments are being used.

#### What you'll do
* Practice with kubectl tool
* Create deployment yaml files
* Launch, update, and scale deployments
* Practice with updating deployments and deployment styles
<br/>

#### Prerequisites
* Before taking this lab, you should have taken at least the labs Introduction to Docker and Hello Node Kubernetes
* Linux System Administration skills
* Dev Ops theory: concepts of continuous deployment
<br/>

## Introduction to deployments
Heterogeneous deployments typically involve connecting two or more distinct infrastructure environments or regions to address a specific technical or operational need. Heterogeneous deployments are called "hybrid", "multi-cloud", or "public-private", depending upon the specifics of the deployment. For the purposes of this lab, heterogeneous deployments include those that span regions within a single cloud environment, multiple public cloud environments (multi-cloud), or a combination of on-premises and public cloud environments (hybrid or public-private).

Various business and technical challenges can arise in deployments that are limited to a single environment or region:

* **Maxed out resources**: In any single environment, particularly in on-premises environments, you might not have the compute, networking, and storage resources to meet your production needs.<br/>

* **Limited geographic reach**: Deployments in a single environment require people who are geographically distant from one another to access one deployment. Their traffic might travel around the world to a central location.<br/>

* **Limited availability**: Web-scale traffic patterns challenge applications to remain fault-tolerant and resilient.<br/>

* **Vendor lock-in**: Vendor-level platform and infrastructure abstractions can prevent you from porting applications.<br/>

* **Inflexible resources**: Your resources might be limited to a particular set of compute, storage, or networking offerings.<br/>

Heterogeneous deployments can help address these challenges, but they must be architected using programmatic and deterministic processes and procedures. One-off or ad-hoc deployment procedures can cause deployments or processes to be brittle and intolerant of failures. Ad-hoc processes can lose data or drop traffic. Good deployment processes must be repeatable and use proven approaches for managing provisioning, configuration, and maintenance.

Three common scenarios for heterogeneous deployment are multi-cloud deployments, fronting on-premises data, and continuous integration/continuous delivery (CI/CD) processes.

The following exercises practice some common use cases for heterogeneous deployments, along with well-architected approaches using Kubernetes and other infrastructure resources to accomplish them.

## Learn about the deployment object
`explain` 指令可以顯示 Deployment object 細節，我們可以透過這個指令來了解 Deployment object 的結構與屬性代表的意思。
```shell
kubectl explain deployment
```
Expected output：
```shell
KIND:     Deployment
VERSION:  apps/v1

DESCRIPTION:
     Deployment enables declarative updates for Pods and ReplicaSets.

FIELDS:
   apiVersion   <string>
     APIVersion defines the versioned schema of this representation of an
     object. Servers should convert recognized schemas to the latest internal
     value, and may reject unrecognized values. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources

   kind <string>
     Kind is a string value representing the REST resource this object
     represents. Servers may infer this from the endpoint the client submits
     requests to. Cannot be updated. In CamelCase. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds

   metadata     <Object>
     Standard object metadata.

   spec <Object>
     Specification of the desired behavior of the Deployment.

   status       <Object>
     Most recently observed status of the Deployment.
```

也可以使用下面這個指令來檢查所有的屬性。
```shell
kubectl explain deployment --recursive
```

而這個指令可以看到該屬性的說明。
```shell
kubectl explain deployment.metadata.name
```
Expected output：
```shell
KIND:     Deployment
VERSION:  apps/v1

FIELD:    name <string>

DESCRIPTION:
     Name must be unique within a namespace. Is required when creating
     resources, although some resources may allow a client to request the
     generation of an appropriate name automatically. Name is primarily intended
     for creation idempotence and configuration definition. Cannot be updated.
     More info: http://kubernetes.io/docs/user-guide/identifiers#names
```

## Create a deployment
首先編輯 `deployments/auth.yaml` 設定檔，並用 `i` 開啟 INSERT 模式。
```shell
vi deployments/auth.yaml
```
<br/>

將 `image` 部署檔內容更改後儲存，要先按 `<Esc>` 退出 INSERT 模式，再使用 `:wq` 和 `<Enter>` 儲存並退出檔案編輯模式：
```
...
containers:
- name: auth
  image: "kelseyhightower/auth:1.0.0"
...
```
<br/>

再來檢查部署檔：
```shell
cat deployments/auth.yaml
```
Expected output：
```shell
apiVersion: apps/v1
kind: Deployment
metadata:
  name: auth
spec:
  replicas: 1
  selector:
    matchLabels:
      app: auth
  template:
    metadata:
      labels:
        app: auth
        track: stable
    spec:
      containers:
        - name: auth
          image: "kelseyhightower/auth:1.0.0"
          ports:
            - name: http
              containerPort: 80
            - name: health
              containerPort: 81
          resources:
            limits:
              cpu: 0.2
              memory: "10Mi"
          livenessProbe:
            httpGet:
              path: /healthz
              port: 81
              scheme: HTTP
            initialDelaySeconds: 5
            periodSeconds: 15
            timeoutSeconds: 5
          readinessProbe:
            httpGet:
              path: /readiness
              port: 81
              scheme: HTTP
            initialDelaySeconds: 5
            timeoutSeconds: 1
```
在這個檔案內容中有一個 `replicas` 屬性，且值為1。這是因為我們在使用 `kubectl create` 建立部署檔時，同時也會建立一個相對應的 pod；所以也就是說，我們可以透過 `replicas` 屬性值來設定 pods 的數量。

我們可以透過下面的指令建立部署檔以及查看 deployments。
```shell
kubectl create -f deployments/auth.yaml
kubectl get deployments
```
Expected output：
```shell
NAME   READY   UP-TO-DATE   AVAILABLE   AGE
auth   1/1     1            1           90s
```

## 來源
* https://google.qwiklabs.com/focuses/639?parent=catalog

## 參考