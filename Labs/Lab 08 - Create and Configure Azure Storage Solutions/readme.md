# Lab 08 - Create and Configure Azure Storage Solutions

## Scenario

Your organization is currently storing data in on-premises data stores. Most of these files are not accessed frequently. You would like to minimize the cost of storage by placing infrequently accessed files in lower-priced storage tiers. You also plan to explore different protection mechanisms that Azure Storage offers, including network access, authentication, authorization, and replication. Finally, you want to determine to what extent Azure Files is suitable for hosting your on-premises file shares.

## Aim

The aim of this lab is to manage Azure Storage by creating and configuring storage accounts for Azure Blobs and Azure Files.


## What You Will Achieve by the End of the Lab

- Create and secure Azure Storage Accounts.

- Configure Azure Blob containers and set access controls.

- Use Storage Browser to manage and secure Azure File shares.

- Implement data protection features like redundancy, network restrictions, and access policies.


## Architecture Diagram

![Lab 08 - Architecture diagram](./Lab%2008%20-%20Architecture%20Diagram.png)

---

## Instructions

## Step 1: Create and configure a storage account

In this task, you will create and configure a storage account. The storage account will use geo-redundant storage and will not have public access.

1. Sign in to the Azure portal - https://portal.azure.com.

2. Search for and select **Storage accounts**, and then click **+ Create**.

3. On the **Basics** tab of the *Create a storage account* blade, specify the following settings (leave others with their default values):

   | Setting | Value |
   |--------|-------|
   | Subscription | the name of your Azure subscription |
   | Resource group | `az104-rg7` (create new) |
   | Storage account name | any globally unique name between 3 and 24 in length consisting of letters and digits |
   | Region | (US) East US |
   | Performance | Standard (notice the Premium option) |
   | Redundancy | Geo-redundant storage |
   | Make read access to data in the event of regional availability | Check the box |

> **Did you know?** You should use the Standard performance tier for most applications. Use the Premium performance tier for enterprise or high-performance applications.

4. On the **Advanced** tab, use the informational icons to learn more about the choices. Take the defaults.

5. On the **Networking** tab, in the *Network access* section, select **Disable public access** and use private access.

6. Review the **Data protection** tab. Accept the defaults.

7. Review the **Encryption** tab. Accept the defaults.

8. Select **Review + Create**, wait for validation, and then click **Create**.

9. Once the storage account is deployed, select **Go to resource**.

10. Review the **Overview** blade.

11. In the **Security + networking** blade, select **Networking**.

12. Change **Public network access** to **Enable from selected networks and IP addresses**.

13. In the **Firewall** section, select the checkbox to **Add your client IP address**.

14. Save your changes.

15. In the **Data management** blade, select **Redundancy** and review info.

16. In **Data management**, select **Lifecycle management** > **Add a rule**.

   - Name the rule: `Movetocool`
   - Click **Next**
   - On *Base blobs* tab: if blobs were last modified more than 30 days ago, **Move to cool storage**
   - Click **Add** when done

---

## Step 2: Create and configure secure blob storage

In this task, create a Blob container. This container will be used to store unstructured data such as files or images, and we’ll apply retention and access policies here.

### Create a blob container and a time-based retention policy

1. In the **Data storage** blade, select **Containers**.

2. Click **+ Add container** and use the following:

   | Setting | Value |
   |--------|-------|
   | Name | `data` |
   | Public access level | Private (default) |

3. Scroll to the ellipsis (…) of your container, select **Access Policy**.

4. In *Immutable blob storage*, click **Add policy**:

   | Setting | Value |
   |--------|-------|
   | Policy type | Time-based retention |
   | Retention period | 180 days |

5. Click **Save**.

### Manage blob uploads

1. Go to the **data** container and click **Upload**.

