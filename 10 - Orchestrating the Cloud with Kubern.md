# 03 Orchestrating the Cloud with Kubernetes
###### 使用 Kubernetes 編排雲
本章將會學習如何：
* 使用 Kubernetes Engine 配置完整的 Kubernetes 集群 ( cluster )。
* 使用 `kubectl` 指令
* 使用 Kubernetes 的部署和服務將應用程序分解為微服務。

## Pods
Pod 是 Kubernetes 中最小的部署單位，Pod 是由一個至多個容器 ( conatainer ) 所組成。如果有多個相互依賴的容器，也可以打包成一個 Pod。

![](/images/3-1.jpg)

在這個例子中，這個 Pod 包含了 monolith 以及 nginx 兩個 conatainer。Kubernets 會在 Pod 建力的同時一併建立某些檔案，並在 Pod 結束時刪除，Pod 中的容器可以自由儲存並使用 Pod 存放在開 volume 內的檔案。

相較於 Docker 中每個容器會擁有各自獨立的 IP，Kubernetes 則是每個在 Pod 中的容器會共享同樣的 IP address 以及 port space ( 網絡命名空 )，這也代表 Pod 中的 container 可以相互溝通。所以，在同一個 Pod 中的 container 會：

* 共享相同的 IP 地址及端口空間
* 共享儲存空間 ( Volumes )



## Creating Pods
Pods 可以透過設定配置文件來建立。
```shell
cat pods/monolith.yaml
```
輸入完指令完後會看到 monolith 的配置文件內容。
```shell
apiVersion: v1
kind: Pod
metadata:
  name: monolith
  labels:
    app: monolith
spec:
  containers:
    - name: monolith
      image: kelseyhightower/monolith:1.0.0
      args:
        - "-http=0.0.0.0:80"
        - "-health=0.0.0.0:81"
        - "-secret=secret"
      ports:
        - name: http
          containerPort: 80
        - name: health
          containerPort: 81
      resources:
        limits:
          cpu: 0.2
          memory: "10Mi"
```

有一些需要注意的是：
* 這個 Pod 由一個 monolith conatainer 組成
* 當容器啟動時，會將這些參數傳遞給容器
* monolith container 現在使用的 http 端口為 80

再來我們透過 Kubernetes 的指令 `kubectl` 來建立 Pod：
```shell
kubectl create -f pods/monolith.yaml
```

接下來使用這個指令列出在默認的命名空間中運行的所有 Pods：
```shell
kubectl get pods
```

> monolith pod 可能需要花幾秒的時間才能啟動並運行。在運行之前，需要從 Docker Hub 拉取 monolith 的 image。

Pod 運行後可以使用 `kubectl describe` 來獲取 Pod 的更多訊息：
```shell
kubectl describe pods monolith
```
結果會看到關於 monolith 這個 Pod 的信息，包括 Pod 的 IP 位址及是件日誌，這些訊息可以幫助我們在排除故障時確認問題在哪。Kubernetes 透過配置文件創建 Pod，並提供一些指令，在 Pod 運行時可以查看他們的狀態，接下來我們要試著與 Pod 互動。

## Interacting with Pods
在預設的情況下，Pod 會被分配到一個私有的 IP 位址，這個 IP 是不能在叢集外部被訪問到的，所以要透過 `kubectl port-forward` 指令來將本地端的 port 映射到 monolith pod 的端口。

打開第二個 Cloud Shell terminal，執行 `kubectl port-forward` 來設置 port 跳轉端口。
```shell
kubectl port-forward monolith 10080:80
```

然後在原本的 terminal 使用 `curl` 指令與 Pod 互動：
```shell
curl http://127.0.0.1:10080
```
就會看到容器回應個一個字串 "hello"。再來使用 `curl` 指令來看看當請求發送至安全端點時會發生什麼。
```shell
curl http://127.0.0.1:10080/secure
```

但這時候會發現 Cloud Shell 會顯示沒有權限，所以我們要嘗試登入從 monolith Pod 取得授權 token，並用超級密碼 "password" 登入。
```shell
curl -u user http://127.0.0.1:10080/login
```

輸入完密碼後就會得到 JWT token。由於 token 通常都不短，所以我們可以創造一個環境變數來記住這個 token，而主機密碼一樣是 "password"。
```shell
TOKEN=$(curl http://127.0.0.1:10080/login -u user|jq -r '.token')
```

