# Getting Started with Cloud Shell and gcloud
## Overview
Cloudd Shell 提供了一些指令讓使用者可以對 GCP 的主機進行資源的計算。Cloud Shell 是基於 Debian ( 精簡的 Linux 發行版 ) 的一種虛擬機，虛擬機上預設會有 5GB 的持久根目錄，可以讓使用者輕鬆管理 GCP 上的專案與資源。而 `gcloud` 指令工具和其他的公用程式已經事先在 Cloud Shell 中安裝，讓我們在使用時可以快速地啟動及運行。

## Prerequisites
熟悉一些 Linux 相關編輯器指令如 `vim`、`emacs` 或 `nano`。

## Task 1: Configure your environment
在本章節中可以了解如何去調整開發環境的各個方面。

#### Understanding regions and zones
Google Compoute Engine 資源位於區域 ( region ) 或區塊中 ( zone )。資源可以在區域 ( region ) 中被運行，每個 region 中包含一個到多個區塊 ( zone )。例如下面這張圖，在 region `us-central1` 指的是 Central US 這塊區域，這塊區域中有數個 zone：`us-central1-a`、`us-central1-b`、`us-central1-c` 以及 `us-central1-f`。<br/>

![](/images/4-1.png)

那些在 zone 中的資源被稱為**區域資源 ( zonal resources )**。VM ( virtual machine ) 實例和持久性區塊儲存空間 ( persistence disks ) 位於同一個區塊 ( zone ) 中。如果要將 persistence disks 連接到虛擬機實例，則兩個資源必須位於同一區域中 ( same zone )。同樣，如果要為實例分配靜態 IP 位址，則兩個資源的實例也必須與靜態 IP 位址在同一個區域 ( region )。

