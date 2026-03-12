# 🛠️ MDI 自動化配置：核心操作清單

本文件彙整 Microsoft Defender for Identity (MDI) 的自動化配置步驟。

---

## 1. 準備工作 (安裝工具)
在管理機（或 DC）上執行，確保模組就緒：

```powershell
# 安裝 MDI 專用模組與網域管理工具
Install-Module -Name DefenderForIdentity -Force
Install-WindowsFeature -Name GPMC, RSAT-AD-PowerShell -IncludeManagementTools

# 生成報告 (將 internal\你的服務帳號 替換為實際使用的帳號)
New-MDIConfigurationReport -Path "C:\Reports" -Mode Domain -Identity "internal\你的服務帳號" -OpenHtmlReport
'''

### 自動化原則部署 (GPO & AD Object)

| 任務目標 | PowerShell 指令 | 關鍵說明 |
| :--- | :--- | :--- |
| 建立/套用 GPO | `Set-MDIConfiguration -GpoName "MDI_Config_GPO" -Mode Domain -Configuration All` | 自動開啟進階稽核原則 |
| 稽核權限(SACL) | `Set-MDIConfiguration -Mode Domain -Configuration DomainObjectAuditing -Identity "帳號"` | 監控敏感物件變更 |
| 回收筒與容器 | `Set-MDIConfiguration -Mode Domain -Configuration AdRecycleBin, DeletedObjectsContainerPermission -Identity "帳號"` | 賦予讀取刪除物件權限 |
| 遠端 SAM 存取 | `Set-MDIConfiguration -Mode Domain -Configuration RemoteSAM -Identity "帳號"` | 允許掃描本機管理員群組 |

---

### 4. 稽核原則關鍵對照表

#### A. 進階稽核原則
* **路徑**：`電腦設定 > 策略 > Windows 設定 > 安全性設定 > 進階稽核原則配置`
* **要求**：確保「帳戶登入」、「帳戶管理」、「DS 存取」皆設為 **成功與失敗**。

#### B. NTLM 稽核與流量

| 設定項目 | 建議值 |
| :--- | :--- |
| 限制 NTLM：在此網域中稽核 NTLM 驗證 | 全部啟用 |
| 限制 NTLM：稽核傳入的 NTLM 流量 | 為所有帳戶啟用稽核 |

# 1. 強制更新原則
gpupdate /force

# 2. 檢查本地稽核原則是否生效
auditpol /get /category:*

# 3. 確認 GPO 是否正確連結到 Domain Controllers OU
$domainDN = (Get-ADDomain).DistinguishedName
Get-GPLink -Target "OU=Domain Controllers,$domainDN" | Select-Object GpoDisplayName, Enabled
