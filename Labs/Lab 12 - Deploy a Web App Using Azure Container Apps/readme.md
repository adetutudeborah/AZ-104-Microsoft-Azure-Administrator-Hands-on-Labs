# Lab 12 - Deploy a Web App Using Azure Container Apps

## Lab Scenario

Your organization has a web application that runs on a virtual machine in your on-premises data center. The organization wants to move all applications to the cloud but doesn’t want to have a large number of servers to manage. You decide to evaluate **Azure Container Apps**.

## Aim

The aim of this lab is to deploy a containerized web application using Azure Container Apps.


## What You Will Achieve by the End of the Lab

- Create and configure an Azure Container App and environment.
- Test and verify deployment of the Azure Container App.


## Architecture Diagram

![Lab 11 - Architecture diagram](./Lab%2012%20Architecture%20Diagram.png)


---

## Task 1: Create and Configure an Azure Container App and Environment

Azure Container Apps take the concept of a managed Kubernetes cluster a step further and manages the cluster environment as well as provides other managed services on top of the cluster. Unlike an Azure Kubernetes cluster, where you must still manage the cluster, an Azure Container Apps instance removes some of the complexity to setting up a Kubernetes cluster.

### Steps:

1. From the Azure portal, search for and select **Container Apps**.
2. Select **+ Create**, then from the drop-down menu, choose **Container App**.
3. Use the following information to fill out the details on the **Basics** tab:

   | Setting               | Action                                             |
   |-----------------------|----------------------------------------------------|
   | Subscription          | Select your Azure subscription                     |
   | Resource group        | `az104-rg9`                                        |
   | Container app name    | `my-app`                                           |
   | Region                | East US                                            |
   | Container Apps Environment | Select **Create new** > Set Environment name to `my-environment` > **Create** |

4. Click **Next: Container** tab and ensure that **Use quickstart image** is checked. You may need to scroll up to view this setting.
5. Ensure **Quickstart image** is set to **Simple hello world container**.
6. Select **Review and create**, then click **Create**.

> **Note**: Wait for the container app to deploy. This will take a couple of minutes.

---

## Task 2: Test and Verify Deployment of the Azure Container App

By default, the Azure container app that you create will accept traffic on **port 80** using the sample **Hello World** application.

1. Azure Container Apps will provide a **DNS name** for the application. Copy and navigate to this URL to ensure that the application is up and running.
2. Select **Go to resource** to view your new container app.
3. Select the link next to **Application URL** to view your application.
4. Verify you receive the `Your Azure Container Apps app is live` message.

---

## Cleanup Your Resources

If you are working with your own subscription, take a minute to delete the lab resources. This will ensure resources are freed up and cost is minimized.

### Azure Portal:

1. In the Azure portal, select the **resource group**.
2. Select **Delete the resource group**.
3. Enter the resource group name.
4. Click **Delete**.

### Azure PowerShell:

```powershell
Remove-AzResourceGroup -Name resourceGroupName
```

### Azure CLI:

```bash
az group delete --name resourceGroupName
```

---

## Implementation Video


[![Watch the video](./Deploy%20a%20Web%20App%20Using%20Azure%20Container%20Apps%20-%20Thumbnail.png)]()

---

## Learn More

- [Configure a container app in Azure Container Apps](https://learn.microsoft.com/training/modules/configure-container-app-azure-container-apps/)

---

## Key Takeaways

- **Azure Container Apps (ACA)** is a serverless platform that allows you to maintain less infrastructure and save costs while running containerized applications.
- Container Apps provides server configuration, container orchestration, and deployment details.
- Workloads on ACA are usually long-running processes like a Web App.
