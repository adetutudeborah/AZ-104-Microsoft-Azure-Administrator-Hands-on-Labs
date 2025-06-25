# Lab 07 - Network Traffic Management with Azure Load Balancer and Application Gateway

## Scenario

Your organization has a public website. You need to load balance incoming public requests across different virtual machines. You also need to provide images and videos from different virtual machines. You plan on implementing an Azure Load Balancer and an Azure Application Gateway. All resources are in the same region.

## Aim

The aim of this lab is to implement and configure Azure network traffic management solutions by deploying and testing an Azure Load Balancer (Layer 4) and an Azure Application Gateway (Layer 7) in a simulated public-facing web environment.

## What You Will Achieve by the End of the Lab

- Deploy a virtual network, subnets, and 3 virtual machines using an ARM template.
- Set up a public Azure Load Balancer to distribute web traffic across VMs.
- Configure health probes and load balancing rules.
- Create a dedicated subnet for the Azure Application Gateway.
- Set up the Application Gateway with path-based routing:
    - Route /image/ requests to one VM
    - Route /video/ requests to another VM
- Test the public IPs to verify traffic routing works as expected.

---

## Instructions

## Task 1: Use a Template to Provision an Infrastructure

In this task, you will use a template to deploy one virtual network, one network security group, and three virtual machines.

