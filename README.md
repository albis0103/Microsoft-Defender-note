# Microsoft-Defender-
# Microsoft 365 Defender 
## 架構
![Microsoft Defender XDR 評估架構](https://learn.microsoft.com/en-us/defender-xdr/media/eval-defender-xdr/m365-defender-eval-architecture.svg)
Microsoft 365 Defender:扮演XDR角色統一MDE, MDI, MDO, MDC EntraID的signal於單一平台管理<br>
各自扮演腳色:KILL CHAIN EXAMPLE<br>
*偵測->武器化->遞送*->**(MDO)**->*利用*->**(MDE)**->安裝->*指揮與控制*->**(MDI & MDC)**->*行動*<br>
MDI: 熟記 **Event ID 4768/4769**，這是區分身分攻擊的關鍵。<br>
MDE: 區分 P1 (基礎保護) 與 P2 (進階 EDR/Hunting)。<br>
MDO: Dynamic Delivery 是解決 Safe Attachment 延遲的最佳方案。<br>
MDC: 記API 模式 (非即時/深度) 與 Proxy 模式 (即時/Web Only) 的取捨(Web Only 的限制，MDC 搭配 MDE 收集端點 Log解決shadow IT)。<br>

Sentinel: SIEM = 大腦 (看)；SOAR = 手腳 (動)。
- **MDI(Microsoft Defensder for Identity)**
	主要是監控地端(on-premises)AD 的流量。<br>
Sensor 是安裝在 地端 Domain Controller (DC) 或 AD FS 上。進行流分析(e.g. K8s, DNS, PRC, SMB, ..etc)與入口網站監控以確認身分，偵測橫向移動或暴力破解等駭客攻擊。<br>
- **Entra ID Protection(cloud)**
  :監控的是雲端的登入行為（例如：異常登入位置、洩漏的認證）。<br>
當攻擊者從地端 AD 橫向移動到雲端（例如透過 ADFS 進行 Golden Ticket 攻擊），MDI 與 Entra ID Protection 的訊號會彙整，呈現一條跨越雲地的完整攻擊線。<br>

**AD延伸之服務**<br>
ADDS:儲存USER帳號,群組原則(GPO)與電腦物件<br>
ADFS:達成跨平台跨組織單一帳號登入(SSO)<br>
ADCS:類似Kerberos架構中的AS負責發放憑證給AD內受信任的實體<br>
*EntraID會trust ADCS發放的憑證*<br>
部屬: <br>
if 有MDE deploy sensor V3<br>
if !MDE deploy sensor V2<br>
設定window事件稽核<br>
[window 事件稽核](https://learn.microsoft.com/en-us/defender-for-identity/deploy/configure-windows-event-collection#configure-ntlm-auditing)
AD 稽核重點<br>
- logon events:監控 Event ID 4624 (登入成功) 4625 (登入失敗)。Logon Type 3 (網路登入) 與 Logon Type 10 (遠端桌面)。<br>
- account management:監控 4720 (建立使用者)、4728 (加入安全性群組，尤其是 Domain Admins)。
- Kerberos 服務ticket: 監控 4768 (TGT 請求;ticket Granting Ticket, 給AS端) 與 4769 (服務票證請求;Ticket Granting Service, TGS端)，這是防範 Golden Ticket（竊取AS的HASH自己做Ticket) 攻擊的關鍵。<br>
- 物件存取 (Object Access): 針對 File Server，啟動「檔案系統稽核」，監控敏感資料夾的讀取與刪除。<br>
Okta:混合雲版的EntraID 把AD身份轉給各個SaaS（不只微軟）<br>
核心功能:1.SSO(single sign-on)<br> 2.MFA <br>3.帳號生命週期管理 (Lifecycle Management) <br>4.通用目錄 (Universal Directory)<br>

OKTA +　EntraID + AD<br>
AD:負責地端（On-premises）控制。管理辦公室內的 Windows 電腦登入、印表機、檔案伺服器（File Server）以及傳統的 ERP 系統。<br>
Okta:負責SaaS整合，提供SSO<br>
ExtraID(原Azure AD)負責Microsoft 365，整合Teams、Outlook 或 Azure 雲端資源<br>
- MDE(Microsoft Defensder for Endpoint)<br>
四大特性:<br>
  1.Prevent:用N.G.protection, ASR減少威脅<br>
  2.Detect:Advance Hunting提threat<br>
  3.Investigate:視覺化kill chain<br>
  4.Respond:隔離設備，終止process<br>
 核心功能:<br>
	1.API:自動整合<br>
	2.ASR(Attack Surface Reduction):Closs 非必要的port或背景程式<br>
	3.自動調查與修復<br>
	4.EDR:偵測,記錄,回傳端點行為(Advaneced Hunting 可找繞過預防機制的潛在入侵姬路)<br>
  5.Secure Score:用於評估組織內安全狀態<br>
 note:ASR為OS層級的保護(硬體，網路，應用，web，裝置，資料夾存取，漏洞，惡意軟體)<br>
方案:<br>
- 	MD for Business:小型org<br>
-   EDR p1:端點保護(e.g.N.G.protect,ASR..)<br>
-   EDR p2:進階EDR,自動修復, Advance hunting<br>
支援MACos(Intune Based, jamf, 其他MDM, 手動下載需手動運許FDA)，Linux(適用server端EDRp1 or EDRp2, 操作指令mdapt health(檢查onboarded)/scan/thread list)，Android & ios (行動版專注web保護，ios無法scan 系統檔案)<br>
Threat intelligent<br>
	:Microsoft 擁有全球數十億裝置的安全遙測資料（來自 Windows、Azure、Office 365、Xbox、Bing 等）透過 AI 與安全專家分析，形成 威脅情報 <br>
Threat intelligent +　ＭＤＥ<br>
：威脅情報pass to MDE端點防護不只是靠本地規則，而是利用全球威脅資料庫，防禦零時差攻擊。<br>
MDE +　MDI<br>
:MDE 可偵測 Pass-the-Hash、Pass-the-Ticket、Kerberoasting 等攻擊行為。可搭配 Defender for Identity 監控 AD Domain Controller。<br>
[note]<br>
**pass the hash**
	:攻擊者取得系統權限從LSASS(local security authority subsystem sevice)讀取NTLM hash value(window將密碼轉成的hash value)<br>
**pass the ticket**
	:竊取keberos的ticket已伪造身分<br>
**Kerberoasting**
	:取得TGS後，離線破解TGS取的密碼<br>
	e.g.在實際運作中：<br>
MDE 會偵測到某個 Process（如 lsass.exe）被不正常讀取。<br>
MDI 會偵測到網路流量中出現異常的 Kerberos Ticket 請求（TGS Request）。<br>
Microsoft Defender XDR 會將這兩個事件關聯成同一個 Incident (事件)，告訴你：「攻擊者在 A 電腦偷了 Hash，然後正嘗試登入 B 電腦。」<br>
**Intune**:microsoft的MDM方案，支援所有OS設備，透過configuration profile 把 onbroading封包與安全規則推到端點，設定防毒原則,ASR,EDR..<br>
note:onbroad:代表設備加入MDM管理<br>
**MDE部屬方法**
step:<br>
1.Start the pilot:準備，確認授權與架構(e.g.EDR p1\p2)<br>
2.pilot:選少量具代表性設備，涵蓋不同OS<br>
3.full deployment:，Intune or SCCM大規模部屬，onbroad測試<br>
![Microsoft XDR](https://learn.microsoft.com/en-us/defender-xdr/media/zero-trust-with-microsoft-365-defender/m365-zero-trust-architecture-defender.png#lightbox)
- MDO(Microsoft Defensder for Office 365)
**EOP 基礎方案(Exchange Online Protection)**
  ：雲端mail內建保護功能（防惡意程式、反垃圾mail、基礎反釣魚）<br>
  核心技術：連線filter(chick IP 手動加list）、租戶允許封鎖名單(check IP, URL, hash, 寄件者地址, 網域), ZAP(zero-hour Auto purge;將事後發現的惡意郵件自動清除)<br><br>
  **Plan1**
  ：safe link（攔截惡意連結）、safe attachment（避免惡意附件，零時差保護）、real time Detection（透過反釣魚政策alert基礎攔截報告，P1專屬）<br>
note:<br>
：傳統把整封mail(mail+附件)送到sandbox檢測完才給user，太久<br>
Dynamic Deliver：safe attechment的設定，讓user先看mail再給附件<br>
safelink具備影像辨識功能，防止QR code phishing<br>
平台延伸性： 服務描述強調 MDO 不僅保護 Exchange，還包含 SharePoint Online、OneDrive for Business 和 Microsoft Teams。這在 Plan 1 就已經包含。<br><br>
  **Plan2**
  ：safe link、safe attachment、Threat Explorer（除了alert報告，有進階塞選跟批量修復，P2專屬）、AIR(自動調查，提出修補建議，P2專屬）、Attack Stimulation（對員工釣魚演練，P2專屬）<br>
安全文件 (Safe Documents)： 與ＭＤＥ整和，在開啟 Office 文件前自動掃描，防止文件內嵌的威脅，P2專屬）。<br>
[note]priority account protection：VIP帳號(e.g.CEO, CFO)在threat explorer會被加強行為分析避免BEC（商業郵件詐騙）<br>
P2適用<br>
E5 (Enterprise)：大型企業版。<br>
A5 (Academic)：教育版（學校、學術單位）。<br>
GCC G5 (Government Community Cloud)：政府機關雲端版。<br><br>
- MDC(Microsoft Defensder for Cloud APPs)<br>
  **CASB(Cloud Access Security Broker)**(架構層面)：透過reverse proxy，控制user與SaaS間使用，但依賴EntraID（e.g.shadow IT 發現、SaaS使用狀況的可視化、所有SaaS威脅防護，以及資訊保護與合規評估)<br>
  **Discover SaaS application**<br>：分析網路流量log，便是組織用哪些SaaS，解決shadow IT，對應[note]app發現政策。<br>
  **information protection**<br>：自動對檔案夾標籤或變更分享權限，確保雲端App中敏感資訊安全。對應[note]檔案政策<br>
  **SSPM(SaaS Security Posture Management)** <br>
  ：提供SaaS安全狀態可視化，自動掃描SaaS安全性與SaaS供應商所定最佳實務（如 CIS 基準）進行比對給出安全分數(secure score)。<br>
  **Continuous Theat Protection**<br:透過UEBA偵測異常活動（勒索軟體、帳號盜用..)。對應[note]靜態：活動政策，動態：異常檢測政策(UEBA)、惡意軟體檢測<br>
**App to app protection**<br>：透過管理OAuth權限（MDC透過API抓取第三方App申請的Scope)，監控評估第三方App。對應[note]OAuth應用程式政策<br><br>
[note]:UEBA(User and Entity Behavier Analytics):透過機器學習與大數據分析觀察User與Entity(ex.server, router..)的行為異常<br>
  [note]:policy類型<br>
 - 活動政策：監控使用者操作(ex.未授權IP下載檔案）
 - 異常檢測政策：透過UEBA行為分析，找出異常
 - OAuth應用程式政策：審查第三方app請求權限
 - 惡意軟體檢測：掃描cloud storage(ex.OneDrive, ..)惡意檔案
 - 檔案政策：對檔案機密內容（個資、信用卡）執行治理（設定為私人）
 - 訪問政策(Access)：登入階段攔截
 - 會話政策(Session)：控制登入後活動(ex.禁止下載..)
 - app發現政策：使用未經授權SaaS會通知，解決shadowIT<br><br>
**整合 SIEM (Microsoft Sentinel)**
MDC 透過 API 將 MDC Apps整合Microsoft Sentinel (可擴展的雲端原生 SIEM 與 SOAR) ，實現警示與發現資料的集中監控。<br>
SIEM(Security Information and Event Management)<br>：在偵測 (Detect)、調查 (Investigate)階段，收集MDI, MDE, MDC的log進行關聯分析，生成alert
SOAR(Security Orchestration, Automation, and Response)<br>：在回應 (Respond)、復原 (Recover)階段，根據alert自動跨產品動作
- Microsoft EntraID Protection
