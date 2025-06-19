# Lab 04 - Manage Governance via Azure Policy

## Scenario

Your organization’s cloud footprint has grown considerably in the last year. During a recent audit, you discovered a substantial number of resources that do not have a defined owner, project, or cost center. To improve management of Azure resources, you decide to implement:

- Apply resource tags to attach important metadata to Azure resources
- Enforce the use of resource tags for new resources using Azure Policy
- Update existing resources with resource tags
- Use resource locks to protect configured resources


## Aim

The primary aim of this lab is to implement governance and operational control over Azure resources using Azure Policy, resource tagging, and resource locks. You will learn how to:

- Create and assign tags via the Azure portal
- Enforce tagging via an Azure Policy
- Apply tagging via an Azure Policy
- Configure and test resource locks


## Architecture Diagram

![Lab 04 - Architecture diagram](./Lab%2004%20-%20Architecture%20diagram.png)

---

## Task 1: Assign Tags via the Azure Portal

In this task, you will create and assign a tag to an Azure resource group via the Azure portal. Tags are a critical component of a governance strategy as outlined by the Microsoft Well-Architected Framework and Cloud Adoption Framework.

1. Sign in to the [Azure portal](https://portal.azure.com).
2. Search for and select **Resource groups**.
3. Click **+ Create** and configure the following:

   | Setting             | Value           |
   |---------------------|-----------------|
   | Subscription name   | *your subscription* |
   | Resource group name | `az104-rg2`     |
   | Location            | `East US`       |

4. Select Next and move to the Tags tab. Provide information for a new tag:

   | Name        | Value |
   |-------------|-------|
   | Cost Center | 000   |

5. Click **Review + Create**, then **Create**.

---

## Task 2: Enforce Tagging via an Azure Policy

In this task, you will assign the built-in `Require a tag and its value on resources` policy to the resource group.

1. In the Azure portal, search for and select **Policy**.
2. Under **Authoring**, select **Definitions**.
3. Search for **Require a tag and its value on resources**.
4. Select the policy, then click **Assign policy**.
5. Set the scope:

   | Setting         | Value            |
   |------------------|------------------|
   | Subscription     | *your subscription* |
   | Resource Group   | `az104-rg2`      |

   > **Note:** You can assign policies on the management group, subscription, or resource group level. You also have the option of specifying exclusions, such as individual subscriptions, resource groups, or resources. In this scenario, we want the tag on all the resources in the resource group.

6. Configure basic properties of the assignment and leave others as default:

   | Setting             | Value                                               |
   |---------------------|-----------------------------------------------------|
   | Assignment name     | Require Cost Center tag and its value on resources |
   | Description         | Require Cost Center tag and its value on all resources in the resource group |
   | Policy enforcement  | Enabled                                             |

7. Click Next and set Parameters to the following values:

   | Tag Name    | Value |
   |-------------|-------|
   | Cost Center | 000   |

8. Leave remediation settings at default, click **Review + Create**, then **Create**.

9. After 5–10 minutes, try to create a **Storage Account** in `az104-rg2` *without* the required tag. You should see a **Validation failed** message.

**To create the storage account:**

1. In the portal, search for and select **Storage Accounts**, and select **+ Create**.

2. On the **Basics** tab of the **Create storage account** blade, complete the configuration.

| Setting            | Value                                                                 |
|--------------------|-----------------------------------------------------------------------|
| Resource group     | az104-rg2                                                             |
| Storage account name | Any globally unique combination of between 3 and 24 lowercase letters and digits, starting with a letter |

3. Select **Review** and then click **Create**.

4. You should receive a **Validation failed** message.

5. View the message to identify the reason for the failure. Verify that the error message states that the resource deployment was disallowed by the policy.

> **Note:** Click the **Raw Error** tab for more detailed information about the error. It should reference the policy definition `Require a tag and its value on resources`. The deployment failed because the storage account you attempted to create did not have a tag named `Cost Center` with its value set to `000`.


---

## Task 3: Apply Tagging via an Azure Policy

In this task, you will apply a policy that will make any child resources of a resource group inherit the Cost Center tag if they are missing it.

1. In **Policy > Assignments**, delete the previously assigned policy.
2. Click **Assign policy** and specify the **Scope:**

   | Setting         | Value            |
   |------------------|------------------|
   | Subscription     | *your subscription* |
   | Resource Group   | `az104-rg2`      |

3. Search for and Select **Inherit a tag from the resource group if missing**.

4. Select Add and then configure the remaining Basics properties of the assignment.

   | Setting             | Value                                                              |
   |---------------------|--------------------------------------------------------------------|
   | Assignment name     | Inherit the Cost Center tag and its value 000 from the resource group if missing |
   | Description         | Inherit the Cost Center tag and its value 000 from the resource group if missing |
   | Policy enforcement  | Enabled                                                            |

5. Click Next and set **Parameters** to the following values:

   | Settings   | Value |
   |-------------|-------|
   | Tag Name |   Cost Center    |

6. Click Next and, on the **Remediation tab**, configure the following settings (leave others with their defaults)

   | Setting                | Value                                                   |
   |------------------------|---------------------------------------------------------|
   | Create a remediation task | Enabled                                             |
   | Policy to remediate     | Inherit a tag from the resource group if missing      |

7. Click **Review + Create**, then **Create**.

8. Wait 5–10 minutes. Create a new **Storage Account** in the resource group `az104-rg2` without adding the tags.


**To create the storage account:**

1. Search for and select **Storage Account** and click **+ Create**.

2. On the **Basics** tab of the *Create storage account* blade, verify that you are using the **Resource Group** that the policy was applied to and specify the following settings (leave others with their defaults) and click **Review**:

| Setting              | Value                                                                 |
|----------------------|-----------------------------------------------------------------------|
| Storage account name | Any globally unique combination of between 3 and 24 lowercase letters and digits, starting with a letter |

3. Verify that this time the validation passed and click **Create**.

4. Once the new storage account is provisioned, click **Go to resource**.

5. Once provisioned, navigate to the **Tags** blade of the storage account. You should see that the tag **Cost Center** with the value **000** has been automatically assigned to the resource.

   | Name        | Value |
   |-------------|-------|
   | Cost Center | 000   |

> **Tip:** You can view all tagged resources by searching for **Tags** in the portal.

---

## Task 4: Configure and Test Resource Locks

In this task, you configure a resource lock to prevent deletion of a resource group.

1. Go to your resource group: **az104-rg2**
2. In the **Settings** blade, select **Locks**
3. Select **Add** and provide the following:

   | Setting   | Value  |
   |-----------|--------|
   | Lock name | rg-lock |
   | Lock type | Delete  |

4. Attempt to delete the resource group:
   - Click **Delete resource group**
   - Enter the resource group name `az104-rg2`
   - Click **Delete**

5. You should receive a **deletion denied** message due to the lock.

> **Note:** Remove the lock if you need to delete the group.

---

## Cleanup Resources

If you are using your own subscription, delete resources to avoid charges:

- Using Azure Portal:
  - Go to the resource group and select **Delete resource group**

- Using Azure PowerShell:
  ```powershell
  Remove-AzResourceGroup -Name resourceGroupName

---

## Implementation Video

[![Watch the video](./Manage%20Governance%20via%20Azure%20Policy%20Img.png)]()

---

## Key Takeaways

- Azure tags are key-value pairs for logically organizing resources.

- Azure Policy establishes conventions and enforces resource compliance.

- Policy remediation tasks can bring non-compliant resources into compliance.

- Resource locks protect resources from deletion/modification and override user permissions.

- Governance Layers:
  - Azure Policy: Pre-deployment enforcement
  - RBAC & Locks: Post-deployment protection

---