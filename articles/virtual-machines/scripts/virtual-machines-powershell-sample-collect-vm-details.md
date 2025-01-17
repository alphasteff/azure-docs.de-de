---
title: Sammeln von Informationen zu allen virtuellen Computern in einem Abonnement mit PowerShell | Microsoft-Dokumentation
description: Sammeln von Informationen zu allen virtuellen Computern in einem Abonnement mit PowerShell
services: virtual-machines-windows
documentationcenter: virtual-machines
author: v-miegge
manager: ???
editor: v-miegge
tags: azure-service-management
ms.assetid: ''
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 07/01/2019
ms.author: v-miegge
ms.custom: mvc
ms.openlocfilehash: b4828b34a089a59e630aaaf7652e8623b3e2c7b8
ms.sourcegitcommit: cf438e4b4e351b64fd0320bf17cc02489e61406a
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 07/08/2019
ms.locfileid: "67659488"
---
# <a name="collect-details-about-all-vms-in-a-subscription-with-powershell"></a>Sammeln von Informationen zu allen virtuellen Computern in einem Abonnement mit PowerShell

Dieses Skript erstellt eine CSV-Datei, die den VM-Namen, den Namen der Ressourcengruppe, die Region, das virtuelle Netzwerk, das Subnetz, die private IP-Adresse, den Betriebssystemtyp und die öffentliche IP-Adresse der VMs im bereitgestellten Abonnement enthält.

Wenn Sie kein [Azure-Abonnement](https://docs.microsoft.com/azure/guides/developer/azure-developer-guide#understanding-accounts-subscriptions-and-billing) besitzen, erstellen Sie ein [kostenloses Konto](https://azure.microsoft.com/free), bevor Sie beginnen.

## <a name="launch-azure-cloud-shell"></a>Starten von Azure Cloud Shell

Azure Cloud Shell ist eine kostenlose interaktive Shell, mit der Sie die Schritte in diesem Artikel ausführen können. Sie verfügt über allgemeine vorinstallierte Tools und ist für die Verwendung mit Ihrem Konto konfiguriert. 

Wählen Sie zum Öffnen von Cloud Shell oben rechts in einem Codeblock einfach die Option **Ausprobieren**. Sie können Cloud Shell auch auf einer separaten Browserregisterkarte starten, indem Sie zu [https://shell.azure.com/powershell](https://shell.azure.com/powershell) navigieren. Wählen Sie **Kopieren**, um die Blöcke mit dem Code zu kopieren. Fügen Sie ihn anschließend in Cloud Shell ein, und drücken Sie die EINGABETASTE, um ihn auszuführen.

## <a name="sample-script"></a>Beispielskript

```azurepowershell-interactive
#Provide the subscription Id where the VMs reside
$subscriptionId = "ea7ded4e-153a-4e65-ad70-25bf9f7b91bc"

#Provide the name of the csv file to be exported
$reportName = "myReport.csv"

Select-AzSubscription $subscriptionId
$report = @()
$vms = Get-AzVM
$publicIps = Get-AzPublicIpAddress 
$nics = Get-AzNetworkInterface | ?{ $_.VirtualMachine -NE $null} 
foreach ($nic in $nics) { 
    $info = "" | Select VmName, ResourceGroupName, Region, VirturalNetwork, Subnet, PrivateIpAddress, OsType, PublicIPAddress 
    $vm = $vms | ? -Property Id -eq $nic.VirtualMachine.id 
    foreach($publicIp in $publicIps) { 
        if($nic.IpConfigurations.id -eq $publicIp.ipconfiguration.Id) {
            $info.PublicIPAddress = $publicIp.ipaddress
            } 
        } 
        $info.OsType = $vm.StorageProfile.OsDisk.OsType 
        $info.VMName = $vm.Name 
        $info.ResourceGroupName = $vm.ResourceGroupName 
        $info.Region = $vm.Location 
        $info.VirturalNetwork = $nic.IpConfigurations.subnet.Id.Split("/")[-3] 
        $info.Subnet = $nic.IpConfigurations.subnet.Id.Split("/")[-1] 
        $info.PrivateIpAddress = $nic.IpConfigurations.PrivateIpAddress 
        $report+=$info 
    } 
$report | ft VmName, ResourceGroupName, Region, VirturalNetwork, Subnet, PrivateIpAddress, OsType, PublicIPAddress 
$report | Export-CSV "$home/$reportName"
```

## <a name="script-explanation"></a>Erläuterung des Skripts
Dieses Skript verwendet folgende Befehle, um einen CSV-Export der Informationen über VMs in einem Abonnement zu erstellen. Jeder Befehl in der Tabelle ist mit der zugehörigen Dokumentation verknüpft.

|Get-Help|Notizen|
|-|-|
|[Select-AzSubscription](https://docs.microsoft.com/powershell/module/Az.Accounts/Set-AzContext)|Legt den Mandanten, das Abonnement und die Umgebung für Cmdlets fest, die in der aktuellen Sitzung verwendet werden sollen.|
|[Get-AzVM](https://docs.microsoft.com/powershell/module/Az.Compute/Get-AzVM)|Ruft die Eigenschaften eines virtuellen Computers ab.|
|[Get-AzPublicIpAddress](https://docs.microsoft.com/powershell/module/Az.Network/Get-AzPublicIpAddress)|Ruft eine öffentliche IP-Adresse ab.|
|[Get-AzNetworkInterface](https://docs.microsoft.com/powershell/module/Az.Network/Get-AzNetworkInterface)|Ruft eine Netzwerkschnittstelle ab.|

## <a name="next-steps"></a>Nächste Schritte

Weitere Informationen zum Azure PowerShell-Modul finden Sie in der [Azure PowerShell-Dokumentation](https://docs.microsoft.com/powershell/azure/overview).

Zusätzliche VM-PowerShell-Skriptbeispiele finden Sie in der [Dokumentation zu Windows-VMs in Azure](https://docs.microsoft.com/azure/virtual-machines/windows/powershell-samples?toc=/azure/virtual-machines/windows/toc.json).


