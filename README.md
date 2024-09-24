# Azure Policy Setup for Allowing Specific Resources and Regions

This README provides a comprehensive guide on how to set up two distinct Azure Policies using Azure CLI: one for allowing resource creation in specific regions and another for restricting resource type creation. These policies are essential for maintaining compliance and governance in your Azure environment.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Creating the Policies](#creating-the-policies)
  - [Policy 1: Allow Specific Regions](#policy-1-allow-specific-regions)
  - [Policy 2: Restrict Resource Types](#policy-2-restrict-resource-types)
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

### Policy 1: Allow Specific Regions

To create a policy that restricts resource deployment to specific regions, follow these steps:

1. **Create a JSON Policy File**: Open your preferred code editor and create a new file named `AllowSpecificRegionsPolicy.json`. Below is an example policy that allows only `eastus` and `westus` regions:

   ```json
   {
     "mode": "Indexed",
     "policyRule": {
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
     },
     "parameters": {}
   }
   ```

2. **Create the Policy Definition Using Azure CLI**: Run the following command, replacing `<path-to-your-json-file>` with the actual path:

   ```bash
   az policy definition create --name 'AllowSpecificRegions' \
       --display-name 'Allow Specific Regions' \
       --description 'Policy to allow resource creation only in specific regions.' \
       --rules '@<path-to-your-json-file>' \
       --mode Indexed
   ```

### Policy 2: Restrict Resource Types

Next, create a policy that restricts the types of resources that can be created.

1. **Create a JSON Policy File**: Create another file named `RestrictResourceTypesPolicy.json`. Below is an example policy that only allows `Microsoft.Storage/storageAccounts` and `Microsoft.Network/virtualNetworks`:

   ```json
   {
     "mode": "Indexed",
     "policyRule": {
       "if": {
         "not": {
           "field": "type",
           "in": [
             "Microsoft.Storage/storageAccounts",
             "Microsoft.Network/virtualNetworks"
           ]
         }
       },
       "then": {
         "effect": "deny"
       }
     },
     "parameters": {}
   }
   ```

2. **Create the Policy Definition Using Azure CLI**: Run the following command:

   ```bash
   az policy definition create --name 'RestrictResourceTypes' \
       --display-name 'Restrict Resource Types' \
       --description 'Policy to restrict resource creation to specific types.' \
       --rules '@<path-to-your-json-file>' \
       --mode Indexed
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

3. **Assign the Allow Specific Regions Policy**:

   ```bash
   az policy assignment create --name 'AllowSpecificRegionsAssignment' \
       --display-name 'Allow Specific Regions Assignment' \
       --scope '/subscriptions/<Your Subscription ID>' \
       --policy 'AllowSpecificRegions'
   ```

4. **Assign the Restrict Resource Types Policy**:

   ```bash
   az policy assignment create --name 'RestrictResourceTypesAssignment' \
       --display-name 'Restrict Resource Types Assignment' \
       --scope '/subscriptions/<Your Subscription ID>' \
       --policy 'RestrictResourceTypes'
   ```

### Assigning to a Resource Group

To assign these policies at the resource group level, use similar commands but specify the resource group scope:

1. **Assign the Allow Specific Regions Policy**:

   ```bash
   az policy assignment create --name 'AllowSpecificRegionsAssignmentRG' \
       --display-name 'Allow Specific Regions Assignment for RG' \
       --scope '/subscriptions/<Your Subscription ID>/resourceGroups/<Your Resource Group Name>' \
       --policy 'AllowSpecificRegions'
   ```

2. **Assign the Restrict Resource Types Policy**:

   ```bash
   az policy assignment create --name 'RestrictResourceTypesAssignmentRG' \
       --display-name 'Restrict Resource Types Assignment for RG' \
       --scope '/subscriptions/<Your Subscription ID>/resourceGroups/<Your Resource Group Name>' \
       --policy 'RestrictResourceTypes'
   ```

## Testing the Configuration

To verify that your policies are working as intended, attempt to create resources that violate these policies.

1. **Test Region Restriction**: Try creating a storage account in an unauthorized region (e.g., `centralus`):

   ```bash
   az storage account create --name "<YourStorageAccountName>" \
       --resource-group "<YourResourceGroupName>" \
       --location "centralus" \
       --sku Standard_LRS
   ```

2. **Test Resource Type Restriction**: Try creating an unauthorized resource type (e.g., `Microsoft.Compute/virtualMachines` if not allowed):

   ```bash
   az vm create --resource-group "<YourResourceGroupName>" \
       --name "<YourVMName>" \
       --image UbuntuLTS
   ```

3. **Check for Errors**: You should receive errors indicating that resource creation is denied due to policy restrictions.

4. **Review Compliance**: You can check compliance status by running:

   ```bash
   az policy assignment list --scope '/subscriptions/<Your Subscription ID>'
   ```

## Conclusion

By following these steps, you can effectively manage which resources can be deployed in your Azure environment based on region and type. This ensures compliance with organizational policies and regulatory requirements, helping maintain a secure and organized cloud infrastructure.

Additional Sources:

 https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/manage/azure-server-management/common-policies

 https://learn.microsoft.com/en-in/azure/governance/policy/overview