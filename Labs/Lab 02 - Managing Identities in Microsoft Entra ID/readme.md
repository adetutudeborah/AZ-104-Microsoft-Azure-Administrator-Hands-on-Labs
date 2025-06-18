# Lab 02: Manage Microsoft Entra ID Identities

## Scenario

Your organization is building a new lab environment for pre-production testing of apps and services. A few engineers are being hired to manage the lab environment, including the virtual machines. To allow the engineers to authenticate by using Microsoft Entra ID, you have been tasked with provisioning users and groups. To minimize administrative overhead, membership of the groups should be updated automatically based on job titles.


## Aim
The aim of this lab is to manage user and group identities in Microsoft Entra ID (formerly Azure Active Directory). We will:

- Create and configure both internal and external user accounts

- Assign important user properties like job title and department

- Set up security groups to control access

- Understand the difference between static and dynamic group membership


## Requirements

To complete this lab, you need:

- An active Azure subscription (preferably one that allows access to Microsoft Entra ID features).

- Access to the Azure Portal: https://portal.azure.com.

- Entra ID Premium P1 or P2 license if you want to explore dynamic groups (Optional but helpful).


## Architecture Diagram

![Lab 01 - Architecture diagram](./Lab%2002%20-%20Architecture%20diagram.png)

---

## Instructions

## Task 1: Create and Configure User Accounts

In this task, you will create and configure user accounts. User accounts will store user data such as name, department, location, and contact information.

1. Sign in to the Azure portal: https://portal.azure.com
2. To proceed, select **Cancel** on the Welcome splash screen.

> **Tip:** If you are new to Azure, search for and select **Quickstart Center**. Watch the *Getting started in the Azure portal* video.

3. Search for and select **Microsoft Entra ID**.
4. Select the **Overview** blade and then the **Manage tenants** tab.

> **Did you know?** A tenant is a specific instance of Microsoft Entra ID containing accounts and groups.

5. Return to the Entra ID page.

### Create a New User

- Go to **Users** > **New user** > **Create new user**
- Fill in the following settings:

| Setting               | Value                   |
|-----------------------|-------------------------|
| User principal name   | az104-user1             |
| Display name          | az104-user1             |
| Auto-generate password| ✅                      |
| Account enabled       | ✅                      |
| Job title             | IT Lab Administrator    |
| Department            | IT                      |
| Usage location        | United States           |

- Select **Review + create** > **Create**
- Refresh the page to confirm creation

### Invite an External User

- Go to **New user** > **Invite an external user**
- Fill in the following settings:

| Setting               | Value                                |
|-----------------------|--------------------------------------|
| Email                 | *your email address*                 |
| Display name          | *your name*                          |
| Send invite message   | ✅                                   |
| Message               | Welcome to Azure and our group project |
| Job title             | IT Lab Administrator                 |
| Department            | IT                                   |
| Usage location        | United States                        |

- Select **Review + invite** > **Invite**
- Confirm that the invited user was created

> **Note:** It is unlikely you will be creating user accounts individually. Do you know how your organization plans to manage them?

---

## Task 2: Create Groups and Add Members

In this task, you create a group account. Group accounts can include user accounts or devices.

### Group Membership Types

- **Static:** Manual member assignment
- **Dynamic:** Auto-membership based on properties like job title

### Create a Group

1. In the Azure portal, go to **Microsoft Entra ID**
2. Under **Manage**, select **Groups**
3. Select **+ New group** and configure:

| Setting          | Value                              |
|------------------|------------------------------------|
| Group type       | Security                           |
| Group name       | IT Lab Administrators              |
| Group description| Administrators that manage the IT lab |
| Membership type  | Assigned                           |

> **Note:** Entra ID Premium P1 or P2 is required for dynamic groups.

4. Select **No owners selected** > Add yourself as the owner
5. Select **No members selected** > Add:
   - `az104-user1`
   - The external (guest) user you invited
6. Select **Create**
7. Refresh and verify group creation
8. Review **Members** and **Owners** info

> **Note:** Do you have a group management strategy for your organization?

---

## Implementation Video

[![Watch the video](./Manage%20Microsoft%20Entra%20ID%20Identities%20Image.png)]()

---

## Learn More 


---

## Key Takeaways

- A **tenant** represents your organization in Microsoft cloud services.
- Microsoft Entra ID includes **user** and **guest** accounts with role-based access.
- Groups help manage access and permissions. Two types:
  - **Security**
  - **Microsoft 365**
- Group membership can be **static** or **dynamic**.

---
