# Azure Policy Setup for Allowing Specific Resources and Regions

This README provides a detailed guide on how to set up Azure Policy using Azure CLI to restrict resource creation to specific types and regions. This is crucial for maintaining compliance and governance in your Azure environment.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Creating a Policy Definition](#creating-a-policy-definition)
- [Assigning the Policy](#assigning-the-policy)
- [Testing the Configuration](#testing-the-configuration)

## Prerequisites

Before you begin, ensure you have the following:

- An active Azure subscription.
- Azure CLI installed and configured on your machine.
- Appropriate permissions to create policies in your Azure environment.

## Creating a Policy Definition

To restrict resource creation to specific types and regions, you need to create a policy definition. This can be done using a JSON file that outlines the rules.

1. **Create a JSON Policy File**: Open your preferred code editor and create a new file named `AzureResourcePolicy.json`. Below is an example policy that allows only specific resource types in designated regions:

   ```json
   {
     "mode": "Indexed",
     "policyRule": {
       "if": {
         "allOf": [
           {
             "field": "type",
             "in": [
               "Microsoft.Storage/storageAccounts",
               "Microsoft.Network/virtualNetworks"
             ]
           },
           {
             "field": "location",
             "notIn": [
               "eastus",
               "westus"
             ]
           }
         ]
       },
       "then": {
         "effect": "deny"
       }
     },
     "parameters": {}
   }
   ```

2. **Save the File**: Ensure the file is saved in an accessible location.

3. **Create the Policy Definition Using Azure CLI**: Open your terminal and run the following command, replacing `<path-to-your-json-file>` with the actual path:

   ```bash
   az policy definition create --name 'AllowSpecificResources' \
       --display-name 'Allow Specific Resources' \
       --description 'Policy to allow specific resources in designated regions.' \
       --rules '@<path-to-your-json-file>' \
       --mode Indexed
   ```

## Assigning the Policy

Once you have your policy definition ready, the next step is to assign it to your desired scope (subscription or resource group).

1. **Login to Azure**: Open your terminal and log in using:

   ```bash
   az login
   ```

2. **Set Your Subscription Context**: If you have multiple subscriptions, set the context to your desired subscription:

   ```bash
   az account set --subscription "<Your Subscription ID>"
   ```

3. **Assign the Policy**: Use the following command to assign the policy to your subscription or resource group:

   - To assign it at the subscription level:

     ```bash
     az policy assignment create --name 'AllowSpecificResourcesAssignment' \
         --display-name 'Allow Specific Resources Assignment' \
         --scope '/subscriptions/<Your Subscription ID>' \
         --policy 'AllowSpecificResources'
     ```

   - To assign it at the resource group level:

     ```bash
     az policy assignment create --name 'AllowSpecificResourcesAssignment' \
         --display-name 'Allow Specific Resources Assignment' \
         --scope '/subscriptions/<Your Subscription ID>/resourceGroups/<Your Resource Group Name>' \
         --policy 'AllowSpecificResources'
     ```

## Testing the Configuration

To verify that your policy is working as intended, attempt to create a resource that violates the policy.

1. **Create a Resource**: Try creating a storage account in an unauthorized region (e.g., `centralus`):

   ```bash
   az storage account create --name "<YourStorageAccountName>" \
       --resource-group "<YourResourceGroupName>" \
       --location "centralus" \
       --sku Standard_LRS
   ```

2. **Check for Errors**: You should receive an error indicating that the resource cannot be created due to policy restrictions.

3. **Review Compliance**: You can check compliance status by running:

   ```bash
   az policy assignment list --scope '/subscriptions/<Your Subscription ID>'
   ```

## Conclusion

By following these steps, you can effectively manage which resources can be deployed in your Azure environment based on type and location. This ensures compliance with organizational policies and regulatory requirements, helping maintain a secure and organized cloud infrastructure.

Citations:
[1] https://learn.microsoft.com/en-us/azure/governance/policy/assign-policy-azurecli
[2] https://stackoverflow.com/questions/44737121/azure-arm-policies-using-the-predefined-definitions-via-cli
[3] https://learn.microsoft.com/en-us/cli/azure/policy/assignment?view=azure-cli-latest
[4] https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/manage/azure-server-management/common-policies
[5] https://www.blinkops.com/blog/managing-policies-with-the-azure-cli
[6] https://github.com/MicrosoftDocs/azure-docs/blob/main/articles/governance/policy/tutorials/create-and-manage.md
[7] https://marcelzehner.ch/2016/09/28/azure-resource-policies-a-complete-walkthrough/
[8] https://learn.microsoft.com/en-in/azure/governance/policy/overview