2. On the *Upload blob* blade, expand the **Advanced** section.

   | Setting | Value |
   |--------|-------|
   | File | Upload a small file |
   | Blob type | Block blob |
   | Block size | 4 MiB |
   | Access tier | Hot |
   | Upload to folder | `securitytest` |
   | Encryption scope | Use existing default container scope |

3. Click **Upload**.

4. Confirm upload success.

5. Select your uploaded file and view options: Download, Delete, Change tier, Acquire lease.

6. Copy the file **URL** (Properties blade) and open it in a new **InPrivate** browser tab.

   - You should receive an XML message: `ResourceNotFound` or `PublicAccessNotPermitted`.

> This is expected since the container has public access disabled.

### Configure limited access to the blob storage

1. Go to your uploaded file, select the ellipsis (…), then **Generate SAS**.

2. Specify:

   | Setting | Value |
   |--------|-------|
   | Signing key | Key 1 |
   | Permissions | Read |
   | Start date | yesterday’s date |
   | Start time | current time |
   | Expiry date | tomorrow’s date |
   | Expiry time | current time |
   | Allowed IP addresses | leave blank |

3. Click **Generate SAS token and URL**.

4. Copy the **Blob SAS URL**.

5. Open it in a new **InPrivate** browser tab.

   - You should be able to view the file contents.

---

## Step 3: Create and configure an Azure File storage

You will create an Azure File Share. This service allows us to store and manage structured file data in a shared format.

### Create the file share and upload a file

1. In the portal, go to your storage account.

2. In **Data storage**, click **File shares**.

3. Click **+ File share** and configure:

   - Name: `share1`
   - Access tier: Transaction optimized (default)
   - Backup: Unchecked

4. Click **Review + create**, then **Create**.

### Explore Storage Browser and upload a file

1. Open **Storage browser**.

2. Select **File shares** and ensure `share1` exists.

3. Click into `share1`, and click **+ Add directory** to create folders if desired.

4. Click **Upload**, select a file, and upload it.

> You can manage Azure File shares here without restrictions.

---

## Restrict network access to the storage account

1. Search for and select **Virtual networks** > **+ Create**.

2. Use resource group `az104-rg7`, and name the vnet `vnet1`.

3. Accept defaults and click **Review + create**, then **Create**.

4. Go to **vnet1**, then **Service endpoints** > **Add**.

   - Service: `Microsoft.Storage`
   - Subnet: Default

5. Click **Add**.

6. Return to your **storage account** > **Networking**.

7. Add existing virtual network: `vnet1`, subnet: Default.

8. In **Firewall**, delete your client IP.

9. Save your changes.

10. Open **Storage browser**, refresh the page.

> You should now receive a message that you're not authorized — expected since you're not connecting from the allowed virtual network. It may take a few minutes to apply.

---

## Cleanup your resources

If you're using your own subscription, delete resources to prevent charges.

- In Azure Portal: Select the **resource group**, click **Delete**, confirm by entering the group name.

- Using PowerShell:
  ```powershell
  Remove-AzResourceGroup -Name resourceGroupName
  ```

- Using Azure CLI:
  ```bash
  az group delete --name resourceGroupName
  ```

---

## Implementation Video


[![Watch the video](./Create%20and%20Configure%20Azure%20Storage%20Solutions%20Thumbmail.png)]()

---

## Learn more with self-paced training

- [Optimize your cost with Azure Blob Storage](https://learn.microsoft.com/en-us/azure/storage/blobs/storage-blob-storage-tiers)
- [Control access to Azure Storage with shared access signatures](https://learn.microsoft.com/en-us/azure/storage/common/storage-sas-overview)

---

## Key takeaways

- An Azure storage account contains all your Azure Storage data objects: blobs, files, queues, and tables.
- Azure storage provides several redundancy models: LRS, ZRS, GRS.
- Blob storage stores large unstructured data, like multimedia files.
- Azure File storage provides shared structured storage with folder support.
- Immutable storage supports WORM data states with time-based or legal-hold policies.
