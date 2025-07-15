# Lab 14 - Implement Monitoring in Azure using Azure Monitor


## Lab Scenario

Your organization has migrated their infrastructure to Azure. It is important that Administrators are notified of any significant infrastructure changes. You plan to examine the capabilities of Azure Monitor, including Log Analytics.

## Aim

The aim of this lab is to learn how to implement monitoring in Azure using Azure Monitor.

By the end of the lab, you will have:

- Provisioned infrastructure (a virtual machine) using a template.

- Created an alert rule that triggers when a virtual machine is deleted.

- Configured action group notifications (e.g., email alerts to an operations team).

- Tested the alert by deleting the virtual machine to ensure notifications are working.

- Created an alert processing rule to suppress notifications during scheduled maintenance periods.

- Used Azure Monitor log queries (KQL) to analyze metrics and logs from the virtual machine.


## Architecture Diagram

![Lab 14 - Architecture diagram](./Monitoring%20in%20Azure%20-%20Architecture%20Diagram.png)

---

## Task 1: Use a Template to Provision an Infrastructure

In this task, you will deploy a virtual machine that will be used to test monitoring scenarios.

1. Download the `\Labs\Lab 14 - Implement Monitoring in Azure using Azure Monitor\az104-14-vms-edge-template.json` lab files to your computer.
2. Sign in to the Azure portal - https://portal.azure.com.
3. From the Azure portal, search for and select **Deploy a custom template**.
4. On the custom deployment page, select **Build your own template in the editor**.
5. On the edit template page, select **Load file**.
6. Locate and select the `\Labs\Lab 14 - Implement Monitoring in Azure using Azure Monitor\az104-14-vms-edge-template.json` file and select **Open**.
7. Select **Save**.

Use the following information to complete the custom deployment fields (leave all other fields as default):

| Setting        | Value              |
|----------------|--------------------|
| Subscription   | Your Azure subscription |
| Resource group | `az104-rg11` (select *Create new* if needed) |
| Region         | East US            |
| Username       | `localadmin`       |
| Password       | Provide a complex password |

8. Select **Review + Create**, then **Create**.
9. Wait for the deployment to finish, then click **Go to resource group**.
10. Review the resources deployed — there should be one virtual network and one virtual machine.

### Configure Azure Monitor for Virtual Machines

1. In the portal, search for and select **Monitor**.
2. Review the available insights, detection, triage, and diagnosis tools.
3. Select **View** in the *VM Insights* box, then select **Configure Insights**.
4. Select **Enable** next to your virtual machine, then **Enable** on the *Azure Monitor - Insights Onboarding* blade.
5. Use default settings for subscription and data collection rules, then select **Configure**.

> Note: It will take a few minutes for the VM agent to install and configure.

---

## Task 2: Create an Alert

Create an alert for when a virtual machine is deleted.

1. On the **Monitor** page, select **Alerts**.
2. Select **+ Create** > **Alert rule**.
3. In the *Scope* section, select your subscription, then click **Apply**.
4. Select the **Condition** tab, then click **See all signals**.
5. Search for and select **Delete Virtual Machine (Virtual Machines)**. Click **Apply**.
6. In the *Alert logic* section, leave **Event level** and **Status** as default.

Leave this pane open for the next task.

---

## Task 3: Configure Action Group Notifications

Send an email notification to the operations team when the alert is triggered.

1. In the alert rule configuration, select **Use action groups** > **Create action group**.

> ✅ You can add up to five action groups to an alert rule. Action groups are executed concurrently. Multiple alert rules can use the same action group.

### Basics Tab

| Setting           | Value             |
|------------------|-------------------|
| Subscription      | Your subscription |
| Resource group    | `az104-rg11`      |
| Region            | Global (default)  |
| Action group name | `Alert the operations team` |
| Display name      | `AlertOpsTeam`    |

### Notifications Tab

| Setting           | Value                          |
|-------------------|--------------------------------|
| Notification type | Email/SMS message/Push/Voice   |
| Name              | `VM was deleted`               |
| Email             | Enter your email address       |

> You should receive an email saying you were added to an action group.

