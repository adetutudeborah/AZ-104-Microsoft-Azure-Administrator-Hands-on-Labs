# Lab 09 - Deploy and Scale Azure Virtual Machines and Scale Sets

## Scenario

Your organization wants to explore deploying and configuring Azure virtual machines. First, you implement an Azure virtual machine with manual scaling. Next, you implement a Virtual Machine Scale Set and explore autoscaling.

## Aim

The aim of the lab is to to deploy, configure, and scale Azure Virtual Machines (VMs) and Virtual Machine Scale Sets (VMSS). Specifically, you'll:

- Create and manage individual virtual machines, including:
     - Deploying VMs across availability zones for high availability.
     - Adjusting compute and storage to meet performance needs (manual/vertical scaling).

- Create and manage Virtual Machine Scale Sets, which enable:
    - Deployment of a group of identical, load-balanced VMs.
    - Automatic scaling based on performance metrics (horizontal scaling/autoscaling).

- Create a virtual machine using Azure PowerShell.

- Create a virtual machine using the CLI.

---

## Task 1: Deploy zone-resilient Azure virtual machines by using the Azure portal

![Task 1 - Architecture diagram](./Azure%20Virtual%20Machines%20Architecture%20Diagram.png)

1. Sign in to the Azure portal - https://portal.azure.com.
2. Search for and select **Virtual machines**, then click **+ Create** and select **Azure virtual machine**.
3. On the **Basics** tab:
   - **Availability zone**: Select Zone 1 and Zone 2.
   - Configure the following:

| Setting                              | Value                            |
|--------------------------------------|----------------------------------|
| Subscription                         | The name of your Azure subscription |
| Resource group                       | `az104-rg8`                      |
| Virtual machine names                | `az104-vm1` and `az104-vm2`      |
| Region                               | East US                          |
| Availability options                 | Availability zone                |
| Availability zone                    | Zone 1, 2                        |
| Security type                        | Standard                         |
| Image                                | Windows Server 2019 Datacenter - x64 Gen2 |
| Azure Spot instance                  | Unchecked                        |
| Size                                 | Standard D2s v3                  |
| Username                             | localadmin                       |
| Password                             | Provide a secure password        |
| Public inbound ports                 | None                             |
| Use existing Windows Server license? | Unchecked                        |

4. Click **Next: Disks >**, then set:
   - **OS disk type**: Premium SSD
   - **Delete with VM**: Checked
   - **Enable Ultra Disk compatibility**: Unchecked

5. Click **Next: Networking >**, take defaults but set:
   - **Delete public IP and NIC when VM is deleted**: Checked
   - **Load balancing options**: None

6. Click **Next: Management >**, set:
   - **Patch orchestration options**: Azure orchestrated

7. Click **Next: Monitoring >**, set:
   - **Boot diagnostics**: Disable

8. Click **Next: Advanced >**, then **Review + Create** > **Create**.

---

## Task 2: Manage compute and storage scaling for virtual machines

1. On `az104-vm1`, go to **Availability + scale** > **Size**.
2. Change size to `D2ds_v4`, click **Resize** (or choose another available size).
3. In **Settings**, go to **Disks**.
4. Under **Data disks**, select **+ Create and attach a new disk**:

| Setting        | Value          |
|----------------|----------------|
| Disk name      | vm1-disk1      |
| Storage type   | Standard HDD   |
| Size (GiB)     | 32             |

5. Click **Apply**.
6. Click **Detach** on the new disk, then **Apply**.
7. Go to **Disks**, select `vm1-disk1`, then **Size + performance** > set:
   - **Storage type**: Standard SSD, click **Save**.
8. Back on `az104-vm1`, go to **Disks** > **Attach existing disk** > Select `vm1-disk1`, click **Apply**.

---

## Task 3: Create and configure Azure Virtual Machine Scale Sets

![Task 3 - Architecture diagram](./Azure%20Virtual%20Machine%20Scale%20Sets%20Architecture%20Diagram.png)

1. Go to **Virtual machine scale sets** > **+ Create**.
2. On **Basics** tab:

| Setting                  | Value                          |
|--------------------------|--------------------------------|
| Subscription             | Your Azure subscription        |
| Resource group           | az104-rg8                      |
| VM scale set name        | vmss1                          |
| Region                   | East US                        |
| Availability zone        | Zones 1, 2, 3                  |
| Orchestration mode       | Uniform                        |
| Security type            | Standard                       |
| Image                    | Windows Server 2019 Datacenter |
| Spot discount            | Unchecked                      |
| Size                     | Standard D2s_v3                |
| Username                 | localadmin                     |
| Password                 | Secure password                |
| Windows Server license   | Unchecked                      |

