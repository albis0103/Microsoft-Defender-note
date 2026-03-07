# Microsoft-Defender-
# Microsoft 365 Defender 
## 架構
![Microsoft Defender XDR 評估架構](https://learn.microsoft.com/en-us/defender-xdr/media/eval-defender-xdr/m365-defender-eval-architecture.svg)
Microsoft 365 Defender:扮演XDR角色統一MDE, MDI, MDO, MDC EntraID的signal於單一平台管理
各自扮演腳色:KILL CHAIN EXAMPLE
*偵測->武器化->遞送*->**(MDO)**->*利用*->**(MDE)**->安裝->*指揮與控制*->**(MDI & MDC)**->*行動*

- **MDI(Microsoft Defensder for Identity)**
	透過MDI sensor，進行流分析(e.g. K8s, DNS, PRC, SMB, ..etc)與入口網站監控以確認身分，偵測橫向移動或暴力破解等駭客攻擊。MDI Sensor:由EntraID扮演，安裝於DC(Domain Controller)或ADFS server上的代理程式。

**AD延伸之服務**
ADDS:儲存USER帳號,群組原則(GPO)與電腦物件
ADFS:達成跨平台跨組織單一帳號登入(SSO)
ADCS:類似Kerberos架構中的AS負責發放憑證給AD內受信任的實體
*EntraID會trust ADCS發放的憑證*
部屬: 
if 有MDE deploy sensor V3
if !MDE deploy sensor V2
設定window事件稽核
[window 事件稽核](https://learn.microsoft.com/en-us/defender-for-identity/deploy/configure-windows-event-collection#configure-ntlm-auditing)
Okta:雲端身分與存取權限管理平台， 管理高價值身份，包括特權帳號與 API 令牌，因此，Okta 經常成為濫用或攻擊的目標。
核心功能:1.SSO(single sign-on) 2.MFA 3.帳號生命週期管理 (Lifecycle Management) 4.通用目錄 (Universal Directory)

OKTA +　EntraID + AD
AD:負責地端（On-premises）控制。管理辦公室內的 Windows 電腦登入、印表機、檔案伺服器（File Server）以及傳統的 ERP 系統。
Okta:負責SaaS整合，提供SSO
ExtraID(原Azure AD)負責Microsoft 365，整合Teams、Outlook 或 Azure 雲端資源

- MDE(Microsoft Defensder for Endpoint)
四大特性:
	1.Prevent:用N.G.protection, ASR減少威脅
  2.Detect:Advance Hunting提threat
  3.Investigate:視覺化kill chain
  4.Respond:隔離設備，終止process
 核心功能:
核心功能:
	1.API:自動整合
	2.ASR(Attack Surface Reduction):Closs 非必要的port或背景程式
	3.自動調查與修復
	4.EDR:偵測,記錄,回傳端點行為(Advaneced Hunting 可找繞過預防機制的潛在入侵姬路)
  5.Secure Score:用於評估組織內安全狀態
 note:ASR為OS層級的保護(硬體，網路，應用，web，裝置，資料夾存取，漏洞，惡意軟體)
方案:
- 	MD for Business:小型org
-   EDR p1:端點保護(e.g.N.G.protect,ASR..)
-   EDR p2:進階EDR,自動修復, Advance hunting
支援MACos(Intune Based, jamf, 其他MDM, 手動下載需手動運許FDA)，Linux(適用server端EDRp1 or EDRp2, 操作指令mdapt health(檢查onboarded)/scan/thread list)，Android & ios (行動版專注web保護，ios無法scan 系統檔案)
Threat intelligent
	:Microsoft 擁有全球數十億裝置的安全遙測資料（來自 Windows、Azure、Office 365、Xbox、Bing 等）透過 AI 與安全專家分析，形成 威脅情報 
Threat intelligent +　ＭＤＥ
：威脅情報pass to MDE端點防護不只是靠本地規則，而是利用全球威脅資料庫，防禦零時差攻擊。
MDE +　MDI
:MDE 可偵測 Pass-the-Hash、Pass-the-Ticket、Kerberoasting 等攻擊行為。可搭配 Defender for Identity 監控 AD Domain Controller。
note:
**pass the hash**
	:攻擊者取得系統權限從LSASS(local security authority subsystem sevice)讀取NTLM hash value(window將密碼轉成的hash value)
**pass the ticket**
	:竊取keberos的ticket已伪造身分
**Kerberoasting**
	:取得TGS後，離線破解TGS取的密碼
**Intune**:microsoft的MDM方案，支援所有OS設備，透過configuration profile 把 onbroading封包與安全規則推到端點，設定防毒原則,ASR,EDR..
note:onbroad:代表設備加入MDM管理
**MDE部屬方法**
step:
	1.Start the pilot:準備，確認授權與架構(e.g.EDR p1\p2)
  2.pilot:選少量具代表性設備，涵蓋不同OS
  3.full deployment:，Intune or SCCM大規模部屬，onbroad測試
![Microsoft XDR](https://learn.microsoft.com/en-us/defender-xdr/media/zero-trust-with-microsoft-365-defender/m365-zero-trust-architecture-defender.png#lightbox)
- MDO(Microsoft Defensder for Office 365)
- MDC(Microsoft Defensder for Cloud APPs)
- Microsoft EntraID Protection
