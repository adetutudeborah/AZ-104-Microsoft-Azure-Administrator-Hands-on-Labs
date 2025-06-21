# Lab 05 - Deploy Azure Resources with ARM Templates and Bicep

## Scenario

Your team wants to look at ways to automate and simplify resource deployments. Your organization is looking for ways to reduce administrative overhead, reduce human error and increase consistency.

## Aim

The aim of this lab is to deploy managed disks in Microsoft Azure using different infrastructure-as-code (IaC) approaches. You'll learn to use:

- Create an Azure Resource Manager template.
- Configure the Cloud Shell and deploy a template with Azure PowerShell.
- Deploy a template with the CLI.
- Deploy a resource by using Azure Bicep.


## Architecture Diagram

![Lab 05 - Architecture diagram](./Lab%2005%20-%20Architecture%20Diagram.png)

---

## Task 1: Create an Azure Resource Manager Template

In this task, we will create a managed disk in the Azure portal. Managed disks are storage designed to be used with virtual machines. Once the disk is deployed you will export a template that you can use in other deployments.

1. Sign in to the Azure portal - https://portal.azure.com.
2. Search for and select **Disks**.
3. On the **Disks** page, select **Create**.
4. On the **Create a managed disk** page, configure the disk and then select **Ok**.

| Setting              | Value                             |
|----------------------|-----------------------------------|
| Subscription         | your subscription                 |
| Resource Group       | az104-rg3 (If necessary, select Create new.) |
| Disk name            | az104-disk1                       |
| Region               | East US                           |
| Availability zone    | No infrastructure redundancy required |
| Source type          | None                              |
| Performance          | Standard HDD (change size)        |
| Size                 | 32 GiB                            |

> **Note:** We are creating a simple managed disk so you can practice with templates. Azure managed disks are block-level storage volumes that are managed by Azure.

5. Click **Review + Create** then select **Create**.
6. Monitor the notifications (upper right) and after the deployment select **Go to resource**.
7. In the **Automation** blade, select **Export template**.
8. Take a minute to review the Template and Parameters files.
9. Click **Download** and save the templates to the local drive. This creates a compressed zipped file.
10. Use File Explorer to extract the content of the downloaded file into the **Downloads** folder on your computer. Notice there are two JSON files (template and parameters).

> **Did you know?** You can export an entire resource group or just specific resources within that resource group.

---

## Task 2: Edit an Azure Resource Manager Template and Redeploy

In this task, you use the downloaded template to deploy a new managed disk. This task outlines how to quickly and easily repeat deployments.

1. In the Azure portal, search for and select **Deploy a custom template**.
2. On the **Custom deployment** blade, select **Build your own template in the editor**.
3. On the **Edit template** blade, click **Load file** and upload the `template.json` file you downloaded.
4. Make these changes:
   - Change `disks_az104_disk1_name` to `disk_name` (two places).
   - Change `az104-disk1` to `az104-disk2` (one place).
5. Save your changes.
6. Select **Edit parameters**, click **Load file**, and upload the `parameters.json`.
7. Make this change:
   - Change `disks_az104_disk1_name` to `disk_name` (one place).
8. Save your changes.
9. Complete the custom deployment settings:

| Setting        | Value         |
|----------------|---------------|
| Subscription   | your subscription |
| Resource Group | az104-rg3     |
| Region         | East US       |
| Disk_name      | az104-disk2   |

10. Select **Review + Create**, then **Create**.
11. Select **Go to resource**. Verify `az104-disk2` was created.
12. On the **Overview** blade, select the resource group `az104-rg3`. You should now have two disks.
13. In the **Settings** section, click **Deployments**.

> **Note:** All deployments details are documented in the resource group. It is a good practice to review the first few template-based deployments to ensure success prior to using the templates for large-scale operations.

14. Select a deployment and review the content of the **Input** and **Template** blades.

---

## Task 3: Configure the Cloud Shell and Deploy a Template with PowerShell

In this task, you work with the Azure Cloud Shell and Azure PowerShell.

1. Select the Cloud Shell icon in the top right of the Azure Portal or go to: https://shell.azure.com
2. When prompted, select **PowerShell**.
3. On the **Getting started** screen:
   - Select **Mount storage account**, select your subscription, then **Apply**.
   - Select **I want to create a storage account**, then **Next**.

