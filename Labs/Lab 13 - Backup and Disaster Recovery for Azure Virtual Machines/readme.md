# Lab 13 - Backup and Disaster Recovery for Azure Virtual Machines

## Lab Scenario

Your organization is evaluating how to backup and restore Azure virtual machines from accidental or malicious data loss. Additionally, the organization wants to explore using Azure Site Recovery for disaster recovery scenarios.

## Aim

The aim of this lab is to learn how to protect Azure virtual machines from data loss and ensure business continuity using Azure Backup and Azure Site Recovery.


## What You Will Achieve by the End of the Lab

- Deploy an Azure virtual machine using an ARM template.
- Create and configure a Recovery Services vault to store backup data and enable recovery options.
- Configure Azure virtual machine-level backup by applying backup policies and defining retention periods.
- Monitor Azure Backup by enabling diagnostics and view job status using logs and metrics.
- Enable virtual machine replication by using Azure Site Recovery to replicate a VM to another region for disaster recovery.


## Architecture Diagram

![Lab 13 - Architecture diagram](./Lab%2013%20Architecture%20Diagram.png)

---

## Task 1: Use a template to provision an infrastructure

In this task, you will use a template to deploy a virtual machine. The virtual machine will be used to test different backup scenarios.

1. Download the `\Labs\Lab 13 - Backup and Disaster Recovery for Azure Virtual Machines\` lab files.
2. Sign in to the Azure portal: https://portal.azure.com.
3. Search for and select **Deploy a custom template**.
4. On the Custom Deployment page, select **Build your own template in the editor**.
5. On the Edit template page, select **Load file**.
6. Locate and select the `Labs\Lab 13 - Backup and Disaster Recovery for Azure Virtual Machines\az104-10-vms-edge-template.json` file and select **Open**.
7. **Review the template.** This template deploys a virtual network and virtual machine.
8. Save your changes.
9. Select **Edit parameters**, then **Load file**.
10. Load and select the `Labs\Lab 13 - Backup and Disaster Recovery for Azure Virtual Machines\az104-13-vms-edge-parameters.json` file.
11. Save your changes.

### Use the following settings:

| Setting       | Value                       |
|---------------|-----------------------------|
| Subscription  | Your Azure subscription     |
| Resource group| az104-rg-region1 (create if needed) |
| Region        | East US                     |
| Username      | localadmin                  |
| Password      | Provide a complex password  |

12. Select **Review + Create**, then **Create**.

> **Note**: Wait for deployment to complete, then select **Go to resource**.

---

## Task 2: Create and configure a Recovery Services vault

In this task, you will create a Recovery Services vault. A Recovery Services vault provides storage for the virtual machine data.

1. In the Azure portal, search for and select **Recovery Services vaults**, then click **+ Create**.

### Specify the following:

| Setting       | Value                       |
|---------------|-----------------------------|
| Subscription  | Your Azure subscription     |
| Resource group| az104-rg-region1            |
| Vault Name    | az104-rsv-region1           |
| Region        | East US                     |

2. Click **Review + Create**, then **Create**.

> Wait for deployment to complete.

3. Click **Go to Resource**.
4. In **Settings**, click **Properties**.
5. Under **Backup Configuration**, click **Update**.
6. Leave the **Storage replication type** set to **Geo-redundant**.
7. Under **Security Settings**, click **Update** next to **Soft Delete and security settings**.
8. Note: **Soft Delete** is enabled with a 14-day retention.

---

## Task 3: Configure Azure virtual machine-level backup

In this task, you will implement Azure virtual-machine level backup. As part of a VM backup, you will need to define the backup and retention policy that applies to the backup. Different VMs can have different backup and retention policies assigned to them.

1. On the Recovery Services vault blade, click **Overview**, then **+ Backup**.

### Configure the backup goal:

| Setting                   | Value               |
|---------------------------|---------------------|
| Where is your workload?   | Azure               |
| What do you want to backup?| Virtual machine    |

2. Select **Backup**.
3. Choose **Standard** under **Policy Subtypes**.
4. Click **Create a new policy**.

### New Backup Policy:

| Setting                                | Value                   |
|----------------------------------------|--------------------------|
| Policy name                            | az104-backup             |
| Frequency                              | Daily                    |
| Time                                   | 12:00 AM                 |
| Timezone                               | Your local time zone     |
| Retain instant recovery snapshot(s) for| 2 Days                   |

5. Click **OK**.
6. In the **Virtual Machines** section, click **Add**.
7. Select **az104-10-vm0**, click **OK**, then click **Enable backup**.

> Wait approximately 2 minutes for the backup to be enabled.

8. Go to **Protected items > Backup items > Azure virtual machine**.
9. Select **View details** for **az104-10-vm0**.
10. Click **Backup now**, accept defaults, and click **OK**.

> Proceed to next task without waiting for backup completion.

---

## Task 4: Monitor Azure Backup

In this task, you will deploy an Azure storage account. Then you will configure the vault to send the logs and metrics to the storage account. This repository can then be used with Log Analytics or other third-party monitoring solutions.

1. In Azure portal, search for and select **Storage accounts**.
2. Click **Create**, use the following settings:

| Setting            | Value                     |
|--------------------|---------------------------|
| Subscription       | Your subscription         |
| Resource group     | az104-rg-region1          |
| Storage account    | Provide globally unique name |
| Region             | East US                   |

3. Click **Review + Create**, then **Create**.

4. Go to your Recovery Services vault.
5. Under **Monitoring**, click **Diagnostic Settings** > **Add diagnostic setting**.

### Diagnostic Setting:

- Name: `Logs and Metrics to storage`
- Categories (select):
  - Azure Backup Reporting Data
  - Addon Azure Backup Job Data
  - Addon Azure Backup Alert Data
  - Azure Site Recovery Jobs
  - Azure Site Recovery Events
- Destination: Archive to a storage account
- Select the storage account created earlier

6. Click **Save**.
7. In the Recovery Services vault, go to **Monitoring > Backup jobs**.
8. Locate the **az104-10-vm0** backup job and **view details**.

---

## Task 5: Enable virtual machine replication

1. In the portal, search for **Recovery Services vaults**, then click **+ Create**.

### Specify the following:

| Setting       | Value                        |
|---------------|------------------------------|
| Subscription  | Your subscription            |
| Resource group| az104-rg-region2 (create new)|
| Vault Name    | az104-rsv-region2            |
| Region        | West US                      |

2. Click **Review + Create**, then **Create**.

> Wait for deployment to complete.

3. Search and select the **az104-10-vm0** virtual machine.
4. Go to **Backup + Disaster recovery > Disaster recovery**.
5. On the **Basics** tab, verify **Target region**.
6. Click **Next: Advanced settings**, verify resource selections.
7. Scroll down and create the **Automation Account**.
8. Click **Review + Start replication**, then **Enable replication**.

> Replication takes 10–15 minutes. Monitor notifications.

9. Once replication is complete, go to **az104-rsv-region2** vault.
10. Under **Protected items**, select **Replicated items**.
11. Verify VM replication health shows as **Healthy** and **Protected**.

---

## Cleanup Your Resources

To avoid incurring charges, delete the resource group:

- **Azure Portal**: Go to Resource Group > Delete
- **Azure PowerShell**:  
  ```powershell
  Remove-AzResourceGroup -Name resourceGroupName
  ```
- **Azure CLI**:  
  ```bash
  az group delete --name resourceGroupName
  ```

---

## Implementation Video


[![Watch the video](./Backup%20and%20Disaster%20Recovery%20for%20Azure%20Virtual%20Machines%20-%20Thumbnail.png)](https://youtu.be/ObKwteRSsFM)

---


## Learn More 

- [Protect your virtual machines by using Azure Backup](https://learn.microsoft.com/training/modules/protect-virtual-machines-with-azure-backup/)

- [Protect your Azure infrastructure with Azure Site Recovery](https://learn.microsoft.com/en-us/training/modules/protect-infrastructure-with-site-recovery/)

---

## Key Takeaways

- Azure Backup provides simple, secure, cost-effective backup and recovery.
- It protects on-premises and cloud resources (VMs, file shares, etc).
- Backup policies configure backup frequency and retention.
- Azure Site Recovery provides disaster recovery protection.
- Recovery Services vaults store backup data and reduce management overhead.
