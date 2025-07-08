# Lab 10 - Deploy and Manage Web Apps with Azure App Service

## Lab Scenario

Your organization is interested in Azure Web Apps for hosting your company websites. The websites are currently hosted in an on-premises data center. The websites are running on Windows servers using the PHP runtime stack. The hardware is nearing end-of-life and will soon need to be replaced. Your organization wants to avoid new hardware costs by using Azure to host the websites.

## Aim

The aim of this lab is to deploy, manage, and scale web applications using Azure App Services.


## What You Will Achieve by the End of the Lab

- Create and configure a web app in Azure.

- Use deployment slots (like staging and production) to support safe and controlled deployments.

- Implement continuous deployment using a GitHub repository.

- Perform slot swaps to move tested code to production with minimal downtime.

- Configure autoscaling to handle increased load and optimize performance and cost.


## Architecture Diagram

![Lab 10 - Architecture diagram](./Lab%2010%20-%20Architecture%20diagram.png)


---

## Task 1: Create and Configure an Azure Web App

1. Sign in to the Azure portal: https://portal.azure.com.
2. Search for and select **App Services**.
3. Select **+ Create**, then choose **Web App**.
4. On the **Basics** tab, enter the following:

   | Setting         | Value                       |
   |-----------------|-----------------------------|
   | Subscription    | your Azure subscription     |
   | Resource group  | `az104-rg9` (create if needed) |
   | Web app name    | any globally unique name    |
   | Publish         | Code                        |
   | Runtime stack   | PHP 8.2                     |
   | Operating system| Linux                       |
   | Region          | East US                     |
   | Pricing plans   | Premium V3 P1V3             |
   | Zone redundancy | Accept defaults             |

5. Click **Review + create**, then **Create**.
6. Wait for deployment to complete, then click **Go to resource**.

> **Note**: If deployment fails, switch to another region and try again.

---

## Task 2: Create and Configure a Deployment Slot

1. On the Web App blade, click the **Default domain** link to view the default web page.
2. Close the new tab and return to the Azure portal.
3. In the **Deployment** section, select **Deployment slots**.
4. Click **Add Slot** and enter:

   | Setting         | Value               |
   |-----------------|---------------------|
   | Name            | `staging`           |
   | Clone settings from | Do not clone settings |

5. Click **Add**.
6. Refresh to view both **Production** and **Staging** slots.
7. Select the new **staging** slot.
8. Observe that the staging slot URL differs from production.

---

## Task 3: Configure Web App Deployment Settings

1. On the **staging** slot blade, select **Deployment Center > Settings**.
2. In the **Source** drop-down, select **External Git**.
3. Enter the following:

   | Field        | Value                                              |
   |--------------|----------------------------------------------------|
   | Repository   | `https://github.com/Azure-Samples/php-docs-hello-world` |
   | Branch       | `master`                                           |

4. Click **Save**.
5. In the **staging** slot, go to **Overview**, then click the **Default domain** link.
6. Verify the Hello World page is displayed.

> **Note**: Deployment may take a minute. Refresh as needed.

---

## Task 4: Swap Deployment Slots

1. Navigate back to the **Deployment slots** blade.
2. Select **Swap**.
3. Review the defaults and click **Start Swap**.
4. Wait for completion.
5. Return to **Home > App Services**, select your Web App.
6. On the **Overview** tab, click the **Default domain** link.
7. Verify that the **Production** slot now displays the Hello World page.

> Copy the default domain URL—you’ll need it for the next task.

---

## Task 5: Configure and Test Autoscaling of the Azure Web App

1. In the **production slot**, go to **Settings > Scale out (App Service plan)**.
2. Set **Scaling** to **Automatic**.
3. Set **Maximum burst** to `2`.
4. Click **Save**.

### Create Load Test

1. In the left pane, select **Diagnose and solve problems**.
2. Under **Load Test your App**, click **Create Load Test**.
3. Click **+ Create** and give the load test a unique name.
4. Click **Review + create**, then **Create**.
5. When ready, click **Go to resource**.

### Configure Load Test

1. Under **Overview**, click **Create** under HTTP requests.
2. On the **Test Plan** tab, click **Add Request**.
3. Enter your app's default domain URL (must start with `https://`).
4. Click **Add**.
5. Click **Review + create**, then **Create**.

> It may take a couple of minutes to set up the test.

6. Navigate to the test from the Load Test home.
7. Monitor live metrics: Virtual users, Response time, Requests/sec.
8. Click **Stop** to end the test run.

---

## Cleanup Your Resources

To avoid charges, delete the resource group:

### Azure Portal

- Go to the **resource group**, click **Delete resource group**.
- Enter the resource group name, click **Delete**.

### Azure PowerShell

```powershell
Remove-AzResourceGroup -Name resourceGroupName
```

### Azure CLI

```bash
az group delete --name resourceGroupName
```

---

## Implementation Video


[![Watch the video](./Lab%2010%20Thumbnail.png)](https://youtu.be/YtH_W93lvJA)

---

## Learn More 

- [Stage a web app deployment for testing and rollback by using App Service deployment slots](https://learn.microsoft.com/training/modules/stage-deploy-app-service-deployment-slots/)
- [Scale an App Service web app to efficiently meet demand with App Service scale up and scale out](https://learn.microsoft.com/training/modules/app-service-scale-up-scale-out/)

---

## Key Takeaways

- Azure App Services lets you quickly build, deploy, and scale web apps.
- Supports multiple development environments including ASP.NET, Java, PHP, and Python.
- Deployment slots allow separate environments for deploying and testing.
- Apps can scale manually or automatically to handle additional demand.
- Diagnostics and testing tools are built into the Azure portal.