Complete the Create Storage Account information:

| Setting            | Value                                      |
|--------------------|--------------------------------------------|
| Resource Group     | az104-rg3                                  |
| Region             | your region                                |
| Storage account    | must be globally unique, 3-24 characters, lowercase/numbers only |
| File share         | fs-cloudshell                              |

4. Select **Create**. Wait for provisioning.
5. Select **Settings** (top bar) > **Go to classic version**.
6. Click **Upload/Download files** icon > **Upload** both template and parameters files.
7. Open **Editor** icon and navigate to `template.json`. Change disk name to `az104-disk3`. Press **Ctrl + S** to save.
8. Deploy with PowerShell:

    ```powershell
    New-AzResourceGroupDeployment -ResourceGroupName az104-rg3 -TemplateFile template.json -TemplateParameterFile parameters.json
    ```

9. Ensure the command completes and the ProvisioningState is Succeeded.

10. Confirm the disk was created:

    ```powershell
    Get-AzDisk
    ```

## Task 4: Deploy a Template with the CLI

1. Continue in the Cloud Shell, select **Bash**. Confirm your choice.
2. Verify your files are available in the Cloud Shell storage. If you completed the previous task, your template files should be available.

    ```bash
    ls
    ```

3.Select the Editor (curly brackets icon) and navigate to the template JSON file.

4. Make a change. For example, change the disk name to az104-disk4. Use Ctrl + S to save your changes.

> **Note**: You can target your template deployment to a resource group, subscription, management group, or tenant. Depending on the scope of the deployment, you use different commands.

**To deploy to a resource group, use:**

    az deployment group create --resource-group az104-rg3 --template-file template.json --parameters parameters.json

5. Ensure the command completes and the ProvisioningState is Succeeded.

6. Confirm the disk was created:

    ```
    az disk list --output table
    ```

---

## Task 5: Deploy a resource by using Azure Bicep

In this task, you will use a Bicep file to deploy a managed disk. Bicep is a declarative automation tool that is built on ARM templates.

1. Locate the **\Labs\Lab 05\azuredeploydisk.bicep** file.

2. Continue working in the Cloud Shell in a Bash session.

3. Select Manage files and then Upload the Bicep file to the Cloud Shell.

4. Click Editor and when prompted Confirm the switch to the Classic Cloud Shell.

5. Select the azuredeploydisk.bicep file

6. Take a minute to read through the Bicep template file. Notice how the disk resource is defined.

7. Make the following changes:

    - Change the managedDiskName value, line 2, to az104-disk5.
    - Change the sku name value, line 26, to StandardSSD_LRS.
    - Change the diskSizeinGiB value; line 7, to 32.

8. Use Ctrl + S to save your changes.

9. Now, deploy the template.

    ```
    az deployment group create --resource-group az104-rg3 --template-file azuredeploydisk.bicep
    ```

10. Confirm the disk was created.

    ```
    az disk list --output table
    ```

---

> Note: You have successfully deployed five managed disks, each in a different way.

## Cleanup Your Resources

- In the **Azure portal**:
  - Select the resource group.
  - Select **Delete the resource group**.
  - Enter the resource group name.
  - Click **Delete**.

- Using **Azure PowerShell**:

    ```powershell
    Remove-AzResourceGroup -Name resourceGroupName
    ```

- Using **Azure CLI**:

    ```
    az group delete --name resourceGroupName
    ```


## Implementation Video

[![Watch the video](./Deploy%20Azure%20Resources%20with%20ARM%20Templates%20and%20Bicep%20Image.png)]()

---

## Learn More 

- **[Deploy Azure infrastructure by using JSON ARM templates](https://learn.microsoft.com/training/modules/create-azure-resource-manager-template-vs-code/)**
- **[Build your first Bicep template](https://learn.microsoft.com/training/modules/build-first-bicep-template/)**

---

## Key Takeaways

- ARM templates help deploy and manage Azure resources as a group.
- Templates are written in JSON and use a declarative approach.
- You can use a separate file for parameters instead of hardcoding them.
- Azure Resource Manager templates can be deployed in a variety of ways including the Azure portal, Azure PowerShell, and CLI.
- Bicep is an alternative to Azure Resource Manager templates. Bicep uses a declarative syntax to deploy Azure resources.
- Bicep supports reuse, type safety, and a better authoring experience.

---
