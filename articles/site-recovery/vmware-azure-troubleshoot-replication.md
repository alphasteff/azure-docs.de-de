---
title: Beheben von Replikationsproblemen bei der Notfallwiederherstellung von VMware-VMs und physischen Servern in Azure mit Azure Site Recovery | Microsoft-Dokumentation
description: Dieser Artikel enthält Informationen zur Problembehandlung für häufige Replikationsprobleme bei der Notfallwiederherstellung von VMware-VMs und physischen Servern in Azure mit Azure Site Recovery.
author: mayurigupta13
manager: rochakm
ms.service: site-recovery
ms.topic: article
ms.date: 06/27/2019
ms.author: mayg
ms.openlocfilehash: ed04c21fc5f3aecb91483dbd1eb7ca5fbf47c3e9
ms.sourcegitcommit: 47ce9ac1eb1561810b8e4242c45127f7b4a4aa1a
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 07/11/2019
ms.locfileid: "67805963"
---
# <a name="troubleshoot-replication-issues-for-vmware-vms-and-physical-servers"></a>Beheben von Problemen bei der Replikation von VMware-VMs und physischen Servern

Dieser Artikel beschreibt einige häufig auftretende Probleme und bestimmte Fehler bei der Replikation von lokalen VMware-VMs und physischen Servern in Azure mithilfe von [Site Recovery](site-recovery-overview.md).

## <a name="step-1-monitor-process-server-health"></a>Schritt 1: Überwachen der Integrität des Prozessservers

