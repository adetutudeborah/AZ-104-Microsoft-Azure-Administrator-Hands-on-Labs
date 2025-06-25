# Lab 06 - Configure Intersite Network Connectivity

## Scenario

Your organization segments core IT apps and services (such as DNS and security services) from other parts of the business, including your manufacturing department. However, in some scenarios, apps and services in the core area need to communicate with apps and services in the manufacturing area. In this lab, you configure connectivity between the segmented areas. This is a common scenario for separating production from development or separating one subsidiary from another.

---

## Aim

The primary goal of this lab is to establish and test secure communication between two isolated Azure virtual networks (VNets), one representing core IT services and the other representing manufacturing services. This scenario simulates a common real-world setup where network segmentation is used for security or organizational boundaries, yet interconnectivity is occasionally required.


## What You Will Achieve by the End of the Lab

- Deploy two virtual networks (VNets) in Azure.
- Create virtual machines (VMs) in separate VNets.
- Use Azure Network Watcher to verify connectivity status.
- Implement virtual network peering to enable secure communication between the VNets.
- Use Azure PowerShell to validate network traffic paths.
- Define a custom route to direct traffic via a future Network Virtual Appliance (NVA).


## Architecture Diagram

![Lab 06 - Architecture diagram](./Lab%2006%20-%20Architecture%20Diagram.png)

---

## Task 1: Create a Core Services Virtual Machine and Virtual Network

In this task, you create a core services virtual network with a virtual machine.

1. Sign in to the Azure portal - https://portal.azure.com.
2. Search for and select **Virtual Machines**.
3. Click **Create** > **Azure Virtual Machine**.
4. On the **Basics** tab:

| Setting              | Value                               |
|----------------------|-------------------------------------|
| Subscription         | your subscription                   |
| Resource group       | az104-rg5 *(Create if necessary)*   |
| Virtual machine name | CoreServicesVM                      |
| Region               | (US) East US                        |
| Availability options | No infrastructure redundancy required |
| Security type        | Standard                            |
| Image                | Windows Server 2019 Datacenter: x64 Gen2 |
| Size                 | Standard_DS2_v3                     |
| Username             | localadmin                          |
| Password             | Provide a complex password          |
| Public inbound ports | None                                |

5. Select **Next: Disks >**, accept defaults.
6. On **Networking** tab:
   - Select **Create new** under Virtual Network.
   - Use:

| Setting              | Value             |
|----------------------|-------------------|
| Name                 | CoreServicesVnet  |
| Address range        | 10.0.0.0/16       |
| Subnet Name          | Core              |
| Subnet address range | 10.0.0.0/24       |

7. On the **Monitoring** tab:
   - Set **Boot Diagnostics** to **Disable**.

8. Click **Review + Create**, then **Create**.

> 💡 *You created the virtual network while creating the VM. You could also create the network separately first.*

---

## Task 2: Create a Virtual Machine in a Different Virtual Network

In this task, you create a manufacturing services virtual network with a virtual machine.

1. Go to **Virtual Machines** > **Create** > **Azure Virtual Machine**.
2. On the **Basics** tab:

| Setting              | Value                              |
|----------------------|------------------------------------|
| Subscription         | your subscription                  |
| Resource group       | az104-rg5                          |
| Virtual machine name | ManufacturingVM                   |
| Region               | (US) East US                       |
| Security type        | Standard                           |
| Availability options | No infrastructure redundancy required |
| Image                | Windows Server 2019 Datacenter: x64 Gen2 |
| Size                 | Standard_DS2_v3                    |
| Username             | localadmin                         |
| Password             | Provide a complex password         |
| Public inbound ports | None                               |

3. Select **Next: Disks >**, accept defaults.
4. On **Networking** tab:
   - Select **Create new** under Virtual Network.
   - Use:

| Setting              | Value              |
|----------------------|--------------------|
| Name                 | ManufacturingVnet  |
| Address range        | 172.16.0.0/16      |
| Subnet Name          | Manufacturing      |
| Subnet address range | 172.16.0.0/24      |

5. On the **Monitoring** tab:
   - Set **Boot Diagnostics** to **Disable**.
6. Click **Review + Create**, then **Create**.

---

## Task 3: Use Network Watcher to Test Connection Between Virtual Machines

In this task, you verify that resources in peered virtual networks can communicate with each other. Network Watcher will be used to test the connection. Before continuing, ensure both virtual machines have been deployed and are running.

1. Ensure both VMs are **running**.
2. Go to **Network Watcher**.
3. Select **Connection troubleshoot** from the left menu.
4. Configure:

| Field             | Value            |
|-------------------|------------------|
| Source type       | Virtual machine  |
| Virtual machine   | CoreServicesVM   |
| Destination type  | Virtual machine  |
| Virtual machine   | ManufacturingVM  |
| Preferred IP      | Both             |
| Protocol          | TCP              |
| Destination port  | 3389             |
| Source port       | *leave blank*    |

