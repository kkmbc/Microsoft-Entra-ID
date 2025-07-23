## Download Microsoft Entra ID (formerly Azure AD)

This guide provides step-by-step instructions for installing and configuring Microsoft Entra ID on a Windows, enabling synchronization between your on-premises Active Directory and Azure AD.

Updated: July 12, 2025        
**⬇️ [Download Microsoft Entra ID](*)**

#### System Requirements

* **Operating System**: Windows Server 2016 or later (Windows Server 2022 is recommended).
* **Hardware**: At least 4 GB RAM and a dual-core processor.
* **Software**: Ensure that .NET Framework 4.7.2 or higher is installed on the system.

#### Account Permissions

* **Azure AD**: Requires Global Administrator access.
* **On-Premises AD**: Enterprise Admin credentials are needed.

#### Network and DNS

* DNS name resolution must function properly for both the local AD and Azure AD domains.
* This example uses a non-routable internal domain, but the setup is the same for routable domains such as **`yourcompany.com`**.


### Installation Steps

### Step 1: Prepare the Server

1. **Join the Server to the Domain**
   Confirm that your Windows Server has been added to your existing on-prem Active Directory domain.

2. **Install Required Roles**
   Open **PowerShell with administrative privileges** and execute:

   ```powershell
   Install-WindowsFeature RSAT-ADDS
   ```

3. **Check Time Synchronization**
   Make sure the server’s time settings match those of your domain controllers. Use the following if necessary:

   ```powershell
   w32tm /query /status
   ```

### Step 2: Install Azure AD Connect

1. **Download and Launch the Installer**
   Run the downloaded `AzureEntraID.exe` setup file.

   * If the installer reports a problem with prerequisites (such as missing .NET or insufficient privileges), close it, address the issue, and run it again.

2. **Enable TLS 1.2 for .NET Framework**
   From an elevated PowerShell session, enter:

   ```powershell
   # Enable strong cryptography
   Set-ItemProperty -Path 'HKLM:\SOFTWARE\Microsoft\.NETFramework\v4.0.30319' -Name 'SchUseStrongCrypto' -Value 1
   Set-ItemProperty -Path 'HKLM:\SOFTWARE\WOW6432Node\Microsoft\.NETFramework\v4.0.30319' -Name 'SchUseStrongCrypto' -Value 1
   ```

   **Restart the server** to apply changes.

   Once restarted, run:

   ```powershell
   [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
   ```

   Verify TLS 1.2 is active:

   ```powershell
   [Net.ServicePointManager]::SecurityProtocol
   ```

   You should see `Tls12` in the output.

3. **Begin Setup**

   * Accept the license terms and proceed by clicking **Continue**
   * Choose **Customize** to adjust installation parameters manually (recommended)

4. **Choose Authentication Method**
   Pick one of the following sign-in options:

   * **Password Hash Synchronization** (default)
   * **Pass-through Authentication**
   * **Federation**

   After selecting, click **Next**

### Step 3: Configure Azure AD Connect

1. **Sign in to Azure AD**
   Provide the credentials of a user with **Global Administrator** rights.

2. **Connect to Local Active Directory**
   Enter your **Enterprise Admin** account for the local AD.
   If using a `.local` or other private domain suffix, a warning might appear — it's safe to proceed.

3. **Select Synchronization Options**
   Choose one of the following methods to define which users/groups to sync:

   * **Sync all users and groups**
   * **Filter by Organizational Unit (OU)**
   * **Filter by attribute**

   Once configured, click **Next**

4. **User Matching (Identifying Users)**
   Default settings can typically be left unchanged unless your environment requires customization.

5. **Optional Features (optional)**
   You may enable the following features based on your needs:

   * **Password Writeback** – allows password resets in Azure AD to be pushed back to on-prem AD
   * **Group Writeback** – syncs Azure AD groups to your local AD
   * **Device Writeback** – enables support for Hybrid Azure AD Join

### Step 4: Finalize the Installation

1. **Confirm and Install**
   Review all chosen settings and click **Install** to proceed.

2. **Initial Synchronization**
   Once installed, the Synchronization Service Manager will open and automatically begin the first sync cycle.

3. **Verify in Azure**
   Log into the Azure portal and navigate to **Azure Active Directory > Users**
   Confirm that users from your on-prem AD are now visible.

### Post-Installation Configuration

#### Check Sync Status

1. Open **Synchronization Service Manager**
2. Check the sync history for any failed attempts or warnings

#### Run Manual Sync

To initiate synchronization manually, use **PowerShell as Administrator** and run:

* **Delta Sync (only changes)**

  ```powershell
  Start-ADSyncSyncCycle -PolicyType Delta
  ```

* **Full Sync**

  ```powershell
  Start-ADSyncSyncCycle -PolicyType Initial
  ```

#### Enable Hybrid Azure AD Join (Optional)

1. Launch the **Azure AD Connect** tool
2. Select **Device Options**
3. Choose **Configure Hybrid Azure AD Join** and follow the wizard steps

### Group Policy Adjustments (if needed)

If active scripting in Internet Explorer is disabled by Group Policy, you can re-enable it:

1. Press `Win + R`, type `gpedit.msc`, and hit Enter
2. Navigate to:
   `User Configuration > Administrative Templates > Windows Components > Internet Explorer > Internet Control Panel > Security Page > Internet Zone`
3. Set **Allow active scripting** to **Enabled**

### Best Practices

* **Backup Regularly**
  Periodically export your Azure AD Connect settings for disaster recovery purposes

* **Monitor Health**
  Use Synchronization Service Manager to review logs and validate successful syncs

* **Use Staging Mode for Redundancy**
  Deploy a secondary Azure AD Connect server in **staging mode** to improve resilience and failover readiness