Site Recovery verwendet den [Prozessserver](vmware-physical-azure-config-process-server-overview.md#process-server), um replizierte Daten zu empfangen, zu optimieren und an Azure zu senden.

Wir empfehlen, dass Sie die Integrität von Prozessservern im Portal überwachen, um sicherzustellen, dass sie verbunden sind und ordnungsgemäß funktionieren sowie dass die Replikation für die dem Prozessserver zugeordneten Quellcomputer ausgeführt wird.

- [Erfahren Sie mehr](vmware-physical-azure-monitor-process-server.md) zur Überwachung von Prozessservern.
- [Bewährte Methoden](vmware-physical-azure-troubleshoot-process-server.md#best-practices-for-process-server-deployment)
- [Problembehandlung](vmware-physical-azure-troubleshoot-process-server.md#check-process-server-health) bei der Integrität des Prozessservers

## <a name="step-2-troubleshoot-connectivity-and-replication-issues"></a>Schritt 2: Problembehandlung bei Konnektivität und Replikation

Fehler bei der ersten und fortlaufenden Replikation werden häufig von Konnektivitätsproblemen zwischen dem Quellserver und dem Prozessserver oder zwischen dem Prozessserver und Azure verursacht. 

Informationen zum Beheben dieser Probleme finden Sie unter [Problembehandlung der Konnektivität und Replikation](vmware-physical-azure-troubleshoot-process-server.md#check-connectivity-and-replication).




## <a name="step-3-troubleshoot-source-machines-that-arent-available-for-replication"></a>Schritt 3: Problembehandlung bei Quellcomputern, die für die Replikation nicht zur Verfügung stehen

Wenn Sie versuchen, den Quellcomputer auszuwählen, um Replikation mithilfe von Site Recovery zu aktivieren, steht der Computer möglicherweise aus einem der folgenden Gründe nicht zur Verfügung:

* **Zwei virtuelle Computer mit derselben Instanz-UUID**: Wenn zwei virtuelle Computer im vCenter dieselbe Instanz-UUID haben, wird der erste vom Konfigurationsserver ermittelte virtuelle Computer im Azure-Portal angezeigt. Um dieses Problem zu beheben, stellen Sie sicher, dass keine zwei virtuellen Computer über dieselbe Instanz-UUID verfügen. Dieses Szenario tritt häufig in Instanzen auf, bei denen ein virtueller Sicherungscomputer aktiv und in unseren Ermittlungsdatensätzen protokolliert wird. Informationen zum Beheben dieses Problems finden Sie unter [Azure Site Recovery VMware-zu-Azure: Bereinigen von doppelten oder veralteten Einträgen](https://social.technet.microsoft.com/wiki/contents/articles/32026.asr-vmware-to-azure-how-to-cleanup-duplicatestale-entries.aspx).
* **Falsche vCenter-Benutzeranmeldeinformationen**: Stellen Sie sicher, dass Sie während der Einrichtung des Konfigurationsservers mithilfe der OVF-Vorlage oder des einheitlichen Setups die richtigen vCenter-Anmeldeinformationen hinzugefügt haben. Informationen zum Überprüfen der Anmeldeinformationen, die Sie während der Einrichtung hinzugefügt haben, finden Sie unter [Ändern der Anmeldeinformationen für die automatische Ermittlung](vmware-azure-manage-configuration-server.md#modify-credentials-for-automatic-discovery).
* **Unzureichende vCenter-Berechtigungen**: Wenn die für den Zugriff auf vCenter bereitgestellten Berechtigungen nicht die erforderlichen Berechtigungen umfassen, kann die Ermittlung von virtuellen Computern möglicherweise fehlschlagen. Stellen Sie sicher, dass die unter [Vorbereiten eines Kontos für die automatische Ermittlung](vmware-azure-tutorial-prepare-on-premises.md#prepare-an-account-for-automatic-discovery) beschriebenen Berechtigungen dem vCenter-Benutzerkonto hinzugefügt werden.
* **Azure Site Recovery-Verwaltungsserver**: Wenn der virtuelle Computer als Verwaltungsserver unter mindestens einer Rolle aus Konfigurationsserver, horizontal skalierter Prozessserver oder Masterzielserver verwendet wird, können Sie den virtuellen Computer im Portal nicht auswählen. Verwaltungsserver können nicht repliziert werden.
* **Bereits geschützt/Failover durch Azure Site Recovery-Dienste ausgeführt**: Wenn der virtuelle Computer bereits mithilfe von Site Recovery geschützt wird bzw. ein Failover dafür ausgeführt wurde, steht er nicht mehr zur Auswahl für den Schutz im Portal zur Verfügung. Stellen Sie sicher, dass der gesuchte virtuelle Computer im Portal noch nicht von einem anderen Benutzer oder unter einem anderen Abonnement geschützt wurde.
* **vCenter nicht verbunden**: Überprüfen Sie, ob sich vCenter im verbundenen Zustand befindet. Dazu wechseln Sie zu Recovery Services-Tresor > Site Recovery-Infrastruktur > Konfigurationsserver > Auf den jeweiligen Konfigurationsserver klicken > auf der rechten Seite wird ein Blatt mit Details zu zugeordneten Servern geöffnet. Überprüfen Sie, ob vCenter verbunden ist. Falls es sich im Zustand „Nicht verbunden“ befindet, beheben Sie das Problem, und [aktualisieren Sie den Konfigurationsserver](vmware-azure-manage-configuration-server.md#refresh-configuration-server) im Portal. Danach wird der virtuelle Computer im Portal aufgeführt.
* **ESXi ausgeschaltet**: Wenn der ESXi-Host, auf dem sich der virtuelle Computer befindet, ausgeschaltet ist, wird der virtuelle Computer im Azure-Portal nicht aufgeführt und kann nicht ausgewählt werden. Schalten Sie den ESXi-Host ein, und [aktualisieren Sie den Konfigurationsserver](vmware-azure-manage-configuration-server.md#refresh-configuration-server) im Portal. Danach wird der virtuelle Computer im Portal aufgeführt.
* **Ausstehender Neustart**: Wenn auf dem virtuellen Computer ein Neustart aussteht, können Sie den Computer im Azure-Portal nicht auswählen. Schließen Sie die ausstehenden Neustartaktivitäten ab, und [aktualisieren Sie den Konfigurationsserver](vmware-azure-manage-configuration-server.md#refresh-configuration-server). Danach wird der virtuelle Computer im Portal aufgeführt.
* **IP nicht gefunden**: Wenn dem virtuellen Computer keine gültige IP-Adresse zugeordnet ist, können Sie ihn im Azure-Portal nicht auswählen. Weisen Sie dem virtuellen Computer eine gültige IP-Adresse zu, und [aktualisieren Sie den Konfigurationsserver](vmware-azure-manage-configuration-server.md#refresh-configuration-server). Danach wird der virtuelle Computer im Portal aufgeführt.

### <a name="troubleshoot-protected-virtual-machines-greyed-out-in-the-portal"></a>Problembehandlung – geschützte virtuelle Computer im Portal grau dargestellt

Virtuelle Computer, die unter Site Recovery repliziert werden, sind im Azure-Portal nicht verfügbar, wenn doppelte Einträge im System vorhanden sind. Informationen zum Löschen veralteter Einträge sowie zum Beheben des Problems finden Sie unter [Azure Site Recovery VMware-zu-Azure: Gewusst wie: Bereinigen von doppelten oder veralteten Einträgen](https://social.technet.microsoft.com/wiki/contents/articles/32026.asr-vmware-to-azure-how-to-cleanup-duplicatestale-entries.aspx).

## <a name="common-errors-and-solutions"></a>Häufige Fehler und Lösungen

### <a name="initial-replication-issues-error-78169"></a>Probleme bei der Erstreplikation [Fehler 78169]

Stellen Sie sicher, dass es keine Probleme im Zusammenhang mit Konnektivität, Bandbreite oder Zeitsynchronisierung gibt, und achten Sie zusätzlich auf Folgendes:

- Azure Site Recovery wird von keiner Antivirensoftware blockiert. Erfahren Sie [mehr](vmware-azure-set-up-source.md#azure-site-recovery-folder-exclusions-from-antivirus-program) zu Ordnerausschlüssen, die für Azure Site Recovery erforderlich sind.

### <a name="missing-app-consistent-recovery-points-error-78144"></a>App-konsistente Wiederherstellungspunkte fehlen [Fehler 78144]

 Dieser Fehler tritt aufgrund von Problemen mit dem Volumeschattenkopie-Dienst (Volume Shadow Copy Service, VSS) auf. Behebung: 
 
- Stellen Sie sicher, das Sie mindestens Version 9.22.2 des Azure Site Recovery-Agents installiert haben. 
- Stellen Sie sicher, dass der VSS-Anbieter als Dienst in den Windows-Diensten installiert ist, und überprüfen Sie die Komponentendienst-MMC, um sicherzustellen, dass der Azure Site Recovery-VSS-Anbieter aufgeführt wird.
- Wenn der VSS-Anbieter nicht installiert ist, erhalten Sie im [Artikel zur Problembehandlung bei Installationsfehlern](vmware-azure-troubleshoot-push-install.md#vss-installation-failures) weitere Informationen.

- Wenn VSS deaktiviert ist:
    - Stellen Sie sicher, dass der Starttyp des VSS-Anbieterdiensts auf **Automatic** (Automatisch) festgelegt ist.
    - Starten Sie die folgenden Dienste neu:
        - VSS-Dienst
        - Azure Site Recovery-VSS-Anbieter
        - VDS-Dienst

- Wenn Sie SQL oder Exchange-Workloads ausführen, sollten Sie die Protokolle dieser Anwendungsschreiber auf Fehler überprüfen. Häufige Fehler und ihre Lösungen werden in den folgenden Artikeln thematisiert:
    -  [Automatische Schließoption der SQL Server-Datenbank ist TRUE](https://support.microsoft.com/help/4504104)
    - [SQL Server 2008 R2 zeigt einen nicht wiederholbaren Fehler](https://support.microsoft.com/help/4504103)
    - [Bekanntes Problem in SQL Server 2016 und 2017](https://support.microsoft.com/help/4493364)
    - [Häufige Probleme mit Exchange Servers 2013 und 2016](https://support.microsoft.com/help/4037535)


### <a name="source-machines-with-high-churn-error-78188"></a>Quellcomputer mit hoher Änderungsrate [Fehler 78188]

Mögliche Ursachen:
- Die Änderungsrate der Daten (geschriebene Byte/Sekunde) auf den aufgelisteten Datenträgern der VM übersteigt die [unterstützten Grenzwerte von Azure Site Recovery](site-recovery-vmware-deployment-planner-analyze-report.md#azure-site-recovery-limits) für den Typ des Speicherkontos des Replikationsziels.
- Es gibt eine Spitze in der Änderungsrate, weshalb große Mengen an Daten noch hochgeladen werden müssen.

So lösen Sie das Problem:
- Stellen Sie sicher, dass der Typ des Zielspeicherkontos (Standard oder Premium) den Anforderungen der Änderungsrate der Quelle entsprechend bereitgestellt wird.
- Wenn die erkannte erhöhte Änderungsrate nur vorrübergehend ist, warten Sie einigen Stunden ab, damit alle ausstehenden Daten hochgeladen werden. Dann können Sie einen Wiederherstellungspunkt erstellen.
- Wenn das Problem weiterhin besteht, können Sie die Replikation mithilfe des [Bereitstellungsplaners](site-recovery-deployment-planner.md#overview) von Site Recovery planen.

### <a name="source-machines-with-no-heartbeat-error-78174"></a>Quellcomputer ohne Heartbeat [Fehler 78174]

Dieser Fehler tritt auf, wenn der Azure Site Recovery-Mobility-Agent auf dem Quellcomputer nicht mit dem Konfigurationsserver kommuniziert.

Führen Sie die folgenden Schritte durch, um die Netzwerkverbindung zwischen der Quell-VM und dem Konfigurationsserver zu überprüfen:

1. Stellen Sie sicher, dass der Quellcomputer ausgeführt wird.
2. Melden Sie sich beim Quellcomputer mit einem Administratorkonto an.
3. Stellen Sie sicher, dass die folgenden Dienste ausgeführt werden. Wenn diese nicht ausgeführt werden, starten Sie sie neu.
   - Svagents (InMage Scout VX-Agent)
   - InMage Scout-Anwendungsdienst
4. Sehen Sie sich die Fehlerdetails in den Protokollen an folgendem Speicherort auf dem Quellcomputer an:

       C:\Program Files (X86)\Microsoft Azure Site Recovery\agent\svagents*log
    
### <a name="process-server-with-no-heartbeat-error-806"></a>Prozessserver ohne Heartbeat [Fehler 806]
Wenn es keinen Heartbeat vom Prozessserver gibt, überprüfen Sie Folgendes:
1. Die Prozessserver-VM wird ordnungsgemäß ausgeführt.
2. Die Fehlerdetails in den folgenden Protokollen:

       C:\ProgramData\ASR\home\svsystems\eventmanager*.log
       and
       C:\ProgramData\ASR\home\svsystems\monitor_protection*.log

### <a name="master-target-server-with-no-heartbeat-error-78022"></a>Masterzielserver ohne Heartbeat [Fehler 78022]

Dieser Fehler tritt auf, wenn der Azure Site Recovery-Mobility-Agent auf dem Masterziel nicht mit dem Konfigurationsserver kommuniziert.

Führen Sie zur Problembehebung die folgenden Schritte durch, um den Dienststatus zu überprüfen:

1. Stellen Sie sicher, dass die Masterziel-VM ausgeführt wird.
2. Melden Sie sich bei der Masterziel-VM mit einem Administratorkonto an.
    - Stellen Sie sicher, dass der svagents-Dienst ausgeführt wird. Wird der Dienst ausgeführt, starten Sie Ihn neu.
    - Sehen Sie sich die Fehlerdetails in den Protokollen an folgendem Speicherort an:
        
          C:\Program Files (X86)\Microsoft Azure Site Recovery\agent\svagents*log

## <a name="error-id-78144---no-app-consistent-recovery-point-available-for-the-vm-in-the-last-xxx-minutes"></a>Fehler-ID 78144: In den letzten „XXX“ Minuten stand kein anwendungskonsistenter Wiederherstellungspunkt für den virtuellen Computer zur Verfügung.

Einige der häufigsten Probleme sind nachfolgend aufgeführt:

#### <a name="cause-1-known-issue-in-sql-server-20082008-r2"></a>Ursache 1: Bekanntes Problem in SQL Server 2008/2008 R2 
**Problembehandlung**: Es gibt ein bekanntes Problem mit SQL Server 2008/2008 R2. Weitere Informationen finden Sie in diesem KB-Artikel: [Fehler bei Azure Site Recovery-Agent oder einer anderen komponentenfreien VSS-Sicherung für einen Server, der SQL Server 2008 R2 hostet](https://support.microsoft.com/help/4504103/non-component-vss-backup-fails-for-server-hosting-sql-server-2008-r2).

#### <a name="cause-2-azure-site-recovery-jobs-fail-on-servers-hosting-any-version-of-sql-server-instances-with-autoclose-dbs"></a>Ursache 2: Azure Site Recovery-Aufträge schlagen auf Servern fehl, die eine beliebige Version von SQL Server-Instanzen mit AUTO_CLOSE DBs hosten 
**Problembehandlung**: Referenz-KB-[Artikel](https://support.microsoft.com/help/4504104/non-component-vss-backups-such-as-azure-site-recovery-jobs-fail-on-ser) 


#### <a name="cause-3-known-issue-in-sql-server-2016-and-2017"></a>Ursache 3: Bekanntes Problem in SQL Server 2016 und 2017
**Problembehandlung**: Referenz-KB-[Artikel](https://support.microsoft.com/help/4493364/fix-error-occurs-when-you-back-up-a-virtual-machine-with-non-component) 


### <a name="more-causes-due-to-vss-related-issues"></a>Weitere Ursachen aufgrund von VSS-bezogenen Problemen:

Überprüfen Sie zur weiteren Problembehandlung die Dateien auf dem Quellcomputer, um den genauen Fehlercode zu erhalten:
    
    C:\Program Files (x86)\Microsoft Azure Site Recovery\agent\Application Data\ApplicationPolicyLogs\vacp.log

Wie können die Fehler in der Datei gefunden werden?
Suchen Sie nach der Zeichenfolge „vacpError“, indem Sie die Datei „vacp.log“ in einem Editor öffnen.
        
    Ex: vacpError:220#Following disks are in FilteringStopped state [\\.\PHYSICALDRIVE1=5, ]#220|^|224#FAILED: CheckWriterStatus().#2147754994|^|226#FAILED to revoke tags.FAILED: CheckWriterStatus().#2147754994|^|

Im obigen Beispiel ist **2147754994** der Fehlercode, der Sie über den Fehler informiert, wie unten dargestellt.

#### <a name="vss-writer-is-not-installed---error-2147221164"></a>VSS Writer ist nicht installiert – Fehler 2147221164 

*Problembehandlung*: Azure Site Recovery verwendet den Microsoft-VSS (Volumeschattenkopie-Dienst), um Tags für die Anwendungskonsistenz zu erstellen. Es installiert einen VSS-Anbieter für den Betrieb, um Momentaufnahmen zur Anwendungskonsistenz zu erstellen. Dieser VSS-Anbieter wird als Dienst installiert. Falls der VSS-Anbieterdienst nicht installiert ist, tritt bei der Erstellung der Momentaufnahme zur Anwendungskonsistenz ein Fehler mit der ID „0x80040154“ auf – „Klasse nicht registriert“. </br>
Weitere Informationen finden Sie im [Artikel zur Problembehandlung bei der VSS Writer-Installation.](https://docs.microsoft.com/azure/site-recovery/vmware-azure-troubleshoot-push-install#vss-installation-failures) 

#### <a name="vss-writer-is-disabled---error-2147943458"></a>VSS Writer ist deaktiviert – Fehler 2147943458

**Problembehandlung**: Azure Site Recovery verwendet den Microsoft-VSS (Volumeschattenkopie-Dienst), um Tags für die Anwendungskonsistenz zu erstellen. Es installiert einen VSS-Anbieter für den Betrieb, um Momentaufnahmen zur Anwendungskonsistenz zu erstellen. Dieser VSS-Anbieter wird als Dienst installiert. Falls der VSS-Anbieterdienst deaktiviert ist, tritt bei der Erstellung der Momentaufnahme zur Anwendungskonsistenz ein Fehler mit der ID „Der angegebene Dienst ist deaktiviert und kann nicht gestartet werden (0x80070422)“ auf. </br>

- Wenn VSS deaktiviert ist:
    - Stellen Sie sicher, dass der Starttyp des VSS-Anbieterdiensts auf **Automatic** (Automatisch) festgelegt ist.
    - Starten Sie die folgenden Dienste neu:
        - VSS-Dienst
        - Azure Site Recovery-VSS-Anbieter
        - VDS-Dienst

####  <a name="vss-provider-notregistered---error-2147754756"></a>VSS PROVIDER NOT_REGISTERED – Fehler 2147754756

**Problembehandlung**: Azure Site Recovery verwendet den Microsoft-VSS (Volumeschattenkopie-Dienst), um Tags für die Anwendungskonsistenz zu erstellen. Überprüfen Sie, ob der Azure Site Recovery VSS-Anbieterdienst installiert ist. </br>

- Wiederholen Sie die Installation des Anbieters mit den folgenden Befehlen:
- Vorhandenen Anbieter deinstallieren: C:\Program Files (x86)\Microsoft Azure Site Recovery\agent\InMageVSSProvider_Uninstall.cmd
- Neuinstallation von: C:\Program Files (x86)\Microsoft Azure Site Recovery\agent\InMageVSSProvider_Install.cmd
 
Stellen Sie sicher, dass der Starttyp des VSS-Anbieterdiensts auf **Automatic** (Automatisch) festgelegt ist.
    - Starten Sie die folgenden Dienste neu:
        - VSS-Dienst
        - Azure Site Recovery-VSS-Anbieter
        - VDS-Dienst

## <a name="next-steps"></a>Nächste Schritte

Wenn Sie weitere Hilfe benötigen, können Sie Ihre Frage im [Azure Site Recovery-Forum](https://social.msdn.microsoft.com/Forums/azure/home?forum=hypervrecovmgr) veröffentlichen. Unsere Community ist sehr aktiv, und einer unserer Techniker kann Sie bei Ihrem Problem unterstützen.