5. Click **Run diagnostic tests**.

> 🔍 *Results will show "UnReachable" – expected because VNets are not peered yet.*

---

## Task 4: Configure Virtual Network Peerings

In this task, you create a virtual network peering to enable communications between resources in the virtual networks.

1. Go to **CoreServicesVnet** > **Peerings** > **+ Add**.
2. Configure:

| Setting                                    | Value                                  |
|--------------------------------------------|----------------------------------------|
| Peering link name (this VNet to remote)    | CoreServicesVnet-to-ManufacturingVnet |
| Virtual network                            | ManufacturingVnet                      |
| Allow remote VNet access                   | ✅ Yes (default)                        |
| Allow forwarded traffic                    | ✅ Yes                                  |
| Peering link name (remote VNet to this)    | ManufacturingVnet-to-CoreServicesVnet |
| Allow this VNet access                     | ✅ Yes                                  |
| Allow forwarded traffic                    | ✅ Yes                                  |

3. Click **Add**.
4. Go to both VNets and confirm **Peering Status** is **Connected**.

---

## Task 5: Use Azure PowerShell to Test the Connection

In this task, you retest the connection between the virtual machines in different virtual networks using Azure PowerShell.

1. From **CoreServicesVM**, copy the **Private IP Address**.
2. Go to **ManufacturingVM** > **Run Command** > **RunPowerShellScript**.
3. Run the following command:

```powershell
Test-NetConnection <CoreServicesVM private IP address> -port 3389
```

> 🟢 *The test should now **succeed** due to VNet peering.*

---

## Task 6: Create a Custom Route

In task 6, you want to create a custom route table to  control network traffic between the perimeter subnet and the internal core services subnet. A network virtual appliance will be installed in the core services subnet, and all traffic should be routed through it.

> By default, Azure handles routing automatically. Subnets in the same VNet, or peered VNets, can communicate with each other seamlessly. But in real-world deployments, you often need inspection, logging, or traffic filtering before allowing communication between systems. That’s where routing and Network Virtual Appliances (NVAs) come in. Custom Route Table gives you more autonomy and control over how traffic flows inside your Azure virtual network and helps you override default system routes


1. Go to **CoreServicesVnet** > **Subnets** > **+ Subnet**:

| Setting              | Value         |
|----------------------|---------------|
| Name                 | perimeter     |
| Address range        | 10.0.1.0/24   |

2. Go to **Route Tables** > **+ Create**:

| Setting              | Value         |
|----------------------|---------------|
| Subscription         | your subscription |
| Resource group       | az104-rg5     |
| Region               | East US       |
| Name                 | rt-CoreServices |
| Propagate routes     | No            |

3. After creation, go to the route table > **Routes** > **+ Add**:

| Setting              | Value                        |
|----------------------|------------------------------|
| Route name           | PerimetertoCore              |
| Destination type     | IP Addresses                 |
| Destination IP       | 10.0.0.0/16                  |
| Next hop type        | Virtual appliance            |
| Next hop address     | 10.0.1.7 (future NVA address)|

4. Go to **Subnets** > **+ Associate**:

| Setting          | Value             |
|------------------|-------------------|
| Virtual network  | CoreServicesVnet  |
| Subnet           | Core              |

> ✅ *You now have a user-defined route to direct traffic from the DMZ to an NVA.*

---

## Cleanup Your Resources

To avoid costs:

- Delete the resource group via portal:
  - Go to **Resource Groups**
  - Select `az104-rg5`
  - Click **Delete resource group**

- Or use PowerShell:
```powershell
Remove-AzResourceGroup -Name az104-rg5
```

- Or use Azure CLI:
```bash
az group delete --name az104-rg5
```

---

## Implementation Video

[![Watch the video](./Configure%20Intersite%20Network%20Connectivity%20Image.png)](https://youtu.be/KurJVawI4U4)

---

## Learn More with Self-Paced Training

- [Distribute services across Azure virtual networks using virtual network peering](https://learn.microsoft.com/en-us/training/modules/integrate-vnets-with-vnet-peering/)
- [Manage and control traffic flow with custom routes](https://learn.microsoft.com/en-us/training/modules/control-network-traffic-flow-with-routes/)

---

## Key Takeaways

- By default, resources in different VNets cannot communicate.
- VNet peering enables seamless connectivity between Azure virtual networks.
- Peered VNets appear as one for connectivity purposes.
- Traffic between VMs in peered VNets uses Microsoft's backbone infrastructure.
- System routes are automatically created; user-defined routes override or supplement them.
- Azure Network Watcher offers tools to monitor and diagnose IaaS network traffic.