然後再次使用 `curl` 指令訪問安全端點即可。
```shell
curl -H "Authorization: Bearer $TOKEN" http://127.0.0.1:10080/secure
```

接著嘗試使用 `kubectl logs` 命令查看 monolith Pod 的日誌。
```shell
kubectl logs monolith
```

打開第三個終端並使用 `-f` 來查看時實更新的日誌，只要有請求進來這個 Pod，日誌就會更新併紀錄。
```shell
kubectl logs -f monolith
```

所以當在第一個 Cloud Shell 輸入指定向 Pod 發出請求時，在第三個 Cloud Shell 下 `kubectl logs` 指令後會看到日誌被更新。 
```shell
curl http://127.0.0.1:10080
```

而如果想要從容器內進行故障排除時，可以使用 `kubectl exec` 指令，他可以在指定的 Pod 內進行互動測試。
```shell
kubectl exec monolith --stdin --tty -c monolith /bin/sh
```
For example, once you have a shell into the monolith container you can test external connectivity using the ping command:
例如，如果在容器中安裝了一個 shell，就可以使用 `ping` 指令測試外部連接。
```shell
ping -c 3 google.com
```

不過在測試完成後要記得註銷。
```
exit
```
與 Pod 交互就像使用 `kubectl` 命令一樣簡單。如果需要遠程訪問容器，或獲取登錄 shell，Kubernetes 提供了啟動和運行所需的一切。

## Services
Pods 並不是永久存在的，他們可能因為某些原因被啟動或是停止，例如啟動失敗或是就緒檢查，這時候會導致一個問題：如果你想要與一組 Pods 溝通時會發生什麼?由於一個 Pod 會有一個 IP address，當想要與一組 Pods 溝通時，每個 Pod IP 都不一樣，操作上非常不易。

而 Service 正式 Kubernetes 提供解決此問題的方法，Service 為 Pods 提供穩定的端點，讓使用者無須考慮到不同節點的問題就能夠輕鬆操作 Pods。<br/>

![](/images/3-2.jpg)

Service 是 Kubernetes 內定義的抽象物件，主要規範了邏輯上的一群 Pods 以及該如何存取這些 Pods 的規則。那麼，
* 誰會使用 Service?
* 什麼是邏輯上的一群 Pod?
* 存取規則是什麼?
<br/>

![](/images/3-3.png)

#### 誰會使用 Service?
整體來說會有兩個不同的角色來存取 Service，一個是紅色箭頭的路徑，是由外部的 Client 端對 Service 進行存取，取得 Pod 服務。另外一個則是綠色箭頭的路徑，代表同集群 ( cluster ) 中的其他 Pod 也有可能透過 Service 來取得某一個 Pod 的服務。不過兩條路徑的存取方式以及存取的 IP 位址也會有所不同。

#### 什麼是邏輯上的一群 Pod?
簡單來說就是：**帶有相同標籤、做類似事情的一群 Pod**。

> 每個 Pod 本身會帶著一或多個不等的標籤在身上，當 Service 指定存取某些特定的標籤時，Label Selector 會根據 Pod 身上自帶的標籤進行分組，並回傳 Service 所要求的 Pod 資訊。( ★詳見參考 )

上圖中就有三種不同顏色的 Pod，代表叢集內有三種不同的 Pod 群組，當收到使用者請求時，便會將請求送至對應群組內的其中一個 Pod 進行處理。

#### 存取規則是什麼?
就是如何存取該服務的規則，例如 TCP、Port 等等相關規則。

---
Service 會使用標籤來決定要在那些 Pod 上運行，如果 Pod 具有符合的標籤，Pod 就會被 Service 自動存取並公開。而 Service 作為中介曾，避免使用者直接和 Pod 進行連線，除了可以彈性的選擇不同的 Pod 進行服務之外，某種程度上也避免 Port 直接暴露在外面的風險。

> 對於使用者而言，只需要知道請求會被處理，但不需要知道是由誰處理、怎麼處理的。

The level of access a service provides to a set of pods depends on the Service's type. Currently there are three types:

Service 提供對一群 Pods 的訪問級別會依照服務類型大致分成3類：
* `ClusterIP (internal)` -- default 代表這個 Service 只在集群內部可見。
* `NodePort` -- 為集群中的每個節點提供一個外部可訪問的 IP address。
* `LoadBalancer` -- 添加從雲端供應商 ( cloud provider ) 提供的負載均衡器 ( load balancer )，它會將請求從 Service 轉發到 Pod 的節點。

