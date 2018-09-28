---
title: Enable Azure Disk Encryption with Azure AD App for Windows IaaS VMs (previous release)| Microsoft Docs
description: This article provides instructions on enabling Microsoft Azure Disk Encryption for Windows IaaS VMs.
author: mestew
ms.service: security
ms.subservice: Azure Disk Encryption
ms.topic: article
ms.author: mstewart
ms.date: 09/19/2018

---

#  Enable Azure Disk Encryption for Windows IaaS VMs (previous release)

**The new release of Azure Disk Encryption eliminates the requirement for providing an Azure AD application parameter to enable VM disk encryption. With the new release, you are no longer required to provide Azure AD credentials during the enable encryption step. All new VMs must be encrypted without the Azure AD application parameters using the new release. To view instructions to enable VM disk encryption using the new release, see [Azure Disk Encryption for Windows VMS](azure-security-disk-encryption-windows.md). VMs that were already encrypted with Azure AD application parameters are still supported and should continue to be maintained with the AAD syntax.**


You can enable many disk-encryption scenarios, and the steps may vary according to the scenario. The following sections cover the scenarios in greater detail for Windows IaaS VMs. Before you can use disk encryption, the [Azure Disk Encryption prerequisites](../security/azure-security-disk-encryption-prerequisites-aad.md) need to be completed. 

Take a [snapshot](../virtual-machines/windows/snapshot-copy-managed-disk.md) and/or back up  before disks are encrypted. Backups ensure that a recovery option is possible  if an unexpected failure occurs during encryption. VMs with managed disks require a backup before encryption occurs. Once a backup is made, you can use the Set-AzureRmVMDiskEncryptionExtension cmdlet to encrypt managed disks by specifying the -skipVmBackup parameter. For more information about how to back up and restore encrypted VMs, see the [Azure Backup](../backup/backup-azure-vms-encryption.md) article. 

>[!WARNING]
>In order to make sure the encryption secrets don’t cross regional boundaries, Azure Disk Encryption needs the Key Vault and the VMs to be co-located in the same region. Create and use a Key Vault that is in the same region as the VM to be encrypted. 


## Enable encryption on new IaaS VMs created from the Marketplace
You can enable disk encryption on new IaaS Windows VM from the Marketplace in Azure using a Resource Manager template. The template creates a new encrypted Windows VM using the Windows Server 2012 gallery image.

1. On the [Resource Manager template](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-create-new-vm-gallery-image), click **Deploy to Azure**.

2. Select the subscription, resource group, resource group location, parameters, legal terms, and agreement. Click **Purchase** to deploy a new IaaS VM where encryption is enabled.

