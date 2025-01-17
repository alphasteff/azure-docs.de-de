---
title: Azure Migrate-Matrix für die Hyper-V-Bewertung und -Migration
description: Fasst die Einstellungen und Einschränkungen für die Bewertung und Migration von Hyper-V-VMs mit dem Azure Migrate-Dienst zusammen.
author: rayne-wiselman
manager: carmonm
ms.service: azure-migrate
ms.topic: conceptual
ms.date: 06/02/2019
ms.author: raynew
ms.openlocfilehash: f6edbe19429b38d68aea1f1ecfe426c9b2d194d0
ms.sourcegitcommit: 47ce9ac1eb1561810b8e4242c45127f7b4a4aa1a
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 07/11/2019
ms.locfileid: "67810081"
---
# <a name="support-matrix-for-hyper-v-assessment-and-migration"></a>Unterstützungsmatrix für die Hyper-V-Bewertung und -Migration

Mit dem [Azure Migrate-Dienst](migrate-overview.md) können Sie Computer bewerten und in die Microsoft Azure-Cloud migrieren. Dieser Artikel fasst die Supporteinstellungen und -einschränkungen für die Bewertung und Migration von lokalen Hyper-V-VMs zusammen.



## <a name="hyper-v-scenarios"></a>Hyper-V-Szenarien

Die Tabelle fasst die unterstützten Szenarien für Hyper-V-VMs zusammen.

**Bereitstellung** | **Details*** 
--- | --- 
**Bewertung lokaler Hyper-V-VMs** | [Richten Sie](tutorial-prepare-hyper-v.md) Ihre erste Bewertung ein.<br/><br/> [Führen](scale-hyper-v-assessment.md) Sie eine umfangreiche Bewertung aus.
**Migrieren von virtuellen Hyper-V-Computern zu Azure** | [Testen Sie](tutorial-migrate-hyper-v.md) die Migration zu Azure.

    

## <a name="azure-migrate-projects"></a>Azure Migrate-Projekte

**Unterstützung** | **Details**
--- | ---
Azure-Berechtigungen | Sie benötigen Berechtigungen für Mitwirkende oder Eigentümer im Abonnement, um ein Azure Migrate-Projekt zu erstellen.
Virtuelle Hyper-V-Computer | Bewerten Sie bis zu 10.000 Hyper-V-VMs in einem einzigen Projekt.

Ein Projekt kann im Rahmen der Bewertungseinschränkungen sowohl VMware-VMs als auch Hyper-V-VMs umfassen.


## <a name="assessment-hyper-v-host-requirements"></a>Bewertung von Anforderungen für Hyper-V-Hosts

| **Unterstützung**                | **Details**               
| :-------------------       | :------------------- |
| **Hostbereitstellung**       | Der Hyper-V-Host kann eigenständig oder in einem Cluster bereitgestellt werden. |
| **Berechtigungen**           | Sie benötigen Administratorrechte auf dem Hyper-V-Host. |
| **Betriebssystem des Hosts** | Windows Server 2016 oder Windows Server 2012 R2.<br/> Sie können keine VMs auf Hyper-V-Hosts mit Windows Server 2019 bewerten. |
| **PowerShell Remoting**   | Muss auf jedem Host aktiviert sein. |
| **Hyper-V-Replikat**       | Wenn Sie Hyper-V Replikate verwenden (oder mehrere VMs mit den gleichen VM-Identifikatoren) und sowohl die ursprünglichen als auch die replizierten VMs mit Azure Migrate ermitteln, dann ist die von Azure Migrate generierte Bewertung möglicherweise nicht korrekt. |


## <a name="assessment-hyper-v-vm-requirements"></a>Bewertung von Anforderungen für Hyper-V-VMs