接下來我們將嘗試：
* 建立服務 ( Service )
* 使用標籤選擇器 ( label selector ) 向外部公開一組有限的 Pod

## Create a service
在建立 Service 之前，首要要創建一個可以處理 https 請求的 secure pod。如果當前所在的目錄不是 `~/orchestrate-with-kubernetes/kubernetes`，要記得先 `cd` 下。
```shell
cd ~/orchestrate-with-kubernetes/kubernetes
```

查看 monolith 服務的配置文件：
```shell
cat pods/secure-monolith.yaml
```

利用配置文件建立 monolith pod。
```shell
kubectl create secret generic tls-certs --from-file tls/
kubectl create configmap nginx-proxy-conf --from-file nginx/proxy.conf
kubectl create -f pods/secure-monolith.yaml
```

建立完 secure pod 後向外部公開 monolith pod，一樣先查看 monolith 的配置文件。
```shell
cat services/monolith.yaml
--------------------------
(Output):

kind: Service
apiVersion: v1
metadata:
  name: "monolith"
spec:
  selector:
    app: "monolith"
    secure: "enabled"
  ports:
    - protocol: "TCP"
      port: 443
      targetPort: 443
      nodePort: 31000
  type: NodePort
```

* Service 中會有一個選擇器 ( selector ) 自動，被用來尋找和公開那些帶有某些標籤 ( label ) 的 pod。這裡的標籤有兩個：
`app: monolith` 和 `secure: enabled`。

* 現在還要對外公開節點端口，這樣我們才能將外部的請求從端口 31000 轉到 nginx 端口 ( 443 )。

由於每個 Pod 本身會帶有一個到多個標籤，如何將使用者請求送到正確的 Pod 就必須仰賴管理者設定的標籤是否妥當。例如，今天有 nginx 與 monolith 兩個伺服器在運作，我們希望將流量導向是 nginx，只需要在 Pod 的 `spec.selector` 設定一組 `nginx` 的標籤並在 Service 中定義，Service 就會根據配置黨內所設定的標籤，透過標籤選擇器 ( label selector ) 找到對應的 Pod，在請求進來時將封包送至指定的 Pod 內。

建立Service 之前，先用 `kubectl create` 指令透過 monolith Service 配置文件建立 monolith Service。
```shell
kubectl create -f services/monolith.yaml
----------------------------------------
service/monolith created
```

因為現在已經在 Service 上公開了一個 port，所以有可能會發生其他 app 啟動時也剛好要占用同一個 port 的情況。通常 Kubernetes 會自行處理 port 的分配，不過在本練習中手動選擇了一個 port 號，在後續進配置運行狀況檢查時會較為輕鬆。

使用 `gcloud compute firewall-rules` 指令將公開節點的流量導向 monolith Service。
```shell
gcloud compute firewall-rules create allow-monolith-nodeport \
  --allow=tcp:31000
```

設定完成後代表可以由 cluster 外部訪問 secure pod Service，而不需要透過端口轉發。首先，要先獲得其中一個節點的外部 IP 位址。
```shell
gcloud compute instances list
```

然後使用 `curl` 指令訪問 secure monolith Service。
```shell
curl -k https://<EXTERNAL_IP>:31000
```

這時候會發現請求 timeout，這時候可以利用下面的指令來回答一些問題：
```shell
kubectl get services monolith

kubectl describe services monolith
```

* Questions:
  * 為何無法從 monolith Service 獲得回應?

  * monolith Service 有多少個端點 ( endpoint )?

  * 一個 Pod 需要有哪些標籤才會被 Service 選擇?

以上的問題跟標籤 ( label ) 有關，下面會提到相關的解決方法。

## Adding Labels to Pods
目前的 monolith Service 沒有對外公布的端點，要解決這個問題的其中一個方法是使用 `kubectl get pods` 並在後面加上標籤名稱進行查詢。輸入指令後便可以看到一個清單，列出一些正在運作且帶有 monolith 標籤的 Pods。
```shell
kubectl get pods -l "app=monolith"
```

