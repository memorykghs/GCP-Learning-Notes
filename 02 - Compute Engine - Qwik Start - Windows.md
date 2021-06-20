# Compute Engine: Qwik Start - Windows
Compute Engine 可以讓你在 Google infrastructure 上建立並運行 VM，同時提供規模 ( scale )、效能 ( performance ) 及價值 ( value )，讓你可以在 Google infrastructure 上可以輕鬆啟動大型寄堧和集群 ( cluster )。

除了 VM 之外，也可以在 Compute Engine 上運行 Windows 應用程式，並利用 VM instance 的許多優勢，例如可靠的 [storage options](https://cloud.google.com/compute/docs/disks/)、[Google network](https://cloud.google.com/compute/docs/vpc) 的速度和 [Autoscaling ( 自動調整規模 )](https://cloud.google.com/compute/docs/autoscaler/)。

這個章節將嘗試在 Compute Engine 中啟動 Windows Server 實例並使用遠程桌面協議 ( RDP ) 連接到該實例。

## Create a virtual machine instance
1. 進入 Google Cloud Platform 後由左側的**導覽 menu** 點選 &rarr; **Compute Engine** &rarr; **VM Instance** 進入管理實例的畫面，並點選 CREATE INSTANCE 建立實例。<br/>
![](/images/1-1.png)
   <br/>

2. 在機器配置 ( Machine configuration ) 的部分，將Series 設置為 N1。<br/>
   ![](/images/2-1.png)
   <br/>


3. 在 Boot disk 的部分，點選 Change 進行設置。<br/>
   ![](/images/2-2.png)
   <br/>

4. 呈上，選擇 Windows Server 2012 R2 Dtatcnter，然後點擊 Select，其他屬性直接套用預設值即可，不需要更改。<br/>
   ![](/images/2-3.png)
   <br/>

   整體設置內容如下：
   ![](/images/2-4.png)
   <br/>

5. 最後點選 Create 來創建實例。

## Remote Desktop (RDP) into the Windows Server

## Test the status of Windows Startup
回到 Google Cloud Platform 的 VM Instances 介面，應該可以看到 Windows Server 實例，並帶有綠色狀態的圖標 ![](/images/2-8.png)。不過伺服器實例 ( server instance ) 可能需要一段時間才能完成初始化，準備接受 RDP 連接。如果要查看伺服器是否已經準備好，可以在 Cloud Shell 中執行以下指令。
```shell
gcloud compute instances get-serial-port-output instance-1
```

如果有出現提示，輸入 n 並按 Enter。一直重複該命令直到出現下面的內容，表示 Windows Server 已經準備好接受 RDP 連接。
```shell
------------------------------------------------------------
Instance setup finished. instance-1 is ready to use.
------------------------------------------------------------
```

#### RDP into the Windows Server
接下來要設定登入 RDP 的密碼，在 Cloud Shell 中執行以下命令，並將指令中的 `[instance]` 替換成前面建立的 Windows Server 名稱 ( 此案例中為 `instance-1` )、`[username]` 替換成自己登入的使用者名稱。
```shell
gcloud compute reset-windows-password [instance] --zone us-central1-a --user [username]
```

p.s. 使用 `gcloud auth list` 指令可以取得當前登入的使用者名稱。

如果 Cloud Shell 中顯示 `Would you like to set or reset the password for [admin] (Y/n)?`，輸入 Y。而有多種方法可以通過 RDP 連接到伺服器，這邊不會針對所有情況進行說明。而如果使用的不是 Windows 但是瀏覽器依舊是使用 Chrome，可以透過 [Chrome RDP for Google Cloud Platform](https://chrome.google.com/webstore/detail/chrome-rdp-for-google-clo/mpbbnannobiobpnfblimoapbephgifkm) 擴充應用程式，直接從瀏覽器通過 RDP 連接到伺服器。在 Google Cloud Platform 的 VM Instance 介面點選該實例的 Connect 屬性就可以安裝 Chrome RDP 擴充。

![](/images/2-5.png)


## Test your understanding
![](/images/2-6.png)
<br/>

![](/images/2-7.png)
<br/>

## 來源
* https://google.qwiklabs.com/focuses/560?parent=catalog