| **Unterstützung**                  | **Details**               
| :----------------------------- | :------------------- |
| **Betriebssystem** | Alle [Windows](https://support.microsoft.com/help/2721672/microsoft-server-software-support-for-microsoft-azure-virtual-machines)- und [Linux](https://docs.microsoft.com/azure/virtual-machines/linux/endorsed-distros)-Betriebssysteme, die von Azure unterstützt werden. |
| **Berechtigungen**           | Sie benötigen Administratorrechte für jede Hyper-V-VM, die Sie bewerten möchten. |
| **Integrationsdienste**       | [Hyper-V Integration Services](https://docs.microsoft.com/virtualization/hyper-v-on-windows/reference/integration-services) müssen auf den von Ihnen bewerteten VMs laufen, um Betriebssysteminformationen zu erfassen. |
| **Erforderliche Änderungen für Azure** | Einige VMs erfordern möglicherweise Änderungen, damit sie in Azure ausgeführt werden können. Bei den folgenden Betriebssystemen führt Azure Migrate diese Änderungen automatisch durch:<br/> – Red Hat Enterprise Linux 6.5+, 7.0+<br/> – CentOS 6.5+, 7.0+</br> – SUSE Linux Enterprise Server 12 SP1+<br/> – Ubuntu 14.04 LTS, 16.04 LTS, 18.04 LTS<br/> – Debian 7,8<br/><br/> Bei anderen Betriebssystemen müssen Sie vor der Migration manuell Anpassungen vornehmen. Die entsprechenden Artikel enthalten eine Anleitung, wie dies zu tun ist. |
| **Linux-Start**                 | Wenn sich „/boot“ in einer dedizierten Partition befindet, sollte diese auf dem Betriebssystemdatenträger und nicht auf mehrere Datenträger verteilt vorhanden sein.<br/> Wenn „/boot“ Teil der Stammpartition (/) ist, sollte sich diese auf dem Betriebssystemdatenträger befinden und sich nicht auf andere Datenträger erstrecken. |
| **UEFI-Start**                  | Die Migration von VMs mit UEFI-Start wird nicht unterstützt. |
| **Verschlüsselte Datenträger/Volumes**    | Die Migration von VMs mit verschlüsselten Datenträgern/Volumes wird nicht unterstützt. |
| **RDM-Datenträger/Pass-Through-Datenträger**      | Wenn VMs über RDM-Datenträger oder Pass-Through-Datenträger verfügen, werden diese nicht in Azure repliziert. |
| **NFS**                        | NFS-Volumes, die als Volumes auf den VMs bereitgestellt sind, werden nicht repliziert. |
| **Zieldatenträger**                | Azure Migrate-Bewertungen empfehlen die Migration auf Azure-VMs mit nur verwalteten Datenträgern. |


## <a name="assessment-appliance-requirements"></a>Anforderungen an die Geräte für die Bewertung

Für Bewertung führt Azure Migrate eine einfache Appliance aus, die virtuelle Hyper-V-Computer ermittelt und VM-Metadaten sowie Leistungsdaten an Azure Migrate sendet. Das Gerät wird auf einem virtuellen Hyper-v-Computer ausgeführt, und Sie richten eine komprimierte Hyper-v-VHD ein, die Sie aus dem Azure-Portal herunterladen. Die folgende Tabelle fasst die Anforderungen an das Gerät zusammen.

| **Unterstützung**                | **Details**               
| :-------------------       | :------------------- |
| **Azure Migrate-Projekt**  |  Ein Gerät kann einem einzelnen Projekt zugeordnet werden.<br/> Sie können bis zu 5000 Hyper-V VMs mit einer einzigen Appliance ermitteln.
| **Hyper-V-Einschränkungen**    |  Sie stellen die Appliance als Hyper-V-VM bereit.<br/> Die bereitgestellte Appliance-VM ist Hyper-V-VM Version 5.0.<br/> Der VM-Host muss Windows Server 2012 R2 oder höher ausführen.<br/> Er benötigt genügend Platz, um 16 GB RAM, 4 virtuelle Prozessoren und 1 externen Schalter für die Appliance-VM zuzuweisen.<br/> Die Appliance benötigt eine statische oder dynamische IP-Adresse und einen Internetzugang.
| **Hyper-V-Appliance**      |  Die Appliance ist als Hyper-V-VM eingerichtet.<br/> Die zum Download bereitgestellte VHD ist Hyper-V-VM Version 5.0.
| **Host**                   | Der VM-Host, der die Appliance-VM ausführt, muss Windows Server 2012 R2 oder höher ausführen.<br/> Er benötigt genügend Platz, um 16 GB RAM, 4 virtuelle Prozessoren und einen externen Schalter für die Appliance-VM zuzuweisen.<br/> Die Appliance benötigt eine statische oder dynamische IP-Adresse und einen Internetzugang. |
| **Migrationsunterstützung**      | Um die Replikation von Computern zu starten, muss der Migrations-Gatewaydienst auf der Appliance 1.18.7141.12919 oder höher sein. Melden Sie sich bei der Appliance-Webanwendung an, um die Version zu überprüfen. |

## <a name="assessment-appliance-url-access"></a>URL-Zugriff auf die Bewertungs-Appliance

Zur Bewertung von VMs muss das Azure Migrate-Gerät mit dem Internet verbunden sein.

- Wenn Sie das Gerät bereitstellen, führt Azure Migrate eine Verbindungsüberwachung zu den in der folgenden Tabelle zusammengefassten URLs durch.
- Sollten Sie einen URL-basierten Firewallproxy verwenden, lassen Sie den Zugriff auf diese URLs in der Tabelle zu und stellen Sie sicher, dass der Proxy alle CNAME-Einträge auflöst, die er beim Durchsuchen der URLs empfängt.
- Wenn Sie einen abfangenden Proxy haben, müssen Sie möglicherweise das Serverzertifikat vom Proxyserver in die Appliance importieren. 

    
**URL** | **Details**  
--- | --- 
*.portal.azure.com | Navigation zum Azure-Portal
*.windows.net | Melden Sie sich bei Ihrem Azure-Abonnement an.
*.microsoftonline.com | Erstellung von Azure Active Directory-Anwendungen für die Kommunikation Appliance-to-Service.
management.azure.com | Erstellung von Azure Active Directory-Anwendungen für die Kommunikation Appliance-to-Service.
dc.services.visualstudio.com | Protokollierung und Überwachung 
*.vault.azure.net | Verwalten von Geheimnissen in Azure Key Vault, während der Kommunikation zwischen Appliance und Dienst.


## <a name="assessment-port-requirements"></a>Portanforderungen für die Bewertung

Die folgende Tabelle fasst die Portanforderungen für die Bewertung zusammen.

**Device** | **Connection**
--- | --- 
**Appliance** | Eingehende Verbindungen auf dem TCP-Port 3389, damit Remotedesktopverbindungen eine Verbindung mit der Appliance aufbauen können.<br/> Eingehende Verbindungen auf Port 44368, um über Remotezugriff über die URL: https://<appliance-ip-or-name>:44368 auf die Appliance-Verwaltungsanwendung zugreifen zu können.<br/> Ausgehende Verbindungen auf Port 443, um Ermittlungs- und Leistungsmetadaten an Azure Migrate zu senden.
**Hyper-V-Host/Cluster** | Eingehende Verbindungen auf den WinRM-Ports 5985 (HTTP) und 5986 (HTTPS), um Konfigurations- und Leistungsmetadaten der Hyper-V-VMs mithilfe einer CIM-Sitzung (Common Information Model) abzurufen.

## <a name="migration-hyper-v-host-requirements"></a>Migrationsanforderungen für Hyper-V-Hosts

| **Unterstützung**                | **Details**               
| :-------------------       | :------------------- |
| **Hostbereitstellung**       | Der Hyper-V-Host kann eigenständig oder in einem Cluster bereitgestellt werden. |
| **Berechtigungen**           | Sie benötigen Administratorrechte auf dem Hyper-V-Host. |
| **Betriebssystem des Hosts** | Windows Server 2019, Windows Server 2016 oder Windows Server 2012 R2. |

## <a name="migration-hyper-v-vm-requirements"></a>Migrationsanforderungen der Hyper-V-VM

| **Unterstützung**                  | **Details**               
| :----------------------------- | :------------------- |
| **Betriebssystem** | Alle [Windows](https://support.microsoft.com/help/2721672/microsoft-server-software-support-for-microsoft-azure-virtual-machines)- und [Linux](https://docs.microsoft.com/azure/virtual-machines/linux/endorsed-distros)-Betriebssysteme, die von Azure unterstützt werden. |
| **Berechtigungen**           | Sie benötigen Administratorrechte für jede Hyper-V-VM, die Sie bewerten möchten. |
| **Integrationsdienste**       | [Hyper-V Integration Services](https://docs.microsoft.com/virtualization/hyper-v-on-windows/reference/integration-services) müssen auf den von Ihnen bewerteten VMs laufen, um Betriebssysteminformationen zu erfassen. |
| **Erforderliche Änderungen für Azure** | Einige VMs erfordern möglicherweise Änderungen, damit sie in Azure ausgeführt werden können. Bei den folgenden Betriebssystemen führt Azure Migrate diese Änderungen automatisch durch:<br/> – Red Hat Enterprise Linux 6.5+, 7.0+<br/> – CentOS 6.5+, 7.0+</br> – SUSE Linux Enterprise Server 12 SP1+<br/> – Ubuntu 14.04 LTS, 16.04 LTS, 18.04 LTS<br/> – Debian 7,8<br/><br/> Bei anderen Betriebssystemen müssen Sie vor der Migration manuell Anpassungen vornehmen. Die entsprechenden Artikel enthalten eine Anleitung, wie dies zu tun ist. |
| **Linux-Start**                 | Wenn sich „/boot“ in einer dedizierten Partition befindet, sollte diese auf dem Betriebssystemdatenträger und nicht auf mehrere Datenträger verteilt vorhanden sein.<br/> Wenn „/boot“ Teil der Stammpartition (/) ist, sollte sich diese auf dem Betriebssystemdatenträger befinden und sich nicht auf andere Datenträger erstrecken. |
| **UEFI-Start**                  | Die Migration von VMs mit UEFI-Start wird nicht unterstützt. |
| **Verschlüsselte Datenträger/Volumes**    | Die Migration von VMs mit verschlüsselten Datenträgern/Volumes wird nicht unterstützt. |
| **RDM-Datenträger/Pass-Through-Datenträger**      | Wenn VMs über RDM-Datenträger oder Pass-Through-Datenträger verfügen, werden diese nicht in Azure repliziert. |
| **NFS**                        | NFS-Volumes, die als Volumes auf den VMs bereitgestellt sind, werden nicht repliziert. |
| **Zieldatenträger**                | Sie können virtuelle Azure-Computer nur mit verwalteten Datenträgern migrieren. |




## <a name="migration-hyper-v-host-url-access"></a>URL-Zugriff für Hyper-V-Hosts für die Migration

Die folgende Tabelle fasst die Anforderungen an den URL-Zugriff für Hyper-V-Hosts zusammen.

**URL** | **Details**  
--- | ---
login.microsoftonline.com | Zugriffssteuerung und Identitätsverwaltung mit Active Directory.
*.backup.windowsazure.com | Für die Übertragung und Koordinierung von Replikationsdaten.
*.hypervrecoverymanager.windowsazure.com | Verbindung mit Azure Migrate-Dienst-URLs herstellen.
*.blob.core.windows.net | Hochladen von Daten in Speicherkonten.
dc.services.visualstudio.com | Laden Sie Anwendungsprotokolle hoch, die für die interne Überwachung verwendet werden.
time.windows.com | Überprüft die Zeitsynchronisierung zwischen Systemzeit und der globalen Zeit.

## <a name="migration-port-access"></a>Zugriff auf den Migrationsport

Die folgende Tabelle fasst die Portanforderungen an Hyper-V-Hosts und VMs für die VM-Migration zusammen.

**Device** | **Connection**
--- | --- 
Hyper-V-Hosts/VMs | Ausgehende Verbindungen auf HTTPS-Port 443 zum Senden von VM-Replikationsdaten an Azure Migrate.

  
## <a name="migration-vm-disk-support"></a>VM-Datenträgersupport für die Migration 

**Unterstützung** | **Details**
--- | ---
Migrierte Datenträger | VMs können nur auf verwaltete Datenträger (HHD Standard, SSD Premium) in Azure migriert werden.
   

## <a name="next-steps"></a>Nächste Schritte

[Vorbereitung auf die Hyper-V VM-Bewertung](tutorial-prepare-hyper-v.md) für die Migration.




 