3. After you deploy the template, verify the VM encryption status using your preferred method:
     - Verify with the Azure CLI by using the [az vm encryption show](/cli/azure/vm/encryption#az-vm-encryption-show) command. 

         ```azurecli-interactive 
         az vm encryption show --name "MySecureVM" --resource-group "MySecureRg"
         ```

     - Verify with Azure PowerShell by using the [Get-AzureRmVmDiskEncryptionStatus](/powershell/module/azurerm.compute/get-azurermvmdiskencryptionstatus) cmdlet. 

         ```azurepowershell-interactive
         Get-AzureRmVmDiskEncryptionStatus -ResourceGroupName 'MySecureRg' -VMName 'MySecureVM'
         ```

     -  Select the VM, then click on **Disks** under the **Settings** heading to verify encryption status in the portal. In the chart under **Encryption**, you'll see if it's enabled. 
           ![Azure portal - Disk Encryption Enabled](./media/azure-security-disk-encryption/disk-encryption-fig2.png)
The following table lists the Resource Manager template parameters for new VMs from the Marketplace scenario using Azure AD client ID:

| Parameter | Description | 
| --- | --- |
| adminUserName | Admin user name for the virtual machine. |
| adminPassword | Admin user password for the virtual machine. |
| newStorageAccountName | Name of the storage account to store OS and data VHDs. |
| vmSize | Size of the VM. Currently, only Standard A, D, and G series are supported. |
| virtualNetworkName | Name of the VNet that the VM NIC should belong to. |
| subnetName | Name of the subnet in the VNet that the VM NIC should belong to. |
| AADClientID | Client ID of the Azure AD application that has permissions to write secrets to your key vault. |
| AADClientSecret | Client secret of the Azure AD application that has permissions to write secrets to your key vault. |
| keyVaultURL | URL of the key vault that the BitLocker key should be uploaded to. You can get it by using the cmdlet `(Get-AzureRmKeyVault -VaultName,-ResourceGroupName).VaultURI` or the Azure CLI `az keyvault show --name "MySecureVault" --query properties.vaultUri` |
| keyEncryptionKeyURL | URL of the key encryption key that's used to encrypt the generated BitLocker key (optional). </br> </br>KeyEncryptionKeyURL is an optional parameter. You can bring your own KEK to further safeguard the data encryption key (Passphrase secret) in your key vault. |
| keyVaultResourceGroup | Resource group of the key vault. |
| vmName | Name of the VM that the encryption operation is to be performed on. |


## <a name="bkmk_RunningWinVM"></a> Enable encryption on existing or running IaaS Windows VMs
In this scenario, you can enable encryption by using a template, PowerShell cmdlets, or CLI commands. The following sections explain in greater detail how to enable Azure Disk Encryption. 

>[!IMPORTANT]
 >It is mandatory to snapshot and/or backup a managed disk based VM instance outside of, and prior to enabling Azure Disk Encryption. A snapshot of the managed disk can be taken from the portal, or [Azure Backup](../backup/backup-azure-vms-encryption.md) can be used. Backups ensure that a recovery option is possible in the case of any unexpected failure during encryption. Once a backup is made, the Set-AzureRmVMDiskEncryptionExtension cmdlet can be used to encrypt managed disks by specifying the -skipVmBackup parameter. The Set-AzureRmVMDiskEncryptionExtension command will fail against managed disk based VMs until a backup has been made and this parameter has been specified. 
>
>Encrypting or disabling encryption may cause the VM to reboot. 
>

### <a name="bkmk_RunningWinVMPSH"></a> Enable encryption on existing or running VMs with Azure PowerShell 
Use the [Set-AzureRmVMDiskEncryptionExtension](/powershell/module/azurerm.compute/set-azurermvmdiskencryptionextension) cmdlet to enable encryption on a running IaaS virtual machine in Azure. 
For information about enabling encryption with Azure Disk Encryption by using PowerShell cmdlets, see the blog posts [Explore Azure Disk Encryption with Azure PowerShell - Part 1](http://blogs.msdn.com/b/azuresecurity/archive/2015/11/17/explore-azure-disk-encryption-with-azure-powershell.aspx) and [Explore Azure Disk Encryption with Azure PowerShell - Part 2](http://blogs.msdn.com/b/azuresecurity/archive/2015/11/21/explore-azure-disk-encryption-with-azure-powershell-part-2.aspx).

-  **Encrypt a running VM using a client secret:** The script below initializes your variables and runs the Set-AzureRmVMDiskEncryptionExtension cmdlet. The resource group, VM, key vault, AAD app, and client secret should have already been created as prerequisites. Replace MySecureRg, MySecureVM, MySecureVault, My-AAD-client-ID, and My-AAD-client-secret with your values.
     ```azurepowershell-interactive
      $rgName = 'MySecureRg';
      $vmName = 'MySecureVM';
      $aadClientID = 'My-AAD-client-ID';
      $aadClientSecret = 'My-AAD-client-secret';
      $KeyVaultName = 'MySecureVault';
      $KeyVault = Get-AzureRmKeyVault -VaultName $KeyVaultName -ResourceGroupName $rgname;
      $diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
      $KeyVaultResourceId = $KeyVault.ResourceId;

      Set-AzureRmVMDiskEncryptionExtension -ResourceGroupName $rgname -VMName $vmName -AadClientID $aadClientID -AadClientSecret $aadClientSecret -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId;
    ```
- **Encrypt a running VM using KEK to wrap the client secret:** Azure Disk Encryption lets you specify an existing key in your key vault to wrap disk encryption secrets that were generated while enabling encryption. When a key encryption key is specified, Azure Disk Encryption uses that key to wrap the encryption secrets before writing to Key Vault. 

     ```azurepowershell-interactive
     $rgName = 'MySecureRg';
     $vmName = ‘MyExtraSecureVM’;
     $aadClientID = 'My-AAD-client-ID';
     $aadClientSecret = 'My-AAD-client-secret';
     $KeyVaultName = 'MySecureVault';
     $keyEncryptionKeyName = 'MyKeyEncryptionKey';
     $KeyVault = Get-AzureRmKeyVault -VaultName $KeyVaultName -ResourceGroupName $rgname;
     $diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
     $KeyVaultResourceId = $KeyVault.ResourceId;
     $keyEncryptionKeyUrl = (Get-AzureKeyVaultKey -VaultName $KeyVaultName -Name $keyEncryptionKeyName).Key.kid;

     Set-AzureRmVMDiskEncryptionExtension -ResourceGroupName $rgname -VMName $vmName -AadClientID $aadClientID -AadClientSecret $aadClientSecret -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -KeyEncryptionKeyUrl $keyEncryptionKeyUrl -KeyEncryptionKeyVaultId $KeyVaultResourceId;

     ```
     
   >[!NOTE]
   > The syntax for the value of disk-encryption-keyvault parameter is the full identifier string:
/subscriptions/[subscription-id-guid]/resourceGroups/[resource-group-name]/providers/Microsoft.KeyVault/vaults/[keyvault-name]</br> 
   > The syntax for the value of the key-encryption-key parameter is the full URI to the KEK as in:
https://[keyvault-name].vault.azure.net/keys/[kekname]/[kek-unique-id] 

- **Verify the disks are encrypted:** To check on the encryption status of a IaaS VM, use the [Get-AzureRmVmDiskEncryptionStatus](/powershell/module/azurerm.compute/get-azurermvmdiskencryptionstatus) cmdlet. 
     ```azurepowershell-interactive
     Get-AzureRmVmDiskEncryptionStatus -ResourceGroupName 'MySecureRg' -VMName 'MySecureVM'
     ```
    
- **Disable disk encryption:** To disable the encryption, use the [Disable-Azure​RmVMDisk​Encryption](/powershell/module/azurerm.compute/disable-azurermvmdiskencryption) cmdlet. 
     ```azurepowershell-interactive
     Disable-AzureRmVMDiskEncryption -ResourceGroupName 'MySecureRG' -VMName 'MySecureVM'
     ```

### <a name="bkmk_RunningWinVMCLI"></a>Enable encryption on existing or running VMs with  Azure CLI
Use the [az vm encryption enable](/cli/azure/vm/encryption#az-vm-encryption-enable) command to enable encryption on a running IaaS virtual machine in Azure.

-  **Encrypt a running VM using a client secret:**

     ```azurecli-interactive
     az vm encryption enable --resource-group "MySecureRg" --name "MySecureVM" --aad-client-id "<my spn created with CLI/my Azure AD ClientID>"  --aad-client-secret "My-AAD-client-secret" --disk-encryption-keyvault "MySecureVault" --volume-type [All|OS|Data]
     ```

- **Encrypt a running VM using KEK to wrap the client secret:**

     ```azurecli-interactive
     az vm encryption enable --resource-group "MySecureRg" --name "MySecureVM" --aad-client-id "<my spn created with CLI which is the Azure AD ClientID>"  --aad-client-secret "My-AAD-client-secret" --disk-encryption-keyvault  "MySecureVault" --key-encryption-key "MyKEK_URI" --key-encryption-keyvault "MySecureVaultContainingTheKEK" --volume-type [All|OS|Data]
     ```

     >[!NOTE]
     > The syntax for the value of disk-encryption-keyvault parameter is the full identifier string: 
/subscriptions/[subscription-id-guid]/resourceGroups/[resource-group-name]/providers/Microsoft.KeyVault/vaults/[keyvault-name] </br> 
     > The syntax for the value of the key-encryption-key parameter is the full URI to the KEK as in:
https://[keyvault-name].vault.azure.net/keys/[kekname]/[kek-unique-id] 

- **Verify the disks are encrypted:** To check on the encryption status of a IaaS VM, use the [az vm encryption show](/cli/azure/vm/encryption#az-vm-encryption-show) command. 

     ```azurecli-interactive
     az vm encryption show --name "MySecureVM" --resource-group "MySecureRg"
     ```

- **Disable encryption:** To disable encryption, use the [az vm encryption disable](/cli/azure/vm/encryption#az-vm-encryption-disable) command. 
     ```azurecli-interactive
     az vm encryption disable --name "MySecureVM" --resource-group "MySecureRg" --volume-type [ALL, DATA, OS]
     ```
 
 > [!NOTE]
 >It is mandatory to snapshot and/or backup a managed disk based VM instance outside of, and prior to enabling Azure Disk Encryption. A snapshot of the managed disk can be taken from the portal, or [Azure Backup](../backup/backup-azure-vms-encryption.md) can be used. Backups ensure that a recovery option is possible in the case of any unexpected failure during encryption. Once a backup is made, the Set-AzureRmVMDiskEncryptionExtension cmdlet can be used to encrypt managed disks by specifying the -skipVmBackup parameter. This command will fail against managed disk based VMs until a backup has been made and this parameter has been specified. 
>
>Encrypting or disabling encryption may cause the VM to reboot. 

### <a name="bkmk_RunningWinVMwRM"> </a>Using the Resource Manager template
You can enable disk encryption on existing or running IaaS Windows VMs in Azure by using the [Resource Manager template to encrypt a running Windows VM](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-running-windows-vm).


1. On the Azure quickstart template, click **Deploy to Azure**.

2. Select the subscription, resource group, resource group location, parameters, legal terms, and agreement. Click **Purchase** to enable encryption on the existing or running IaaS VM.

The following table lists the Resource Manager template parameters for existing or running VMs that use an Azure AD client ID:

| Parameter | Description |
| --- | --- |
| AADClientID | Client ID of the Azure AD application that has permissions to write secrets to the key vault. |
| AADClientSecret | Client secret of the Azure AD application that has permissions to write secrets to the key vault. |
| keyVaultName | Name of the key vault that the BitLocker key should be uploaded to. You can get it by using the cmdlet `(Get-AzureRmKeyVault -ResourceGroupName <MyResourceGroupName>). Vaultname` or the Azure CLI command `az keyvault list --resource-group "MySecureGroup" |Convertfrom-JSON`|
|  keyEncryptionKeyURL | URL of the key encryption key that's used to encrypt the generated BitLocker key. This parameter is optional if you select **nokek** in the UseExistingKek drop-down list. If you select **kek** in the UseExistingKek drop-down list, you must enter the _keyEncryptionKeyURL_ value. |
| volumeType | Type of volume that the encryption operation is performed on. Valid values are _OS_, _Data_, and _All_. |
| sequenceVersion | Sequence version of the BitLocker operation. Increment this version number every time a disk-encryption operation is performed on the same VM. |
| vmName | Name of the VM that the encryption operation is to be performed on. |


## <a name="bkmk_VHDpre"> </a>New IaaS VMs created from customer-encrypted VHD and encryption keys
In this scenario, you can enable encrypting by using the Resource Manager template, PowerShell cmdlets, or CLI commands. The following sections explain in greater detail the Resource Manager template and CLI commands. 

Use the instructions in the appendix for preparing pre-encrypted images that can be used in Azure. After the image is created, you can use the steps in the next section to create an encrypted Azure VM.

* [Prepare a pre-encrypted Windows VHD](azure-security-disk-encryption-appendix.md#bkmk_preWin)
* [Prepare a pre-encrypted Linux VHD](azure-security-disk-encryption-appendix.md#bkmk_preLinux)

>[!IMPORTANT]
 >It is mandatory to snapshot and/or backup a managed disk based VM instance outside of, and prior to enabling Azure Disk Encryption. A snapshot of the managed disk can be taken from the portal, or [Azure Backup](../backup/backup-azure-vms-encryption.md) can be used. Backups ensure that a recovery option is possible in the case of any unexpected failure during encryption. Once a backup is made, the Set-AzureRmVMDiskEncryptionExtension cmdlet can be used to encrypt managed disks by specifying the -skipVmBackup parameter. The Set-AzureRmVMDiskEncryptionExtension command will fail against managed disk based VMs until a backup has been made and this parameter has been specified. 
>
>Encrypting or disabling encryption may cause the VM to reboot. 



### <a name="bkmk_VHDprePSH"> </a> Encrypt VMs with pre-encrypted VHDs with Azure PowerShell
You can enable disk encryption on your encrypted VHD by using the PowerShell cmdlet [Set-AzureRmVMOSDisk](/powershell/module/azurerm.compute/set-azurermvmosdisk#examples). The example below gives you some common parameters. 

```powershell
$VirtualMachine = New-AzureRmVMConfig -VMName "MySecureVM" -VMSize "Standard_A1"
$VirtualMachine = Set-AzureRmVMOSDisk -VM $VirtualMachine -Name "SecureOSDisk" -VhdUri "os.vhd" Caching ReadWrite -Windows -CreateOption "Attach" -DiskEncryptionKeyUrl "https://mytestvault.vault.azure.net/secrets/Test1/514ceb769c984379a7e0230bddaaaaaa" -DiskEncryptionKeyVaultId "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myresourcegroup/providers/Microsoft.KeyVault/vaults/mytestvault"
New-AzureRmVM -VM $VirtualMachine -ResouceGroupName "MySecureRG"
```


### <a name="bkmk_VHDpreRM"> </a>Encrypt IaaS VMs with pre-encrypted VHDs with a Resource Manager template 
You can enable disk encryption on your encrypted VHD by using the [Resource Manager template](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-create-pre-encrypted-vm).

1. On the Azure quickstart template, click **Deploy to Azure**.

2. Select the subscription, resource group, resource group location, parameters, legal terms, and agreement. Click **Create** to enable encryption on the new IaaS VM.

The following table lists the Resource Manager template parameters for your encrypted VHD:

| Parameter | Description |
| --- | --- |
| newStorageAccountName | Name of the storage account to store the encrypted OS VHD. This storage account should already have been created in the same resource group and same location as the VM. |
| osVhdUri | URI of the OS VHD from the storage account. |
| osType | OS product type (Windows/Linux). |
| virtualNetworkName | Name of the VNet that the VM NIC should belong to. The name should already have been created in the same resource group and same location as the VM. |
| subnetName | Name of the subnet on the VNet that the VM NIC should belong to. |
| vmSize | Size of the VM. Currently, only Standard A, D, and G series are supported. |
| keyVaultResourceID | The ResourceID that identifies the key vault resource in Azure Resource Manager. You can get it by using the PowerShell cmdlet `(Get-AzureRmKeyVault -VaultName &lt;MyKeyVaultName&gt; -ResourceGroupName &lt;MyResourceGroupName&gt;).ResourceId` or using the Azure CLI command `az keyvault show --name "MySecureVault" --query id`|
| keyVaultSecretUrl | URL of the disk-encryption key that's set up in the key vault. |
| keyVaultKekUrl | URL of the key encryption key for encrypting the generated disk-encryption key. |
| vmName | Name of the IaaS VM. |

## Enable encryption on a newly added data disk
You can [add a new disk to a Windows VM using PowerShell](../virtual-machines/windows/attach-disk-ps.md), or [through the Azure portal](../virtual-machines/windows/attach-managed-disk-portal.md). 

### Enable encryption on a newly added disk with Azure PowerShell
 When using Powershell to encrypt a new disk for Windows VMs, a new sequence version should be specified. The sequence version has to be unique. The script below generates a GUID for the sequence version. In some cases, a newly added data disk might be encrypted automatically by the Azure Disk Encryption extension. If this occurs, we recommend running the Set-AzureRmVmDiskEncryptionExtension cmdlet again with new sequence version.
 

-  **Encrypt a running VM using a client secret:** The script below initializes your variables and runs the Set-AzureRmVMDiskEncryptionExtension cmdlet. The resource group, VM, key vault, AAD app, and client secret should have already been created as prerequisites. Replace MySecureRg, MySecureVM, MySecureVault, My-AAD-client-ID, and My-AAD-client-secret with your values. This example uses "All" for the -VolumeType parameter, which includes both OS and Data volumes. If you only want to encrypt the OS volume, use "OS" for the -VolumeType parameter. 

     ```azurepowershell-interactive
      $sequenceVersion = [Guid]::NewGuid();
      $rgName = 'MySecureRg';
      $vmName = 'MySecureVM';
      $aadClientID = 'My-AAD-client-ID';
      $aadClientSecret = 'My-AAD-client-secret';
      $KeyVaultName = 'MySecureVault';
      $KeyVault = Get-AzureRmKeyVault -VaultName $KeyVaultName -ResourceGroupName $rgname;
      $diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
      $KeyVaultResourceId = $KeyVault.ResourceId;

      Set-AzureRmVMDiskEncryptionExtension -ResourceGroupName $rgname -VMName $vmName -AadClientID $aadClientID -AadClientSecret $aadClientSecret -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -VolumeType 'all' –SequenceVersion $sequenceVersion;
    ```
- **Encrypt a running VM using KEK to wrap the client secret:** Azure Disk Encryption lets you specify an existing key in your key vault to wrap disk encryption secrets that were generated while enabling encryption. When a key encryption key is specified, Azure Disk Encryption uses that key to wrap the encryption secrets before writing to Key Vault. This example uses "All" for the -VolumeType parameter, which includes both OS and Data volumes. If you only want to encrypt the OS volume, use "OS" for the -VolumeType parameter. 

     ```azurepowershell-interactive
     $sequenceVersion = [Guid]::NewGuid();
     $rgName = 'MySecureRg';
     $vmName = 'MyExtraSecureVM';
     $aadClientID = 'My-AAD-client-ID';
     $aadClientSecret = 'My-AAD-client-secret';
     $KeyVaultName = 'MySecureVault';
     $keyEncryptionKeyName = 'MyKeyEncryptionKey';
     $KeyVault = Get-AzureRmKeyVault -VaultName $KeyVaultName -ResourceGroupName $rgname;
     $diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
     $KeyVaultResourceId = $KeyVault.ResourceId;
     $keyEncryptionKeyUrl = (Get-AzureKeyVaultKey -VaultName $KeyVaultName -Name $keyEncryptionKeyName).Key.kid;

     Set-AzureRmVMDiskEncryptionExtension -ResourceGroupName $rgname -VMName $vmName -AadClientID $aadClientID -AadClientSecret $aadClientSecret -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -KeyEncryptionKeyUrl $keyEncryptionKeyUrl -KeyEncryptionKeyVaultId $KeyVaultResourceId -VolumeType 'all' –SequenceVersion $sequenceVersion;

     ```

    >[!NOTE]
    > The syntax for the value of disk-encryption-keyvault parameter is the full identifier string: 
/subscriptions/[subscription-id-guid]/resourceGroups/[resource-group-name]/providers/Microsoft.KeyVault/vaults/[keyvault-name]</br> 
   > The syntax for the value of the key-encryption-key parameter is the full URI to the KEK as in:
https://[keyvault-name].vault.azure.net/keys/[kekname]/[kek-unique-id] 

### Enable encryption on a newly added disk with Azure CLI
  The Azure CLI command will automatically provide a new sequence version for you when you run the command to enable encryption. Acceptable values for the volume-yype parameter are All, OS, and Data. You may need to change the volume-type parameter to OS or Data if you're only encrypting one type of disk for the VM. The examples use "All" for the volume-type parameter. 

-  **Encrypt a running VM using a client secret:**

     ```azurecli-interactive
     az vm encryption enable --resource-group "MySecureRg" --name "MySecureVM" --aad-client-id "<my spn created with CLI/my Azure AD ClientID>"  --aad-client-secret "My-AAD-client-secret" --disk-encryption-keyvault "MySecureVault" --volume-type "All"
     ```

- **Encrypt a running VM using KEK to wrap the client secret:**

     ```azurecli-interactive
     az vm encryption enable --resource-group "MySecureRg" --name "MySecureVM" --aad-client-id "<my spn created with CLI which is the Azure AD ClientID>"  --aad-client-secret "My-AAD-client-secret" --disk-encryption-keyvault  "MySecureVault" --key-encryption-key "MyKEK_URI" --key-encryption-keyvault "MySecureVaultContainingTheKEK" --volume-type "all"
     ```


## Enable encryption using Azure AD client certificate-based authentication.
You can use client certificate authentication with or without KEK. The scripts require that [Azure Disk Encryption prerequisites](azure-security-disk-encryption-prerequisites-aad.md) are complete. Before using the PowerShell scripts, you should already have the certificate uploaded to the key vault and deployed to the VM. If you're using KEK too, the KEK should already exist. For more information, see the  [Certificate-based authentication for Azure AD](azure-security-disk-encryption-prerequisites-aad.md#bkmk_Cert) section of the prerequisites article.


### Enable encryption using certificate-based authentication with Azure PowerShell

```powershell
## Fill in 'MySecureRg', 'My-AAD-client-ID', 'MySecureVault, and ‘MySecureVM’.

$rgName = 'MySecureRg';
$AADClientID ='My-AAD-client-ID';
$KeyVaultName = 'MySecureVault';
$VMName = ‘MySecureVM’;
$diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
$KeyVaultResourceId = $KeyVault.ResourceId;

# Fill in the certificate path and the password so the thumbprint can be set as a variable. 

$certPath = '$CertPath = "C:\certificates\mycert.pfx';
$CertPassword ='Password'
$Cert = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2($CertPath, $CertPassword)
$aadClientCertThumbprint = $cert.Thumbprint;

# Enable disk encryption using the client certificate thumbprint

Set-AzureRmVMDiskEncryptionExtension -ResourceGroupName $rgname -VMName $VMName -AadClientID $AADClientID -AadClientCertThumbprint $AADClientCertThumbprint -DiskEncryptionKeyVaultUrl $DiskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId
```
  
### Enable encryption using certificate-based authentication and a KEK with Azure PowerShell

```powershell
# Fill in 'MySecureRg', 'My-AAD-client-ID', 'MySecureVault,, 'MySecureVM’, and "KEKName.

$rgName = 'MySecureRg';
$AADClientID ='My-AAD-client-ID';
$KeyVaultName = 'MySecureVault';
$VMName = 'MySecureVM';
$keyEncryptionKeyName ='KEKName';
$diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
$KeyVaultResourceId = $KeyVault.ResourceId;
$keyEncryptionKeyUrl = (Get-AzureKeyVaultKey -VaultName $KeyVaultName -Name $keyEncryptionKeyName).Key.kid;

## Fill in the certificate path and the password so the thumbprint can be read and set as a variable. 

$certPath = '$CertPath = "C:\certificates\mycert.pfx';
$CertPassword ='Password'
$Cert = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2($CertPath, $CertPassword)
$aadClientCertThumbprint = $cert.Thumbprint;

# Enable disk encryption using the client certificate thumbprint and a KEK

Set-AzureRmVMDiskEncryptionExtension -ResourceGroupName $rgname -VMName $VMName -AadClientID $AADClientID -AadClientCertThumbprint $AADClientCertThumbprint -DiskEncryptionKeyVaultUrl $DiskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -KeyEncryptionKeyUrl $keyEncryptionKeyUrl -KeyEncryptionKeyVaultId $KeyVaultResourceId
```

## Disable encryption
You can disable encryption using Azure PowerShell, the Azure CLI, or with a Resource Manager template. 

- **Disable disk encryption with Azure PowerShell:** To disable the encryption, use the [Disable-Azure​RmVMDisk​Encryption](/powershell/module/azurerm.compute/disable-azurermvmdiskencryption) cmdlet. 
     ```azurepowershell-interactive
     Disable-AzureRmVMDiskEncryption -ResourceGroupName 'MySecureRG' -VMName 'MySecureVM'
     ```

- **Disable encryption with the Azure CLI:** To disable encryption, use the [az vm encryption disable](/cli/azure/vm/encryption#az-vm-encryption-disable) command. 
     ```azurecli-interactive
     az vm encryption disable --name "MySecureVM" --resource-group "MySecureRg" --volume-type [ALL, DATA, OS]
     ```
- **Disable encryption with a Resource Manager Template:** 

    1. Click **Deploy to Azure** from the [Disable disk encryption on running Windows VM](https://github.com/Azure/azure-quickstart-templates/tree/master/201-decrypt-running-windows-vm)  template.
    2. Select the subscription, resource group, location, VM, legal terms, and agreement.
    3.  Click **Purchase** to disable disk encryption on a running Windows VM. 

## Next steps

> [!div class="nextstepaction"]
> [Enable Azure Disk Encryption for Linux](azure-security-disk-encryption-linux-aad.md)