2. Select **Review + Create**, then **Create**.

### Details Tab (Back in Alert Rule Setup)

| Setting             | Value                            |
|----------------------|----------------------------------|
| Alert rule name      | `VM was deleted`                |
| Alert rule description | A VM in your resource group was deleted |

3. Select **Review + create**, then **Create**.

---

## Task 4: Trigger an Alert and Confirm It Is Working

> ⚠️ Don’t delete the VM until the alert rule is fully deployed.

1. Go to **Virtual machines** in the portal.
2. Select the checkbox for `az104-vm0`.
3. Click **Delete** from the menu bar.
4. Check **Apply force delete**, confirm the deletion, then click **Delete**.
5. In the portal title bar, select **Notifications** and wait for deletion to complete.
6. You should receive a notification email titled:

```
Important notice: Azure Monitor alert "VM was deleted" was activated
```

7. In the Azure portal, go to **Monitor** > **Alerts**.

You should see **three verbose alerts** triggered by deleting the VM.

8. Click the name of one of the alerts (e.g., "VM was deleted") to view more details.

---

## Task 5: Configure an Alert Processing Rule

Create a rule to suppress notifications during a maintenance window.

1. In **Monitor** > **Alerts**, select **Alert processing rules** > **+ Create**.
2. Select your **Subscription**, then **Apply**.
3. On the **Rule settings** tab, choose **Suppress notifications**.
4. Click **Next: Scheduling**.

### Scheduling

| Setting           | Value                              |
|------------------|-------------------------------------|
| Apply the rule   | At a specific time                  |
| Start            | Today’s date at 10:00 PM            |
| End              | Tomorrow’s date at 7:00 AM          |
| Time zone        | Your local timezone                 |

5. Click **Next: Details**.

### Details

| Setting           | Value                                     |
|------------------|--------------------------------------------|
| Resource group    | `az104-rg11`                              |
| Rule name         | `Planned Maintenance`                     |
| Description       | Suppress notifications during maintenance |

6. Select **Review + create**, then **Create**.

---

## Task 6: Use Azure Monitor Log Queries

1. In the Azure portal, search for and select **Monitor**, then click **Logs**.
2. If necessary, close the splash screen and select a scope (your Subscription). Click **Apply**.
3. Under the **Queries** tab, select **Virtual machines** in the left pane.
4. Hover over the **Count heartbeats** query and click **Run**.

> You should receive a heartbeat count for when the VM was running.

5. In the query editor, switch from **Simple mode** to **KQL mode**.
6. Replace the query with the following, then click **Run**:

```kql
InsightsMetrics
| where TimeGenerated > ago(1h)
| where Name == "UtilizationPercentage"
| summarize avg(Val) by bin(TimeGenerated, 5m), Computer
| render timechart
```

> If you have trouble pasting, try pasting into Notepad first, then copy/paste into the editor.

---

## Cleanup Your Resources

To avoid unnecessary charges, delete lab resources.

- In the portal: select the resource group > **Delete resource group** > Confirm the name > **Delete**.
- Azure PowerShell:
```powershell
Remove-AzResourceGroup -Name az104-rg14
```
- Azure CLI:
```bash
az group delete --name az104-rg11
```

---

## Implementation Video

[![Watch the video](./Implement%20monitoring%20in%20Azure%20using%20Azure%20Monitor%20-%20Thumbnail.png)](https://youtu.be/V3QTzNvwC6M)

---

## Learn More 

- [Improve incident response with alerting on Azure](https://learn.microsoft.com/en-us/training/modules/incident-response-with-alerting-on-azure/)
- [Monitor your Azure virtual machines with Azure Monitor](https://learn.microsoft.com/en-us/training/modules/monitor-azure-vm-using-diagnostic-data/)

---

## Key Takeaways

- Alerts help you detect and address issues before they impact users.
- You can alert on any metric or log source in Azure Monitor.
- An alert rule captures signals indicating events or issues.
- When conditions are met, an alert triggers and can notify via email, SMS, push, or voice.
- Action groups define who is notified and what actions are taken.

---
