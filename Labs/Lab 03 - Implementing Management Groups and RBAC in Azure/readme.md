# Lab 03 - Implementing Management Groups and RBAC in Azure

## Scenario

To simplify management of Azure resources in your organization, you have been tasked with implementing the following functionality:

- Creating a management group that includes all your Azure subscriptions.
- Granting permissions to submit support requests for all subscriptions in the management group. The permissions should be limited only to:
  - Create and manage virtual machines
  - Create support request tickets (do not include adding Azure providers)


## Aim

The aim of this lab is to implement Azure Role-Based Access Control (RBAC) and organize Azure subscriptions using Management Groups. You will learn how to:

- Organize subscriptions under a management group
- Assign built-in roles to users or groups
- Create and assign a custom role with limited permissions (least privilege)
- Monitor role assignments using Azure Activity Logs


## Architecture Diagram

![Lab 03 - Architecture diagram](./Lab%2003%20-%20Architecture%20diagram.png)

---

## Task 1: Implement Management Groups

1. Sign in to the Azure portal: [https://portal.azure.com](https://portal.azure.com)
2. Search for and select **Microsoft Entra ID**.
3. In the **Manage** blade, select **Properties**.
4. Review the **Access management for Azure resources** area. Ensure you can manage access to all Azure subscriptions and management groups in the tenant.
5. Search for and select **Management groups**.
6. On the **Management groups** blade, click **+ Create**.

### Create a management group with the following settings:

| Setting                 | Value       |
|-------------------------|-------------|
| Management group ID     | az104-mg1 *(must be unique in the directory)* |
| Management group name   | az104-mg1   |

7. Submit and refresh the page to verify the management group was created.

> **Note**: The root management group is built-in and includes all management groups and subscriptions. It allows global policies and role assignments at the directory level.

---

## Task 2: Review and Assign a Built-in Azure Role

1. Select the **az104-mg1** management group.
2. Go to **Access control (IAM)** → **Roles** tab.
3. Browse through built-in roles. View a role to see Permissions, JSON, and Assignments.
4. Click **+ Add** → **Add role assignment**.
5. On the **Add role assignment** blade:
   - Search and select **Virtual Machine Contributor**
   - Click **Next**
6. On the **Members** tab:
   - Select **Members**
   - Search and select the **helpdesk group**
   - Click **Select**
7. Click **Review + assign** twice to complete the assignment.
8. Confirm the assignment on the **Role assignments** tab.

> **Tip**: Always assign roles to groups, not individuals.

---

## Task 3: Create a Custom RBAC Role

1. On the **az104-mg1** management group, go to **Access control (IAM)**.
2. Click **+ Add** → **Add custom role**.
3. On the **Basics** tab, enter:

| Setting            | Value                                       |
|--------------------|---------------------------------------------|
| Custom role name   | Custom Support Request                      |
| Description        | A custom contributor role for support requests. |
| Baseline permission| Clone a role → **Support Request Contributor** |

4. Click **Next** to go to **Permissions** tab.
5. Click **+ Exclude permissions**.
6. Search for `.Support` under **Microsoft.Support**.
7. Check `Other: Registers Support Resource Provider`, and click **Add**.

> **Note**: This will add the excluded permission to the `NotActions` section of the role.

8. On **Assignable scopes**, ensure your management group is selected.
9. Click **Next**, review the JSON, and then click **Review + Create** → **Create**.

---

## Task 4: Monitor Role Assignments with the Activity Log

1. Go to the **az104-mg1** resource in the portal.
2. Click on **Activity log**.
3. Filter the activities to show **Role Assignments** or specific operations like `Create Role Assignment`.
4. Review who created roles and what changes were made.

---

## Cleanup Your Resources

If you are using your own subscription, delete the lab resources to avoid extra costs.

### Using the Azure Portal
- Go to the **management group**
- Click **Delete the management group**
- Confirm the name and click **Delete**

### Using Azure PowerShell or Azure CLI
```powershell
az account management-group delete --name az104-mg1
```

![Remove management group](./mg%20resource%20delete.PNG)

---

## Implementation Video

[![Watch the video](./Implementing%20Management%20Groups%20and%20RBAC%20in%20Azure.png)](https://youtu.be/UHLLBb82HLQ)

---

## Learn More 

- **[Secure your Azure resources with Azure role-based access control (Azure RBAC)](https://learn.microsoft.com/en-us/azure/role-based-access-control/overview)**
- **[Create custom roles for Azure resources with RBAC](https://learn.microsoft.com/en-us/azure/role-based-access-control/custom-roles)**

---

## Key Takeaways

- Management groups logically organize Azure subscriptions.
- The root management group includes all other groups and subscriptions.
- Use built-in roles to control access to resources.
- Create and customize roles for least-privilege access.
- Role definitions are JSON-based and include `Actions`, `NotActions`, and `AssignableScopes`.
- Use **Activity Log** to monitor role-related actions.

---
