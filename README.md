
This guide incorporating the provided code snippets for setting up Azure Policies for allowed locations and allowed resource types using Azure CLI.


# Azure Policy Setup for Allowed Locations and Resource Types

This README provides a comprehensive guide on how to set up two distinct Azure Policies using Azure CLI: one for allowing resource creation in specific regions and another for restricting resource type creation. These policies are essential for maintaining compliance and governance in your Azure environment.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Creating the Policies](#creating-the-policies)
  - [Policy 1: Allowed Locations](#policy-1-allowed-locations)
  - [Policy 2: Allowed Resource Types](#policy-2-allowed-resource-types)
- [Assigning the Policies](#assigning-the-policies)
  - [Assigning to a Subscription](#assigning-to-a-subscription)
  - [Assigning to a Resource Group](#assigning-to-a-resource-group)
- [Testing the Configuration](#testing-the-configuration)

## Prerequisites

Before you begin, ensure you have the following:

- An active Azure subscription.
- Azure CLI installed and configured on your machine.
- Appropriate permissions to create and assign policies in your Azure environment.

## Creating the Policies

### Policy 1: Allowed Locations

To create a policy that restricts resource deployment to specific regions, follow these steps:

1. **Create the Allowed Locations Policy**: Run the following command in your terminal:

   ```bash
   az policy definition create --name "allowed-locations" \
       --display-name "Allowed Locations" \
       --description "This policy restricts resource deployment to specified Azure regions." \
       --rules '{
           "if": {
               "not": {
                   "field": "location",
                   "in": [
                       "eastus",
                       "westus"
                   ]
               }
           },
           "then": {
               "effect": "deny"
           }
       }' \
       --mode All
   ```

### Policy 2: Allowed Resource Types

Next, create a policy that restricts the types of resources that can be created.

1. **Create the Allowed Resource Types Policy**: Run the following command:

   ```bash
   az policy definition create --name "allowed-resource-types" \
       --display-name "Allowed Resource Types" \
       --description "This policy restricts the types of resources that can be deployed." \
       --rules '{
           "if": {
               "allOf": [
                   {
                       "field": "type",
                       "notIn": [
                           "Microsoft.Compute/virtualNetworks",
                           "Microsoft.Storage/storageAccounts"
                       ]
                   }
               ]
           },
           "then": {
               "effect": "deny"
           }
       }' \
       --mode All
   ```

## Assigning the Policies

### Assigning to a Subscription

To assign these policies at the subscription level, use the following commands:
1. **Login to Azure**: Open your terminal and log in using:
 ```bash
az login
   ```
2. **Set Your Subscription Context**: If you have multiple subscriptions, set the context to your desired subscription:

   ```bash
   az account set --subscription "<Your Subscription ID>"
   ```

3. **Assign the Allowed Locations Policy**:

   ```bash
   az policy assignment create --name 'AllowedLocationsAssignment' \
       --display-name 'Allowed Locations Assignment' \
       --scope '/subscriptions/<Your Subscription ID>' \
       --policy 'allowed-locations'
   ```

4. **Assign the Allowed Resource Types Policy**:

   ```bash
   az policy assignment create --name 'AllowedResourceTypesAssignment' \
       --display-name 'Allowed Resource Types Assignment' \
       --scope '/subscriptions/<Your Subscription ID>' \
       --policy 'allowed-resource-types'
   ```

### Assigning to a Resource Group

1. **Assign the Allowed Locations Policy**:

   ```bash
   az policy assignment create --name 'AllowedLocationsAssignmentRG' \
       --display-name 'Allowed Locations Assignment for RG' \
       --scope '/subscriptions/<Your Subscription ID>/resourceGroups/<Your Resource Group Name>' \
       --policy 'allowed-locations'
   ```

2. **Assign the Allowed Resource Types Policy**:

   ```bash
   az policy assignment create --name 'AllowedResourceTypesAssignmentRG' \
       --display-name 'Allowed Resource Types Assignment for RG' \
       --scope '/subscriptions/<Your Subscription ID>/resourceGroups/<Your Resource Group Name>' \
       --policy 'allowed-resource-types'
   ```

## Testing the Configuration

To verify that your policies are working as intended, attempt to create resources that violate these policies.
 1. **Test Region Restriction**: Try creating a storage account in an unauthorized region (e.g., centralus):
 ```bash
az storage account create --name "<YourStorageAccountName>" \
    --resource-group "<YourResourceGroupName>" \
    --location "centralus" \
    --sku Standard_LRS
```
2. **Test Resource Type Restriction**: Try creating an unauthorized resource type (e.g., Microsoft.Compute/virtualMachines if not allowed):
 ```bash
az vm create --resource-group "<YourResourceGroupName>" \
    --name "<YourVMName>" \
    --image UbuntuLTS
```
3. **Check for Errors**: You should receive errors indicating that resource creation is denied due to policy restrictions.
Review Compliance: You can check compliance status by running:
 ```bash
az policy assignment list --scope '/subscriptions/<Your Subscription ID>'
```
## Conclusion
By following these steps, you can effectively manage which resources can be deployed in your Azure environment based on region and type. This ensures compliance with organizational policies and regulatory requirements, helping maintain a secure and organized cloud infrastructure.


## Additional Sources:

 https://learn.microsoft.com/en-in/azure/governance/policy/overview

 https://learn.microsoft.com/en-us/azure/governance/policy/assign-policy-azurecli

 https://learn.microsoft.com/en-us/cli/azure/policy/assignment?view=azure-cli-latest

 https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/manage/azure-server-management/common-policies
