
# Lab 01: Implement a Azure Hub and Spoke Network Architecture

## Scenario

Your organization is migrating a web-based application to **Azure**. The first task is to set up virtual networks and subnets, followed by secure VNet peering.

## Aim

The aim of this lab is to design and implement a secure hub-and-spoke network architecture in Microsoft Azure using:

- Virtual Networks (VNets) for isolating resources

- Subnets to segment traffic (frontend, backend, and firewall zones)

- VNet Peering to enable private and secure communication between VNets

This structure mimics a production-grade Azure network setup where:

- The spoke network (app-vnet) hosts application components

- The hub network (hub-vnet) serves as the central point for shared services like firewalls or VPN gateways

## Requirements

- Create a **hub and spoke** network architecture.
- Two virtual networks:
  - `app-vnet`: Hosts the application.
    - Subnets:
      - `frontend`: Hosts the web servers.
      - `backend`: Hosts the database servers.
  - `hub-vnet`: Contains a subnet for the firewall.
- Establish secure and private communication via **Virtual Network Peering**.
- Both VNets must be in the **same Azure region**.


## Architecture Diagram

![Lab 01 - Architecture diagram](./Lab%2001%20-%20Architecture%20diagram.png)

---

## Instructions

> **Note:** You will need an Azure subscription with **Contributor** role to complete this exercise.

### Step 1: Create `app-vnet` with Subnets

1. Sign in to the Azure portal: https://portal.azure.com
2. Search for and select **Virtual Networks**.
3. Click **+ Create** and enter the following:

| Property              | Value         |
|-----------------------|---------------|
| Resource group        | `RG1`         |
| Virtual network name  | `app-vnet`    |
| Region                | `East US`     |
| IPv4 address space    | `10.1.0.0/16` |

4. Add the following subnets:

| Subnet Name | Address Range  |
|-------------|----------------|
| `frontend`  | `10.1.0.0/24`  |
| `backend`   | `10.1.1.0/24`  |

5. Leave all other settings at default.
6. Click **Review + Create**, then click **Create**.

### Step 2: Create `hub-vnet` with Subnet

1. Go back to **Virtual Networks** > **+ Create**.
2. Use the following configuration:

| Property              | Value              |
|-----------------------|--------------------|
| Resource group        | `RG1`              |
| Name                  | `hub-vnet`         |
| Region                | `East US`          |
| IPv4 address space    | `10.0.0.0/16`      |
| Subnet name           | `AzureFirewallSubnet` |
| Subnet address range  | `10.0.0.0/26`      |

3. Click **Review + Create**, then **Create**.
4. Verify both VNets and subnets were successfully deployed.

### Step 3: Configure VNet Peering

1. In the Azure portal, go to **Virtual Networks** and select `app-vnet`.
2. In the left pane, go to **Peerings** under Settings.
3. Click **+ Add** and configure the following:

| Property                     | Value               |
|------------------------------|---------------------|
| Remote peering link name     | `app-vnet-to-hub`   |
| Virtual network              | `hub-vnet`          |
| Local virtual network peering link name | `hub-to-app-vnet` |

4. Leave all other settings as default.
5. Click **Add**.
6. Once completed, verify the peering status shows **Connected**.

---

## Implementation

---

## Learn More

- [Introduction to Azure Virtual Networks (Microsoft Learn)](https://learn.microsoft.com/en-us/training/modules/introduction-to-azure-virtual-networks/)
- [Introduction to Hub & Spoke Network Architecture](https://www.linkedin.com/pulse/introduction-hub-spoke-network-architecture-kumoraicloud-zpzgf/)
- [Hub-spoke network topology with Azure Virtual WAN](https://learn.microsoft.com/en-us/azure/architecture/networking/architecture/hub-spoke-vwan-architecture)
- [What is Hub and Spoke Topology?](https://www.cbtnuggets.com/blog/technology/networking/what-is-hub-and-spoke-topology)

---

## Key Takeaways

- **Azure VNets** provide a secure and isolated environment for cloud resources.
- Use **non-overlapping CIDR blocks** when designing VNets.
- Subnets allow for segmentation and security zoning.
- Services like **Azure Firewall** require dedicated subnets.
- **VNet Peering** allows fast and private communication between virtual networks.
