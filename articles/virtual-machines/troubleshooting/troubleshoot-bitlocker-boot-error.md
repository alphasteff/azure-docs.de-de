---
title: Problembehandlung für BitLocker-Startfehler auf einer Azure-VM | Microsoft-Dokumentation
description: Erfahren Sie, wie Sie BitLocker-Startfehler auf einer Azure-VM beheben.
services: virtual-machines-windows
documentationCenter: ''
author: genlin
manager: cshepard
editor: v-jesits
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: troubleshooting
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 03/25/2019
ms.author: genli
ms.openlocfilehash: 27a675982711f8d8f0b36ea0cc2600de45e97a6e
ms.sourcegitcommit: e72073911f7635cdae6b75066b0a88ce00b9053b
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 07/19/2019
ms.locfileid: "68348458"
---
# <a name="bitlocker-boot-errors-on-an-azure-vm"></a>BitLocker-Startfehler auf einer Azure-VM

 Dieser Artikel beschreibt BitLocker-Fehler, die beim Starten eines virtuellen Windows-Computers (VM) in Microsoft Azure auftreten können.

[!INCLUDE [updated-for-az.md](../../../includes/updated-for-az.md)]

## <a name="symptom"></a>Symptom

 Eine Windows-VM wird nicht gestartet. Beim Überprüfen der Screenshots im Fenster [Startdiagnose](../windows/boot-diagnostics.md) wird eine der folgenden Fehlermeldungen angezeigt:

- Stecken Sie das USB-Laufwerk mit dem BitLocker-Schlüssel ein.

- Sie sind ausgesperrt! Geben Sie den Wiederherstellungsschlüssel ein, um den Computer wieder verwenden zu können (Tastaturlayout: USA). Es wurden zu häufig die falschen Anmeldeinformationen eingegeben, daher wurde Ihr PC zum Schutz Ihrer Privatsphäre gesperrt. Um den Wiederherstellungsschlüssel abzurufen, wechseln Sie von einem anderen PC oder mobilen Gerät aus zu https://windows.microsoft.com/recoverykeyfaq. Die Schlüssel-ID lautet XXXXXXX, falls Sie sie benötigen. Sie können Ihren PC auch zurücksetzen.

- Geben Sie das Kennwort ein, um dieses Laufwerk zu entsperren: [ ]. Drücken Sie die EINFG-TASTE, um das Kennwort während der Eingabe anzuzeigen.
- Geben Sie den Wiederherstellungsschlüssel ein. Laden Sie den Wiederherstellungsschlüssel von einem USB-Gerät.

## <a name="cause"></a>Ursache

Dieses Problem kann auftreten, wenn der virtuelle Computer die BitLocker-Wiederherstellungsschlüsseldatei (BEK) zum Entschlüsseln des verschlüsselten Datenträgers nicht finden kann.

## <a name="solution"></a>Lösung

Um dieses Problem zu beheben, beenden Sie die VM, heben ihre Zuordnung auf und starten sie dann neu. Durch diesen Vorgang ruft der virtuelle Computer die BEK-Datei aus Azure Key Vault ab und legt sie dann auf dem verschlüsselten Datenträger ab. 

Wenn das Problem mit dieser Methode nicht behoben wird, gehen Sie folgendermaßen vor, um die BEK-Datei manuell wiederherzustellen:

1. Erstellen Sie eine Momentaufnahme des Systemdatenträgers des betroffenen virtuellen Computers als Sicherung. Weitere Informationen finden Sie unter [Erstellen einer Momentaufnahme eines Datenträgers](../windows/snapshot-copy-managed-disk.md).
2. [Fügen Sie den Systemdatenträger an einen virtuellen Wiederherstellungscomputer an.](troubleshoot-recovery-disks-portal-windows.md) Zum Ausführen des Befehls [manage-bde](https://docs.microsoft.com/windows-server/administration/windows-commands/manage-bde) in Schritt 7 muss das Feature **BitLocker Drive Encryption** auf der Wiederherstellungs-VM aktiviert werden.

    Wenn Sie einen verwalteten Datenträger anfügen, wird möglicherweise eine Fehlermeldung darüber angezeigt, dass Verschlüsselungseinstellungen enthalten sind und er daher nicht als Datenträger für Daten verwendet werden kann. Führen Sie in diesem Fall das folgende Skript aus, um das Anfügen des Datenträgers erneut zu versuchen:

    ```Powershell
    $rgName = "myResourceGroup"
    $osDiskName = "ProblemOsDisk"

    New-AzDiskUpdateConfig -EncryptionSettingsEnabled $false |Update-AzDisk -diskName $osDiskName -ResourceGroupName $rgName

    $recoveryVMName = "myRecoveryVM" 
    $recoveryVMRG = "RecoveryVMRG" 
    $OSDisk = Get-AzDisk -ResourceGroupName $rgName -DiskName $osDiskName;

    $vm = get-AzVM -ResourceGroupName $recoveryVMRG -Name $recoveryVMName 

    Add-AzVMDataDisk -VM $vm -Name $osDiskName -ManagedDiskId $osDisk.Id -Caching None -Lun 3 -CreateOption Attach 

    Update-AzVM -VM $vm -ResourceGroupName $recoveryVMRG
    ```
     Sie können einen verwalteten Datenträger nicht an einen virtuellen Computer anfügen, der aus einem Blobimage wiederhergestellt wurde.

3. Nachdem der Datenträger angefügt wurde, stellen Sie eine Remotedesktopverbindung mit der Wiederherstellungs-VM her, damit Sie einige Azure PowerShell-Skripts ausführen können. Achten Sie darauf, dass die [aktuelle Version von Azure PowerShell](https://docs.microsoft.com/powershell/azure/overview) auf der Wiederherstellungs-VM installiert ist.

4. Öffnen Sie eine Azure PowerShell Sitzung mit erhöhten Rechten („Als Administrator ausführen“). Führen Sie die folgenden Befehle aus, um sich beim Azure-Abonnement anzumelden.

    ```Powershell
    Add-AzAccount -SubscriptionID [SubscriptionID]
    ```

5. Führen Sie das folgende Skript aus, um den Namen der BEK-Datei zu überprüfen:

    ```powershell
    $vmName = "myVM"
    $vault = "myKeyVault"
    Get-AzKeyVaultSecret -VaultName $vault | where {($_.Tags.MachineName -eq $vmName) -and ($_.ContentType -match 'BEK')} `
            | Sort-Object -Property Created `
            | ft  Created, `
                @{Label="Content Type";Expression={$_.ContentType}}, `
                @{Label ="Volume"; Expression = {$_.Tags.VolumeLetter}}, `
                @{Label ="DiskEncryptionKeyFileName"; Expression = {$_.Tags.DiskEncryptionKeyFileName}}
    ```

    Nachfolgend sehen Sie ein Beispiel für die Ausgabe: Suchen Sie den Namen der BEK-Datei für den angefügten Datenträger. In diesem Fall wird davon ausgegangen, dass der Laufwerkbuchstabe des angefügten Datenträgers F und der Name der BEK-Datei „EF7B2F5A-50C6-4637-9F13-7F599C12F85C.BEK“ lautet.

    ```
    Created             Content Type Volume DiskEncryptionKeyFileName               
    -------             ------------ ------ -------------------------               
    4/5/2018 7:14:59 PM Wrapped BEK  C:\    B4B3E070-836C-4AF5-AC5B-66F6FDE6A971.BEK
    4/7/2018 7:21:16 PM Wrapped BEK  F:\    EF7B2F5A-50C6-4637-9F13-7F599C12F85C.BEK
    4/7/2018 7:26:23 PM Wrapped BEK  G:\    70148178-6FAE-41EC-A05B-3431E6252539.BEK
    4/7/2018 7:26:26 PM Wrapped BEK  H:\    5745719F-4886-4940-9B51-C98AFABE5305.BEK
    ```

    Wenn zwei doppelte Volumes angezeigt werden, ist das Volume mit dem neueren Zeitstempel die aktuelle BEK-Datei, die von der Wiederherstellungs-VM verwendet wird.

    Wenn **Inhaltstyp** den Wert **Wrapped BEK** (Umschlossene BEK-Datei) aufweist, wechseln Sie zu den [Schlüsselverschlüsselungsschlüssel-Szenarien (KEK)](#key-encryption-key-scenario).

    Nun, da Ihnen der Name der BEK-Datei für das Laufwerk bekannt ist, müssen Sie die Datei „secret-file-name.BEK“ erstellen, um das Laufwerk zu entsperren.

6.  Laden Sie die BEK-Datei auf den Wiederherstellungsdatenträger herunter. Im folgenden Beispiel wird die BEK-Datei im Ordner „C:\BEK“ gespeichert. Stellen Sie sicher, dass der Pfad `C:\BEK\` vorhanden ist, bevor Sie die Skripts ausführen.

    ```powershell
    $vault = "myKeyVault"
    $bek = " EF7B2F5A-50C6-4637-9F13-7F599C12F85C"
    $keyVaultSecret = Get-AzKeyVaultSecret -VaultName $vault -Name $bek
    $bekSecretBase64 = $keyVaultSecret.SecretValueText
    $bekFileBytes = [Convert]::FromBase64String($bekSecretbase64)
    $path = "C:\BEK\DiskEncryptionKeyFileName.BEK"
    [System.IO.File]::WriteAllBytes($path,$bekFileBytes)
    ```

7.  Führen Sie den folgenden Befehl aus, um den angefügten Datenträger mithilfe der BEK-Datei zu entsperren.

    ```powershell
    manage-bde -unlock F: -RecoveryKey "C:\BEK\EF7B2F5A-50C6-4637-9F13-7F599C12F85C.BEK
    ```
    In diesem Beispiel ist der angefügte Betriebssystemdatenträger das Laufwerk F. Achten Sie darauf, den richtigen Laufwerkbuchstaben zu verwenden. 

    - Wenn der Datenträger mit dem BEK-Schlüssel entsperrt wurde, kann das BitLocker-Problem als gelöst betrachtet werden. 

    - Wenn der Datenträger mit dem BEK-Schlüssel nicht entsperrt werden kann, können Sie den Schutz aufheben, um BitLocker vorübergehend zu deaktivieren, indem Sie den folgenden Befehl ausführen.
    
        ```powershell
        manage-bde -protectors -disable F: -rc 0
        ```      
    - Wenn Sie vorhaben, den virtuellen Computer mit dem Systemdatenträger neu zu erstellen, müssen Sie das Laufwerk vollständig entschlüsseln. Führen Sie zu diesem Zweck den folgenden Befehl aus:

        ```powershell
        manage-bde -off F:
        ```
8.  Trennen Sie den Datenträger von der Wiederherstellungs-VM, und fügen Sie den Datenträger dann erneut als Systemdatenträger an den betroffenen virtuellen Computer an. Weitere Informationen finden Sie unter [Beheben von Problemen mit einer Windows-VM durch Anfügen des Betriebssystemdatenträgers an eine Wiederherstellungs-VM](troubleshoot-recovery-disks-windows.md).

### <a name="key-encryption-key-scenario"></a>Schlüsselverschlüsselungsschlüssel-Szenario

Gehen Sie bei einem Szenario mit Schlüsselverschlüsselungsschlüssel folgendermaßen vor:

1. Stellen Sie sicher, dass das angemeldete Benutzerkonto die „nicht umschlossene“ Berechtigung in den Key Vault-Zugriffsrichtlinien unter **USER|Key permissions|Cryptographic Operations|Unwrap Key** (BENUTZER|Schlüsselberechtigungen|Kryptografische Vorgänge|Umschließen des Schlüssels aufheben) erfordert.
2. Speichern Sie die folgenden Skripts als PS1-Datei:

    ```powershell
    #Set the Parameters for the script
    param (
            [Parameter(Mandatory=$true)]
            [string] 
            $keyVaultName,
            [Parameter(Mandatory=$true)]
            [string] 
            $kekName,
            [Parameter(Mandatory=$true)]
            [string]
            $secretName,
            [Parameter(Mandatory=$true)]
            [string]
            $bekFilePath,
            [Parameter(Mandatory=$true)]
            [string] 
            $adTenant
            )
    # Load ADAL Assemblies. The following script assumes that the Azure PowerShell version you installed is 1.0.0. 
    $adal = "${env:ProgramFiles}\WindowsPowerShell\Modules\Az.Accounts\1.0.0\PreloadAssemblies\Microsoft.IdentityModel.Clients.ActiveDirectory.dll"
    $adalforms = "${env:ProgramFiles}\WindowsPowerShell\Modules\Az.Accounts\1.0.0\PreloadAssemblies\Microsoft.IdentityModel.Clients.ActiveDirectory.Platform.dll"
    [System.Reflection.Assembly]::LoadFrom($adal)
    [System.Reflection.Assembly]::LoadFrom($adalforms)

    # Set well-known client ID for AzurePowerShell
    $clientId = "1950a258-227b-4e31-a9cf-717495945fc2" 
    # Set redirect URI for Azure PowerShell
    $redirectUri = "urn:ietf:wg:oauth:2.0:oob"
    # Set Resource URI to Azure Service Management API
    $resourceAppIdURI = "https://vault.azure.net"
    # Set Authority to Azure AD Tenant
    $authority = "https://login.windows.net/$adtenant"
    # Create Authentication Context tied to Azure AD Tenant
    $authContext = New-Object "Microsoft.IdentityModel.Clients.ActiveDirectory.AuthenticationContext" -ArgumentList $authority
    # Acquire token
    $authResult = $authContext.AcquireTokenAsync($resourceAppIdURI, $clientId, $redirectUri, $platformParameters).result
    # Generate auth header 
    $authHeader = $authResult.CreateAuthorizationHeader()
    # Set HTTP request headers to include Authorization header
    $headers = @{'x-ms-version'='2014-08-01';"Authorization" = $authHeader}

    ########################################################################################################################
    # 1. Retrieve wrapped BEK
    # 2. Make KeyVault REST API call to unwrap the BEK
    # 3. Convert the Base64Url string returned by KeyVault unwrap to Base64 string 
    # 4. Convert Base64 string to bytes and write to the BEK file
    ########################################################################################################################

    #Get wrapped BEK and place it in JSON object to send to KeyVault REST API
    $keyVaultSecret = Get-AzureKeyVaultSecret -VaultName $keyVaultName -Name $secretName
    $wrappedBekSecretBase64 = $keyVaultSecret.SecretValueText
    $jsonObject = @"
    {
    "alg": "RSA-OAEP",
    "value" : "$wrappedBekSecretBase64"
    }
    "@

    #Get KEK Url
    $kekUrl = (Get-AzureKeyVaultKey -VaultName $keyVaultName -Name $kekName).Key.Kid;
    $unwrapKeyRequestUrl = $kekUrl+ "/unwrapkey?api-version=2015-06-01";

    #Call KeyVault REST API to Unwrap 
    $result = Invoke-RestMethod -Method POST -Uri $unwrapKeyRequestUrl -Headers $headers -Body $jsonObject -ContentType "application/json" -Debug

    #Convert Base64Url string returned by KeyVault unwrap to Base64 string
    $base64UrlBek = $result.value;
    $base64Bek = $base64UrlBek.Replace('-', '+');
    $base64Bek = $base64Bek.Replace('_', '/');
    if($base64Bek.Length %4 -eq 2)
    {
        $base64Bek+= '==';
    }
    elseif($base64Bek.Length %4 -eq 3)
    {
        $base64Bek+= '=';
    }

    #Convert base64 string to bytes and write to BEK file
    $bekFileBytes = [System.Convert]::FromBase64String($base64Bek);
    [System.IO.File]::WriteAllBytes($bekFilePath,$bekFileBytes)
    ```
3. Legen Sie die Parameter fest. Das Skript verarbeitet das KEK-Geheimnis, um den BEK-Schlüssel zu erstellen und diesen dann in einem lokalen Ordner auf der Wiederherstellungs-VM zu speichern.

4. Beim Start des Skripts wird die folgende Ausgabe angezeigt:

        GAC    Version        Location                                                                              
        ---    -------        --------                                                                              
        False  v4.0.30319     C:\Program Files\WindowsPowerShell\Modules\Az.Accounts\...
        False  v4.0.30319     C:\Program Files\WindowsPowerShell\Modules\Az.Accounts\...

    Sobald das Skript abgeschlossen ist, wird die folgende Ausgabe angezeigt:

        VERBOSE: POST https://myvault.vault.azure.net/keys/rondomkey/<KEY-ID>/unwrapkey?api-
        version=2015-06-01 with -1-byte payload
        VERBOSE: received 360-byte response of content type application/json; charset=utf-8


5. Um den angefügten Datenträger mithilfe der BEK-Datei zu entsperren, führen Sie den folgenden Befehl aus:

    ```powershell
    manage-bde -unlock F: -RecoveryKey "C:\BEK\EF7B2F5A-50C6-4637-9F13-7F599C12F85C.BEK
    ```
    In diesem Beispiel ist der angefügte Betriebssystemdatenträger das Laufwerk F. Achten Sie darauf, den richtigen Laufwerkbuchstaben zu verwenden. 

    - Wenn der Datenträger mit dem BEK-Schlüssel entsperrt wurde, kann das BitLocker-Problem als gelöst betrachtet werden. 

    - Wenn der Datenträger mit dem BEK-Schlüssel nicht entsperrt werden kann, können Sie den Schutz aufheben, um BitLocker vorübergehend zu deaktivieren, indem Sie den folgenden Befehl ausführen.
    
        ```powershell
        manage-bde -protectors -disable F: -rc 0
        ```      
    - Wenn Sie vorhaben, den virtuellen Computer mit dem Systemdatenträger neu zu erstellen, müssen Sie das Laufwerk vollständig entschlüsseln. Führen Sie zu diesem Zweck den folgenden Befehl aus:

        ```powershell
        manage-bde -off F:
        ```

6. Trennen Sie den Datenträger von der Wiederherstellungs-VM, und fügen Sie den Datenträger dann erneut als Systemdatenträger an den betroffenen virtuellen Computer an. Weitere Informationen finden Sie unter [Beheben von Problemen mit einer Windows-VM durch Anfügen des Betriebssystemdatenträgers an eine Wiederherstellungs-VM](troubleshoot-recovery-disks-windows.md).