1. Download the `\Labs\Lab07` lab files (template and parameters).
2. Sign in to the Azure portal - [https://portal.azure.com](https://portal.azure.com)
3. Search for and select **Deploy a custom template**.
4. Select **Build your own template in the editor**.
5. On the edit template page, click **Load file**.
6. Select `\Allfiles\Lab07\az104-07-vms-template.json` and click **Open**.
7. Click **Save**.
8. Click **Edit parameters**, then load `\Allfiles\Lab07\az104-07-vms-parameters.json`.
9. Click **Save**.

**Deployment Settings:**

| Setting         | Value                   |
|-----------------|-------------------------|
| Subscription    | Your Azure subscription |
| Resource group  | `az104-rg7` (create new if necessary) |
| Password        | Provide a secure password |

> **Note:** If you get a VM size error, select a SKU available in your subscription with at least 2 cores.

10. Click **Review + Create**, then **Create**.

> Note: Wait for the deployment to complete before moving to the next task. The deployment should take approximately 5 minutes.

> Note: Review the resources being deployed. There will be one virtual network with three subnets. Each subnet will have a virtual machine

---

## Task 2: Configure an Azure Load Balancer

In this task, you implement a Load Balancer in front of two Azure VMs.

![Load Balancer - Architecture diagram](./Load%20balancer%20-%20Architecture%20diagram.png)


1. In the Azure portal, search for **Load balancers**, then click **+ Create**.

**Load Balancer Settings:**

| Setting         | Value            |
|-----------------|------------------|
| Subscription    | Your subscription |
| Resource group  | `az104-rg7`      |
| Name            | `az104-lb`       |
| Region          | Same as VMs      |
| SKU             | Standard         |
| Type            | Public           |
| Tier            | Regional         |

2. Go to **Frontend IP configuration** > **Add a frontend IP configuration**:

**Frontend IP Settings:**

| Setting         | Value           |
|-----------------|-----------------|
| Name            | `az104-fe`      |
| IP type         | IP address      |
| Public IP       | Create new      |

**Public IP Settings:**

| Setting         | Value            |
|-----------------|------------------|
| Name            | `az104-lbpip`    |
| SKU             | Standard         |
| Tier            | Regional         |
| Assignment      | Static           |
| Routing Pref.   | Microsoft network |

3. Click **Next: Backend pools** > **Add a backend pool**:

**Backend Pool Settings:**

| Setting                 | Value              |
|-------------------------|--------------------|
| Name                    | `az104-be`         |
| Virtual network         | `az104-06-vnet1`   |
| Backend pool config     | NIC                |
| Add VMs:                | `az104-06-vm0`, `az104-06-vm1` |

4. Click **Next: Inbound rules** > **+ Add**:

**Inbound Rule Settings:**

| Setting              | Value               |
|----------------------|---------------------|
| Name                 | `az104-lbrule`      |
| IP Version           | IPv4                |
| Frontend IP Address  | `az104-fe`          |
| Backend pool         | `az104-be`          |
| Protocol             | TCP                 |
| Port                 | 80                  |
| Backend port         | 80                  |
| Health probe         | Create new          |

**Health Probe Settings:**

| Name        | Protocol | Port | Interval |
|-------------|----------|------|----------|
| `az104-hp`  | TCP      | 80   | 5        |

- Session persistence: None
- Idle timeout: 4 min
- TCP reset: Disabled
- Floating IP: Disabled
- Outbound SNAT: Recommended

5. Copy the public IP address from the Load Balancer frontend.
6. Navigate to the IP in a browser — ensure you see **"Hello World from az104-06-vm0"** or **vm1**.
7. Refresh to verify round-robin load balancing.

---

## Task 3: Configure an Azure Application Gateway


![Application Gateway - Architecture diagram](./Application%20gateway%20-%20Architecture%20diagram.png)

1. Go to **Virtual networks** > `az104-06-vnet1` > **Subnets** > **+ Subnet**

**New Subnet Settings:**

| Setting         | Value          |
|-----------------|----------------|
| Name            | `subnet-appgw` |
| Start Address   | `10.60.3.224`  |
| Size            | `/27`          |

> **Note:** Application Gateway requires a dedicated `/27` subnet.

2. Go to **Application gateways** > **+ Create**.

**Basics Tab:**

| Setting            | Value             |
|--------------------|-------------------|
| Name               | `az104-appgw`     |
| Region             | Same as VMs       |
| Tier               | Standard V2       |
| Autoscaling        | No                |
| Instance count     | 2                 |
| HTTP2              | Disabled          |
| Virtual network    | `az104-06-vnet1`  |
| Subnet             | `subnet-appgw`    |

**Frontend Tab:**

| Setting               | Value          |
|-----------------------|----------------|
| Frontend IP type      | Public         |
| Public IP address     | Add new        |
| Name                  | `az104-gwpip`  |
| Availability zone     | 1              |

**Backends Tab:**

- Add backend pool: `az104-appgwbe`
  - VMs: `az104-06-nic1`, `az104-06-nic2`
- Add backend pool: `az104-imagebe`
  - VM: `az104-06-nic1`
- Add backend pool: `az104-videobe`
  - VM: `az104-06-nic2`

**Configuration Tab:**

- **Rule Name:** `az104-gwrule`
- **Listener Name:** `az104-listener`
- **Protocol:** HTTP
- **Port:** 80
- **Frontend IP:** Public IPv4
- **Listener Type:** Basic

**Backend Target Settings:**

- Target: `az104-appgwbe`
- Backend Settings: `az104-http` (new)

**Path-Based Routing Rules:**

| Path        | Target Name | Backend Target     |
|-------------|-------------|--------------------|
| `/image/*`  | images      | `az104-imagebe`    |
| `/video/*`  | videos      | `az104-videobe`    |

3. Click **Next > Review + create** > **Create**
4. Wait 5–10 minutes for the deployment to finish.

**Test the Application Gateway:**

1. Copy the frontend public IP.
2. Test the following URLs in a browser:

- `http://<frontend-ip>/image/` → should go to image server (vm1)
- `http://<frontend-ip>/video/` → should go to video server (vm2)

---

## Cleanup

> To avoid charges, delete the resource group when done.

- Azure portal: Delete resource group `az104-rg7`
- PowerShell: `Remove-AzResourceGroup -Name az104-rg6`
- CLI: `az group delete --name az104-rg6`

---
## Implementation Video

[![Watch the video](./Network%20Traffic%20Management%20with%20Azure%20Load%20Balancer%20and%20Application%20Gateway%20-%20Thumbnail.png)]()

---

## Learn More with Self-Paced Training

- [Improve application scalability and resiliency by using Azure Load Balancer](https://learn.microsoft.com/training/modules/improve-app-scalability-resiliency-with-load-balancer/)
- [Load balance your web service traffic with Application Gateway.](https://learn.microsoft.com/training/modules/load-balance-web-traffic-with-application-gateway/)

---

## Key Takeaways

- Azure Load Balancer is used to spread network traffic across multiple virtual machines. It works at Layer 4 (Transport Layer), handling TCP and UDP traffic.

- A Public Load Balancer handles traffic from the internet, while a Private Load Balancer is used inside your network with private IPs.

- The Basic Load Balancer is good for small apps with simple needs. The Standard Load Balancer is better for high-performance and more reliable systems.

- Azure Application Gateway is for managing web traffic and works at Layer 7 (Application Layer).

- The Standard tier of Application Gateway supports full web traffic management. The WAF (Web Application Firewall) tier adds security by blocking harmful traffic.

- Application Gateway can route traffic based on things like the URL path (e.g., /images/) or host name.

---