3. On **Spot**, accept defaults.
4. On **Disks**, accept defaults.
5. On **Networking**:
   - Click **Edit virtual network link**:
     - Name: `vmss-vnet`
     - Address range: `10.82.0.0/20`
     - Subnet name: `subnet0`
     - Subnet range: `10.82.0.0/24`
   - Edit network interface:
     - NIC NSG: Advanced > Create new:
       - Name: `vmss1-nsg`
       - Add inbound rule:
         - Source: Any
         - Port: *
         - Destination: Any
         - Service: HTTP
         - Action: Allow
         - Priority: 1010
         - Name: `allow-http`
   - Public IP address: Enabled
   - Load balancer:
     - Load balancing: Azure load balancer
     - Create new:
       - Name: `vmss-lb`

6. On **Management**, set:
   - **Boot diagnostics**: Disable

7. On **Health** and **Advanced**, leave defaults.
8. Click **Review + create** > **Create**

---

## Task 4: Scale Azure Virtual Machine Scale Sets

1. Go to `vmss1` > **Availability + scale** > **Scaling**
2. Set **Custom autoscale**:
   - Mode: Scale based on metric

### Scale Out Rule

| Setting                                | Value         |
|----------------------------------------|---------------|
| Metric source                          | Current resource (vmss1) |
| Metric name                            | Percentage CPU |
| Operator                               | Greater than   |
| Threshold                              | 70             |
| Duration (minutes)                     | 10             |
| Statistic                              | Average        |
| Operation                              | Increase percent by |
| Cool down (minutes)                    | 5              |
| Percentage                             | 50             |

- Click **Add**, then **Save**

### Scale In Rule

| Setting                                | Value         |
|----------------------------------------|---------------|
| Operator                               | Less than     |
| Threshold                              | 30            |
| Operation                              | Decrease percent by |
| Percentage                             | 50            |

- Click **Add**, then **Save**

### Instance Limits

| Setting        | Value |
|----------------|-------|
| Minimum        | 2     |
| Maximum        | 10    |
| Default        | 2     |

- Click **Save**

---

## Task 5: Create a virtual machine using Azure PowerShell (Optional)

1. Open Cloud Shell (https://shell.azure.com) > select **PowerShell**
2. Run:

```powershell
New-AzVm `
 -ResourceGroupName 'az104-rg8' `
 -Name 'myPSVM' `
 -Location 'East US' `
 -Image 'Win2019Datacenter' `
 -Zone '3' `
 -Size 'Standard_D2s_v3' `
 -Credential (Get-Credential)
```

3. Verify with:

```powershell
Get-AzVM -ResourceGroupName 'az104-rg8' -Status
```

4. Deallocate:

```powershell
Stop-AzVM -ResourceGroupName 'az104-rg8' -Name 'myPSVM'
```

---

## Task 6: Create a virtual machine using the CLI (Optional)

1. Open Cloud Shell > select **Bash**
2. Run:

```bash
az vm create --name myCLIVM --resource-group az104-rg8 --image Ubuntu2204 --admin-username localadmin --generate-ssh-keys
```

3. Verify:

```bash
az vm show --name myCLIVM --resource-group az104-rg8 --show-details
```

4. Deallocate:

```bash
az vm deallocate --resource-group az104-rg8 --name myCLIVM
```

---

## Cleanup your resources

- In the Azure portal: Delete the resource group.
- PowerShell:

```powershell
Remove-AzResourceGroup -Name az104-rg8
```

- CLI:

```bash
az group delete --name az104-rg8
```

---

## Implementation Video

[![Watch the video](./Deploy%20and%20Scale%20Azure%20Virtual%20Machines%20and%20Scale%20Sets.png)](https://youtu.be/Of2chThRwPE)

---

## Learn more

- [Create a Windows virtual machine in Azure](https://learn.microsoft.com/training/modules/create-windows-virtual-machine-in-azure/)
- [Build a scalable application with Virtual Machine Scale Sets](https://learn.microsoft.com/training/modules/build-app-with-scale-sets/)
- [Connect to virtual machines through the Azure portal using Azure Bastion](https://learn.microsoft.com/en-us/training/modules/connect-vm-with-azure-bastion/)


---

## Key takeaways

- Azure virtual machines are on-demand, scalable computing resources.
- Azure virtual machines provide both vertical and horizontal scaling options.
- Configuring Azure virtual machines includes choosing an OS, size, storage, and networking.
- Azure Virtual Machine Scale Sets let you create and manage a group of load-balanced VMs.
- VMs in a Scale Set are created from the same image and configuration.
- Scale Sets can scale automatically based on demand or schedule.
