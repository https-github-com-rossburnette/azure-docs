---
title: Remove machines from Azure Automation Update Management
description: This article tells how to remove Azure and non-Azure machines managed with Update Management.
services: automation
ms.topic: conceptual
ms.date: 10/26/2021
ms.custom: mvc
---
# Remove VMs from Update Management

When you're finished managing updates on your Azure or non-Azure machines in your environment, you can stop managing them with the [Update Management](overview.md) feature. To stop managing them, you will edit the saved search query `MicrosoftDefaultComputerGroup` in your Log Analytics workspace that is linked to your Automation account.

## Sign into the Azure portal

Sign in to the [Azure portal](https://portal.azure.com).

## To remove your machines

1. In the Azure portal, launch **Cloud Shell** from the top navigation of the Azure portal. If you are unfamiliar with Azure Cloud Shell, see [Overview of Azure Cloud Shell](../../cloud-shell/overview.md).

2. Use the following method to identify the UUID of an Azure virtual machine or non-Azure machine that you want to remove from management.

   # [Azure VM](#tab/azure-vm)

   ```azurecli
   az vm show -g MyResourceGroup -n MyVm -d
   ```

   # [Non-Azure machine](#tab/non-azure-machine)

   ```kusto
   Heartbeat
   | where TimeGenerated > ago(30d)
   | where ComputerEnvironment == "Non-Azure"
   | summarize by Computer, VMUUID
   ```

   ---

3. In the Azure portal, navigate to **Log Analytics workspaces**. Select your workspace from the list.

4. In your Log Analytics workspace, select **Computer Groups** from the left-hand menu.

5. From **Computer Groups** in the right-hand pane, the **Saved groups** tab is shown by default.

6. From the table, click the icon **Run query** to the right of the item **MicrosoftDefaultComputerGroup** with the **Legacy category** value **Updates**.

7. In the query editor, review the query and find the UUID for the machine. Remove the UUID for the machine and repeat the steps for any other machines you want to remove.

   > [!NOTE]
   > For added protection, before making edits be sure to make a copy of the query. Then you can restore it if a problem occurs.

   If you want to start with the original query and re-add machines in support of a cleanup or maintenance activity, copy the following query:

   ```kusto
   Heartbeat
   | where Computer in~ ("") or VMUUID in~ ("")
   | distinct Computer
   ```

8. Save the saved search when you're finished editing it by selecting **Save > Save as function** from the top bar. When prompted, specify the following:

    * **Name**: Updates__MicrosoftDefaultComputerGroup
    * **Save as computer Group** is selected
    * **Legacy category**: Updates

>[!NOTE]
>Machines are still shown after you have unenrolled them because we report on all machines assessed in the last 24 hours. After removing the machine, you need to wait 24 hours before they are no longer listed.

## Next steps

To re-enable managing your Azure or non-Azure machine, see [Enable Update Management by browsing the Azure portal](enable-from-portal.md) or [Enable Update Management from an Azure VM](enable-from-vm.md).