---
title: 'Azure'
description: 'Connect to Sysdig'
---

This topic describes how to connect your Azure environment to Sysdig. You can connect single subscriptions or entire tenants using Terraform. 

<Accordion title='Cloud Security Posture Management (CSPM)'>

**Cloud Security Posture Management (CSPM):**

* Monitors and detects misconfigurations in your cloud resources.  
* Ensures your cloud environment complies with industry standards and regulations.  
* Provides a comprehensive inventory of all cloud assets, helping you maintain visibility and control over your environment.

To enable CSPM, connect your Azure environment.
</Accordion>

<Accordion title='Review Azure Roles and Permissions'>

### Security Principals

The onboarding process involves two [security principals](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/understand-security-principals):

* Installer: The primary security principal, either a User or a Service Principal. This security principal will be used to perform the onboarding. Sysdig does not have access to this security principal.  
* Sysdig: A Service Principal (robot user) created during onboarding with specific, less permissive roles. Sysdig will be given access to this security principal.

### Azure Role Types

Azure Identity and Access Management (IAM) is separated into two control planes:

* [Entra ID Roles](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/custom-overview): Applied to the entire Tenant.  
* [Azure RBAC Roles](https://learn.microsoft.com/en-us/azure/role-based-access-control/overview): Applied to the Subscription or Management Group being onboarded.

## Prerequisites
</Accordion>

* Sysdig Secure SaaS with Admin permissions  
* Terraform v1.5.0+ installed  
* Azure CLI installed. See [How to install the Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli).  
* A security principal with the permissions required to install, as mentioned in [Security Principals](https://docs.sysdig.com/en/sysdig-secure/connect-azure/#security-principals). For the required permissions, see [Permissions Required to Install](https://docs.sysdig.com/en/sysdig-secure/azure-permissions-and-resources/#base-azure-integration---cloud-security-posture-management-cspm). To grant permissions, you need:  
  * `SP_ID`: Your Installer security principal ID. To retrieve this, open the Azure CI and use the command `az ad sp list --display-name "terraform-runner" --query "[0].appId" --output tsv`.  
  * `ROOT_MANAGEMENT_GROUP_ID`: Your Root Management Group ID. To retrieve this, open the Azure CLI and use the command `az account management-group list --query "[].{name:name, id:id}" --output tsv`.

## Prepare Your Environment

### 1\. Configure Installation Permissions

Ensure the principal you log in to Azure with has the [necessary roles and permissions](https://docs.sysdig.com/en/sysdig-secure/azure-permissions-and-resources/#base-azure-integration---cloud-security-posture-management-cspm) to install. You can:

* Use an existing principal who meets the permissions requirements.  
* Create a new principal and set up permissions.  
* Add permissions to an existing principal.  
1. Log in to Azure.  
2. Check Entra ID Roles:  
   * Navigate to the Entra ID console and select Roles and Administrators.  
   * Verify and add necessary roles.  
3. Check Azure RBAC Roles:  
   * For Single Subscriptions: Navigate to Subscriptions, select the target subscription, and verify roles.  
   * For Management Groups: Navigate to Management Groups, select the target group, and verify roles.

### 2\. Authenticate and Configure Terraform

A common way to do this is:

1. Ensure you are logged in to the correct Tenant.  
   Log in using the Azure CLI:

az login \--tenant "TENANT\_ID\_OR\_DOMAIN"

2.   
3. You will be presented with a web page to select your user account. Be sure to log in as the user you configured in Step 1\.  
4. Confirm you are logged in as the correct user by running:

az ad signed-in-user show

5.   
6. For alternative ways to authenticate Terraform, see the Terraform documentation: [Authenticating to Azure Active Directory](https://registry.terraform.io/providers/hashicorp/azuread/latest/docs#authenticating-to-azure-active-directory) and [Authenticating to Azure](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs#authenticating-to-azure).

### 3\. Collect your Azure Tenant ID and Subscription ID

#### Tenant ID

1. Sign in to the Azure portal.  
2. Navigate to Microsoft Entra ID \> Properties.  
3. Scroll down to the Tenant ID section.  
4. Find your tenant ID in the box.  
5. Select the Copy to clipboard icon shown next to the Tenant ID.  
6. Store this value. You can paste this value into a text document or other location.

#### Subscription ID

1. Sign in to the Azure portal.  
2. Under the Azure services heading, select Subscriptions. If you don’t see Subscriptions here, use the search box to find it.  
3. Find the subscription in the list, and note the Subscription ID shown in the second column. If no subscriptions appear, or you don’t see the right one, you may need to switch directories to show the subscriptions from a different Microsoft Entra tenant.  
4. To easily copy the Subscription ID, select the subscription name to display more details. Select the Copy to clipboard icon shown next to the Subscription ID in the Essentials section. You can paste this value into a text document or other location.

## Install Azure Using the Wizard

1. Log in to Sysdig Secure.  
2. Select Integrations \> Cloud Accounts \> Azure and click Add Azure Account on the top right corner.  
3. Connect your Azure [Tenant](https://docs.sysdig.com/en/sysdig-secure/connect-azure/#tenant-multi-subscription) or [Single Subscription](https://docs.sysdig.com/en/sysdig-secure/connect-azure/#single-subscription).  
   * This enables CSPM and lets you onboard Vulnerability Management and CDR after completing.

### Tenant Multi-Subscription

1. Enter your:  
   1. Tenant ID: The ID of the tenant you want to onboard.  
   2. Subscription ID: The ID of the subscription where the Sysdig resources will be created.  
2. Specify Management Groups:  
   1. For onboarding the entire Tenant: Enter Root Management Group ID.  
   2. For a subset: Enter Management Group IDs in a comma-separated list.  
3. Generate and apply the Terraform code:  
   1. Create a `main.tf` file.  
   2. Copy the snippet provided into the file.  
   3. Run the command: `terraform init && terraform apply`.

Within an hour after deployment, your accounts will appear on the Cloud Accounts page.

### Single Subscription

1. Enter your:  
   1. Tenant ID: The ID of the tenant which contains the subscription you want to onboard.  
   2. Subscription ID: The ID of the subscription you want to onboard.  
2. Generate and apply the Terraform code:  
   1. Create a `main.tf` file.  
   2. Copy the snippet provided into the file.  
   3. Run the command: `terraform init && terraform apply`.

Within an hour after deployment, your accounts will appear on the Cloud Accounts page.

## Check the Connection Status

Within 5 minutes, after you apply Terraform, your accounts will appear on the Sysdig Cloud Accounts page. You can add more features after this initial connection by following instructions to [Add New Features](https://docs.sysdig.com/en/azure/add-new-features).

You can verify your CSPM configuration by checking the connection status.

1. In Sysdig Secure, select Integrations \> Cloud Accounts \> Azure.  
   The Status column shows the overall connection status:  
   * Connected  
   * Error  
   * Unknown  
2. Select the desired account to review the individual services in the detail drawer.  
   The health status for CSPM configuration is given below:

| CSPM Status | Description |
| ----- | ----- |
| Healthy ✅ | The account has been successfully connected, and all the resources have been scanned. |
| Error ❌ | Authentication errors. For example: Invalid account ID Invalid client secret Invalid access credentials Access token errors Deny policy created by the user is preventing Sysdig from collecting resources The scan takes too long and eventually times out. Unknown error |