> 相關文件：[Regions and Zones documentation](https://cloud.google.com/compute/docs/regions-zones/)

>  Google Compute Engine 可以讓使用者在 Google 基礎設施上建立以及運行虛擬機，persistence disks 則是虛擬機上的主要儲存空間，就像是實體硬碟一樣。

首先可以用下面兩個指令來查看當前默認區域 ( default region ) 和區塊設置 ( zone settings )：
```shell
gcloud config get-value compute/zone
------------------------------------------------
Your active configuration is: [cloudshell-28573]
(unset)
```
```shell
gcloud config get-value compute/region
-----------------------------------------------
Your active configuration is: [cloudshell-28573]
(unset)
```

如果 `google-compute-default-region` 或 `google-compute-default-zone ` 返回的結果出現 (unset)，表示沒有默認的區塊和區域。

#### Identify your default region and zone
1. 首先複製 project ID，project ID 可以在下面這個地方找到，或是使用指令顯示。
  * 在 Google Cloud Console 中，在信息中心的 Project info 下。（點擊導航菜單 &rarr; 主頁 &rarr; 儀表板 ）
  <br/>

  * 使用指令：
    ```shell
    # command 1
    gcloud config list project

    # command 2
    gcloud compute project-info describe
    ```
    <br/>

2. 在 Cloud Shell 中，執行 `gcloud` 指令，並把 `<your_project_ID>` 替換成自己的 project ID。
    ```shell
    gcloud compute project-info describe --project <your_project_ID>
    ----------------------------------------------------------------
    commonInstanceMetadata:
      fingerprint: K-glf5h9X6Y=
      items:
      - key: ssh-keys
        value: student-03-e0a497f02ef5:ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDcTQIMvytSPwK/yuGygZ+Kv89d5zL3nZeaz0+Za18zUAo+hbNCflRQ32xaDsrQdmXKvUhN3W8r5+/ObYq/T3ZUmKQYjczti3i0wpqiBx1b0lQm8J8aPpFPQh8s+Vusxnh+7mUomOxkKmMuNv65FPVVk+Y4zF+3M7K6aGkZcr2KE+ycwQpr/IislV7YQuvB6tux5uUCb0Uou5vvaoiIgbDbuhfjbIdPhugmUi98a4WOpbrrzp9VbgmdiwyEIJpx2uZtYb+8ChuKGCLUvRUIWf+6cxqwrX81zAAx5ZndeWl0T7gTnGLqyqznv5uSWGqHdg8PoPUNHS0k9UtzgT3u1aJF
          student-03-e0a497f02ef5@qwiklabs.net
      - key: enable-oslogin
        value: 'true'
      - key: google-compute-default-zone
        value: us-central1-a
      - key: google-compute-default-region
        value: us-central1
      kind: compute#metadata
    creationTimestamp: '2021-06-10T08:32:35.909-07:00'
    defaultNetworkTier: PREMIUM
    defaultServiceAccount: 273643412135-compute@developer.gserviceaccount.com
    id: '3479649671665651324'
    kind: compute#project
    name: qwiklabs-gcp-02-df16550b7731
    quotas:
    - limit: 1000.0
      metric: SNAPSHOTS
      usage: 0.0
    - limit: 5.0
      metric: NETWORKS
      usage: 1.0
    - limit: 100.0
      metric: FIREWALLS
      usage: 4.0
    - limit: 100.0
      metric: IMAGES
      usage: 0.0
    ...
    selfLink: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-df16550b7731
    xpnProjectStatus: UNSPECIFIED_XPN_PROJECT_STATUS
    ```

    在輸出的結果中會顯示 default region 跟 default zone，key 值分別是 `google-compute-default-region` 以及 `google-compute-default-zone`。

    > 如果輸出結果中沒有 `google-compute-default-region`、`google-compute-default-zone` 這兩個 key 值，代表沒有設定 default region 和 default zone。

#### Set environment variables
環境參數可以被用來對環境進行設定，還可以節省寫 API 或可執行腳本的時間。

1. 第一步先創造一個環境變數來儲存 project ID 的值。一樣將指令中的 `<your_project_ID>` 替換成自己的 project ID ( 如果不知道 project ID 的值，可以參考上面的步驟取得 )。
    ```shell
    export PROJECT_ID=<your_project_ID>
    ```
    <br/>

2. 在來建立另外一個變數儲存 zone 的資訊，將指令中的 `<your_zone>` 替換成先前取得的 default zone 的值。
    ```shell
    export ZONE=<your_zone>
    ```
    <br/>

3. 最後執行已下指令確認環境變數是否成功被建立
    ```shell
    echo $PROJECT_ID
    echo $ZONE
    ```
    <br/>

#### Create a virtual machine with the gcloud tool 
###### 使用 gcloud 工具建立虛擬機

1. 使用已下指令建立虛擬機實例 ( virtual machine instance )。
    ```shell
    gcloud compute instances create gcelab2 --machine-type n1-standard-2 --zone $ZONE
    ```
    輸入指令後應該會看到這些訊息被輸出：<br/>
    ![](/images/4-2.png)

    ###### 指令內容
    * `gcloud compute` -- 可以直接透過這個指令用更比 Conpute Engine API 更簡單的格式來管理 Compute Engine 上的資源。
    <br/>

    * `instances create` -- 建立一個新的實例。
    <br/>

    * `gcelab2` -- 虛擬機的名稱。
    <br/>

    * `--machine-type` -- 指定虛擬機類行為 _n1-standard-2_。
    <br/>

    * `--zone` -- 指定虛擬機要在哪個區塊 ( zone ) 上運行。
    <br/>

    另外也可以執行 `gcloud compute instance create --help` 指令查看幫助內容，按 ENTER 或空白鍵可以繼續往下滾動內容，要退出的話按 Q 即可。

#### Explore gcloud commands
1. `gcloud` 工具中也提供了一些簡單的使用指南，可以透過在任何 `gcloud` 指令後面加上 `-h`、 `--help` 或是 `gcloud help` 來檢視，差別在於 `-h` 顯示的說明較為簡略。
    ```shell
    # 1
    gcloud -h

    # 2
    gcloud --help

    # 3
    gcloud help
    ```
    <br/>

2. 想要查看 `gcloud config` 指令的說明內容，可以執行以下指令。同樣的按 ENTER 或是空白線可以繼續往下查看內容，退出則按 Q。下面兩個指令會一樣拿到很長一串的說明╮(╯∀╰)╭。
    ```shell
    # 1
    gcloud config --help

    # 2
    gcloud help config
    ```
    <br/>

3. 查看環境配置列表 ( environment configuration ) 或所有屬性及配置：
    ```shell
    # 查看環境配置列表
    gcloud config list

    # 查看所有屬性及其配置
    gcloud config list --all
    ```
    <br/>

4. 列出所有組件 ( component )，它會顯示在這個 project 中使用的 gcloud 組件。
    ```shell
    gcloud components list
    ```
    <br/>

## Task 2: Install a new component
接下來要安裝一個 gcloud 組件**自動完成模式**，它的功能有點像是在編輯器內撰寫程式碼時，會出現自動提示，並且可以在提示窗格 ( pane ) 中選擇程式片段 ( snippets ) 來幫助我們更快的完成一句指令。我們可以在下拉選單中自動完成 ( auto-complete ) 某些靜態訊息，例如指令、子命令名稱 ( sub-command names )、標誌名稱 ( flag names ) 或枚舉標誌值 ( enumerated flag values )。

#### Auto-complete mode
1. 安裝測試版套件：
    ```shell
    sudo apt-get install google-cloud-sdk
    ```
    <br/>

2. 啟用 `gcloud interactive` 模式。
    ```shell
    gcloud beta interactive
    ```

    完成後可以看到有一行指令顯示在 Cloud Shell 最下方，按下 F2 互動模式切換成 ON 或是 OFF。   在互動模式下，按下 TAB 來完成片段指令的選擇。如果出現下拉選單，先按下 TAB 進入下拉選單，在用空白鍵選擇要的選項。
    <br/>

3. 最後嘗試在自動完成模式下完成已下指令，記得將 `<your_vm>` 替換成前面建立的虛擬機實例名稱。
    ```shell
    gcloud compute instances describe <your_vm>
    ```
    <br/>

4. 要退出互動模式的話可以執行 `exit` 指令。
    ```shell
    exit
    ```
    <br/>

## Task 3: Connect to your VM instance with SSH
`gcloud compute ssh` 指令可以在虛擬機實例外多包裝一層 SSH，負責用來身分驗證 ( authentication ) 及實例名稱與 IP 位址間的映射。

1. 要使用 SSH 的話，需要先將 SSH 連接到 VM。
    ```shell
    gcloud compute ssh gcelab2 --zone $ZONE
    ---------------------------------------
    WARNING: The public SSH key file for gcloud does not exist.
    WARNING: The private SSH key file for gcloud does not exist.
    WARNING: You do not have an SSH key for gcloud.
    WARNING: [/usr/bin/ssh-keygen] will be executed to generate a key.
    This tool needs to create the directory
    [/home/gcpstaging306_student/.ssh] before being able to generate SSH Keys.

    Do you want to continue? (Y/n)
    ```
    <br/>

2. 輸入 Y 繼續動作。
    ```shell  
    Generating public/private rsa key pair.
    Enter passphrase (empty for no passphrase)
    ```
    <br/>

3. 不設定密碼的話直接按 ENTER 繼續。
4. 現在由於不需要執行任何動作，所以要斷開與 SSH 的連接並退出遠端 shell，回到原本專案的 shell。
    ```shell
    exit
    ```
    <br/>

## Task 4: Use the Home directory
主目錄中的內容會持續存在，即使當下開了多個 Cloud Shell session，或是 VM 被終止又重新啟動的情況下也是，當中的內容會一直被保存在裡面。

1. 更改當前所在的工作目錄。
    ```shell
    cd $HOME
    ```
    <br/>

2. 使用 `vi` 打開編輯器，編輯 `.bashrc` 配置文件，要退出編輯器的話先按 ESC，輸入 `:wq` 並按下 ENTER 即可。
    ```shell
    vi ./.bashrc
    ```
    <br/>

## Test your understanding
【多選題】<br/>
![](/images/4-3.png)

## 來源
* https://google.qwiklabs.com/focuses/563?parent=catalog

## 參考
* https://tn710617.github.io/zh-tw/createAPersistentDisk/