但如果搜尋的是  `app=monolith` 和 `secure=enabled` 這兩個標籤的 Pod 呢?
```shell
kubectl get pods -l "app=monolith,secure=enabled"
```

此時查詢是沒有結果的，代表目前沒有在運作的 Pod 是同時帶有這兩個標籤的，所以我們需要對 Pod 使用 `kubectl label` 添加 `secure=enabled` 標籤，然後使用 `kubectl get pods` 指令來確認 Pod 的標籤是否被更新。
```shell
kubectl label pods secure-monolith 'secure=enabled'
kubectl get pods secure-monolith --show-labels
```

添加完 monolith pod 的標籤後，查看 monolith 服務上的端點列表。
```shell
kubectl describe services monolith | grep Endpoints
```

最後進行測試。
```shell
gcloud compute instances list
curl -k https://<EXTERNAL_IP>:31000
```

## Deploying Applications with Kubernetes 
###### 使用 Kubernetes 部署應用程序
在這個章節將嘗試在 Kubernetes 上部署應用程序，確保運行的 Pods 數量等於使用者指定的 Pods 需求數量。我們可以在部署的過程中擴展與管理容器。<br/>

![](/images/4-5.jpg)

Deployments 的好處在於對 Pod 的底層細節進行抽象管理。我們可以使用 Replica Sets 去管理 Pod 的啟動與停止。當 Pods 需要被更新或是被增加，抑或是出於某種原因需要重新啟動時，Deployment 會負責處理這塊。來看一個小小的例子：<br/>

![](/images/4-6.jpg)

Pods 的生命週期與建立他們的節 ( Node ) 點有關。在上圖的例子中，Node3 當機了 ( 帶走了一個 Pod )，這時候我們不需要重新在某個 Node 上手動建立一個新的 Pod，而是由 Deployment 在 Node2 上建立新的 Pod 並啟動它。

接下來我們就要使用前面提到跟 Pod 與 Service 有關的知識，讓部署整體應用程序分解為更小的服務。

## Creating Deployments
我們可以把 monolith app 分為三個獨立的部分：
* `auth` -- 為通過身分驗證的使用者產生 JWT token。

* `hello` -- 請求進來時，回應 hello 給使用者。
* `frontend` -- 指將回應回傳給使用者的部分，以本章的例子來說，就是將 hello 回傳給使用者。

建立完 Deployment 之後，我們將對 auth 與 hello 兩個 deployments 內部及外部的婦誤。完成後就能夠像前面使用 monolith 一樣與微服務進行互動，只是現在每個部分都可以獨立的擴展和部署。

先檢視身分驗證的部署配置文件。
```shell
cat deployments/auth.yaml
-----------------------------------------------
apiVersion: apps/v1
kind: Deployment
metadata:
  name: auth
spec:
  selector:
    matchlabels:
      app: auth
  replicas: 1
  template:
    metadata:
      labels:
        app: auth
        track: stable
    spec:
      containers:
        - name: auth
          image: "kelseyhightower/auth:2.0.0"
          ports:
            - name: http
              containerPort: 80
            - name: health
              containerPort: 81
...
```

這時候 Kubernetes 正在使用 2.0.0 版本的身分驗證配置文件創建一個 replica。當使用 `kubectl create` 指令建立身分驗證的部署時，它也會依照部署配置檔中 Replica 的數量建立相對數量的 Pods。( 下一章會提到 )


繼續建立部署對象。
```shell
kubectl create -f deployments/auth.yaml
```

然後為這個 deployment 建立 Service。
```shell
kubectl create -f services/auth.yaml
```

然後建立另外一個 hello app 的 deployment 與 Service。
```shell
kubectl create -f deployments/hello.yaml
kubectl create -f services/hello.yaml
```

公開 frontend Deployment。
```shell
kubectl create configmap nginx-frontend-conf --from-file=nginx/frontend.conf
kubectl create -f deployments/frontend.yaml
kubectl create -f services/frontend.yaml
```

最後我們先獲得 Service 的外部 IP 位址，並使用 `curl` 指令進行互動，就可以獲得回傳訊息 hello。
```shell
kubectl get services frontend
curl -k https://<EXTERNAL-IP>
```

## 來源
* https://google.qwiklabs.com/focuses/557?parent=catalog#

## 參考
* https://tachingchen.com/tw/blog/kubernetes-pod/
* https://tachingchen.com/tw/blog/kubernetes-service/