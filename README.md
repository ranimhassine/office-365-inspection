# office-365-inspection


A customized version of [365Inspect: Community Edition](https://github.com/soteria-security/365Inspect) by Soteria Security, modified by **Consultim-IT** to generate branded Microsoft 365 security assessment reports.

---

##About

| File | Change |
|------|--------|
| `365InspectConsultimTemplate.html` | New branded dark-theme HTML report template with Consultim-IT logo, gold color palette, stat pills, styled finding cards, and Chart.js visualizations. |

---

## Requirements

Make sure the following PowerShell modules are installed before running:

```powershell
Install-Module -Name ExchangeOnlineManagement -Scope CurrentUser -Force
Install-Module -Name MicrosoftTeams -Scope CurrentUser -Force
Install-Module -Name Microsoft.Graph -Scope CurrentUser -Force
Install-Module -Name PnP.PowerShell -Scope CurrentUser -Force
```

---

## Setup

**1. Clone the original 365Inspect repository:**

```powershell
git clone https://github.com/soteria-security/365Inspect.git
cd 365Inspect
```

**2. Replace / add the Consultim-IT files:**

Copy the following two files into the root of the cloned folder (same level as `365InspectDefaultTemplate.html`):

- `365Inspect.ps1` ← replaces the original
- `365InspectConsultimTemplate.html` ← new branded template

Your folder should look like this:

```
365Inspect/
├── 365Inspect.ps1                    ✅ replaced
├── 365InspectConsultimTemplate.html  ✅ new
├── 365InspectDefaultTemplate.html
├── Inspectors/
├── Write-ErrorLog.ps1
└── ...
```

> ⚠️ You must run the script **from inside this folder**. It uses relative paths to load inspectors and helper scripts.

---

## How to Run

### Step 1 — Open PowerShell and navigate to the folder

```powershell
cd "C:\Path\To\365Inspect"
```

### Step 2 — Allow script execution for this session

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
```

### Step 3 — Pre-authenticate to all required services

This step is required every time you open a new PowerShell window.

```powershell
# Connect to Microsoft Graph
Connect-MgGraph -Scopes "AuditLog.Read.All","Reports.Read.All","Policy.Read.All","Directory.Read.All","Organization.Read.All","User.Read.All","UserAuthenticationMethod.Read.All"

# Connect to Exchange Online (use -Device to avoid WAM broker crash)
Connect-ExchangeOnline -UserPrincipalName admin@yourtenant.onmicrosoft.com -Device

# Connect to Security & Compliance Center
Connect-IPPSSession -UserPrincipalName admin@yourtenant.onmicrosoft.com
```

> When using `-Device`, a code will appear in the terminal. Open [https://microsoft.com/devicelogin](https://microsoft.com/devicelogin) in your browser, enter the code, and complete MFA sign-in.

### Step 4 — Run the script

```powershell
.\365Inspect.ps1 `
  -OutPath "C:\Reports\Output" `
  -UserPrincipalName admin@yourtenant.onmicrosoft.com `
  -Auth ALREADY_AUTHED `
  -pnpPowerShellApplicationId "your-pnp-app-client-id"
```

Replace the following values:

| Parameter | Description |
|-----------|-------------|
| `-OutPath` | Folder where the HTML report will be saved |
| `-UserPrincipalName` | Your admin account email (e.g. `admin@M365x123456.onmicrosoft.com`) |
| `-Auth` | Use `ALREADY_AUTHED` if you pre-authenticated in Steps 3. Use `MFA` to let the script handle auth (may cause WAM errors on some systems) |
| `-pnpPowerShellApplicationId` | Your Azure App Client ID registered for PnP PowerShell |

---

## Output

The script generates a branded HTML report in the `-OutPath` folder:

```
C:\Reports\Output\Report_2026-03-27_11-36-17.html
```

Open the file in any modern browser (Edge, Chrome, Firefox). The report includes:

- 📊 **Stat pills** — at-a-glance Critical / High / Medium / Low / Informational counts
- 📈 **Charts** — findings by severity (bar) and risk distribution by area (doughnut)
- 📋 **Findings summary table** — all findings with severity badges and anchor links
- 🗂️ **Finding detail cards** — full details per finding including affected objects, remediation steps, PowerShell examples, and references
- 📎 **Appendix** — list of all inspectors executed

---

## Troubleshooting

### Script crashes with WAM / NullReferenceException
This is a known Windows Authentication Manager (WAM) broker bug with `Connect-IPPSSession`. Fix:

```powershell
Set-MgGraphOption -DisableLoginByWAM $true
Connect-IPPSSession -UserPrincipalName admin@yourtenant.onmicrosoft.com -Device
```

### "Authentication needed. Please call Connect-MgGraph"
Your Graph session expired or was not started. Re-run Step 3 above before launching the script, and always use `-Auth ALREADY_AUTHED`.

### Report opens blank
The template file is missing from the script folder. Make sure `365InspectConsultimTemplate.html` is in the **same directory** as `365Inspect.ps1`.

### Module not found errors
Install missing modules with `-Scope CurrentUser` (no admin rights needed):

```powershell
Install-Module -Name ExchangeOnlineManagement -Scope CurrentUser -Force -AllowClobber
Install-Module -Name MicrosoftTeams -Scope CurrentUser -Force -AllowClobber
```

### Execution policy error
Run this at the start of every new PowerShell session:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
```

---

## PnP PowerShell App Registration

To use SharePoint inspectors, you need to register an Azure AD application for PnP PowerShell:

1. Follow the official guide: [https://pnp.github.io/powershell/articles/registerapplication.html](https://pnp.github.io/powershell/articles/registerapplication.html)
2. Copy the **Application (Client) ID** from the registered app
3. Pass it as `-pnpPowerShellApplicationId` when running the script

---

## Credits

- Original tool: [365Inspect by Soteria Security](https://github.com/soteria-security/365Inspect)
- Branding and modifications: **Consultim-IT**
