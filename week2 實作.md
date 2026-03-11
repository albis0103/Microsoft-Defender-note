03/10 note <br>
安裝DC01(AD)與Client01<br>
安全規則：限制慾ＲＤＰ此ＶＭ需從freedom之網域下設備<br>
要設定DC幫Client DNS解析, 於DCO1 property, forwarder 設定edit 8.8.8.8 or 165.95.1.1 <br>
Azure會將每個VM組態模組化分開ex .ip（公網）, .vnet（內網）,..; 當你刪除 VM 時，Azure 預設不會自動刪除 Public IP (PIP) 和 Network Interface (NIC)。<br>
DC01 -ping-> client01 error<br>
client01 -ping-> DC01 ok<br>
判斷ICMP問題（因為ping protocol 為icmp)<br>
go to 防火牆設定 File and printer sharing 開啟<br>
GPO群組原則存放於DC之sysval(C:\Windows\SYSVOL\sysvol\[DomainName]\Policies), 內容存放欲群體變更或限制的組態設定或規則<br>
是 DC 間同步設定的核心目錄。以後建了 第二台 DC，兩台 DC 會透過 DFSR (Distributed File System Replication) 自動同步這個資料夾裡的 GPO 設定。<br>
GPO只能以OU(Organization Unit)為單位，用ADUC(Activity Directory User and Computer)<br>
預設的 Users 和 Computers 資料夾： 這兩個在 ADUC 裡圖示長得像資料夾的，其實是「容器 (Container)」，它們不能連結 GPO。<br>
正確做法： 你必須在 ADUC 裡按右鍵「新建 > 組織單位 (OU)」，把你的 Client01 移進去，GPO 才會對它生效。<br>
<br>
03/11<br>
hyper-v virtual switch<br>
 - External: internet -> VM
 - Internal: Host -> VM
 - Private:VM -> VM
