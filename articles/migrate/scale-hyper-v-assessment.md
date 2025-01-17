---
title: Bewerten einer großen Anzahl von Hyper-V-VMs für die Migration zu Azure mit Azure Migrate | Microsoft-Dokumentation
description: Hier erfahren Sie, wie Sie eine große Anzahl von Hyper-V-VMs mithilfe des Azure Migrate-Diensts für die Migration zu Azure bewerten.
author: rayne-wiselman
manager: carmonm
ms.service: azure-migrate
ms.topic: conceptual
ms.date: 07/10/2019
ms.author: raynew
ms.openlocfilehash: 95704f2694892b349d0967fca2160dabd990b472
ms.sourcegitcommit: 47ce9ac1eb1561810b8e4242c45127f7b4a4aa1a
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 07/11/2019
ms.locfileid: "67810141"
---
# <a name="assess-large-numbers-of-hyper-v-vms-for-migration-to-azure"></a>Bewerten einer großen Anzahl von virtuellen Hyper-V-Computern für die Migration zu Azure

In diesem Artikel wird beschrieben, wie eine große Anzahl von lokalen Hyper-V-VMs (> 1.000) mithilfe des Azure Migrate-Serverbewertungstools für die Migration zu Azure bewertet wird.

[Azure Migrate](migrate-services-overview.md) stellt einen Hub mit Tools bereit, die Ihnen dabei helfen, Apps, Infrastrukturen und Workloads zu ermitteln, zu bewerten und zu Microsoft Azure zu migrieren. Der Hub umfasst Azure Migrate-Tools sowie Angebote von unabhängigen Drittanbietern (Independent Software Vendors, ISVs). 


In diesem Artikel werden folgende Vorgehensweisen behandelt:
> [!div class="checklist"]
> * Planen Sie die Bewertung in gewünschtem Umfang.
> * Konfigurieren Sie Azure-Berechtigungen, und bereiten Sie Hyper-V für die Bewertung vor.
> * Erstellen Sie ein Azure Migrate-Projekt, und erstellen Sie eine Bewertung.
> * Überprüfen Sie die Bewertung beim Planen der Migration.


> [!NOTE]
> Wenn Sie einen Proof of Concept ausprobieren möchten, um einige virtuelle Computer zu bewerten, bevor Sie eine Bewertung im gewünschtem Umfang durchführen, lesen Sie unsere [Tutorialreihe](tutorial-prepare-hyper-v.md).

## <a name="plan-for-assessment"></a>Planen für die Bewertung

Bei der Planung für die Bewertung einer großen Anzahl von Hyper-V-VMS sind einige Dinge zu berücksichtigen:

- **Planen von Azure Migrate-Projekten**: Finden Sie heraus, wie Sie Azure Migrate-Projekte bereitstellen möchten. Wenn sich Ihre Rechenzentren beispielsweise in verschiedenen geografischen Regionen befinden oder Sie ermittlungs-, bewertungs- oder migrationsspezifische Metadaten in einer anderen geografischen Region speichern müssen, benötigen Sie möglicherweise mehrere Projekte.
- **Planen von Appliances**: Azure Migrate verwendet eine lokale Azure Migrate-Appliance, die als Hyper-V-VM bereitgestellt wird, um VMs kontinuierlich für die Bewertung und Migration zu identifizieren. Die Appliance überwacht Umgebungsänderungen wie z.B. das Hinzufügen von VMs, Datenträgern oder Netzwerkadaptern. Sie sendet außerdem deren Meta- und Leistungsdaten an Azure. Sie müssen herausfinden, wie viele Appliances bereitgestellt werden sollen.


## <a name="planning-limits"></a>Planen von Einschränkungen
 
Verwenden Sie die in dieser Tabelle zusammengefassten Einschränkungen für die Planung.

**Planung** | **Einschränkungen**
--- | --- 
**Azure Migrate-Projekte** | Bewerten Sie bis zu 10.000 VMs in einem Projekt.
**Azure Migrate-Appliance** | Eine einzelne Appliance kann bis zu 5.000 virtuelle Computer ermitteln.<br/> Eine einzelne Appliance kann Verbindungen mit bis zu 300 Hyper-V-Hosts herstellen.<br/> Eine einzelne Appliance kann nur einer einzelnen Azure Migrate-Ressource zugeordnet werden.<br/><br/> 
**Azure Migrate-Bewertung** | Sie können bis zu 10.000 virtuelle Computer in einer einzelnen Bewertung bewerten.



## <a name="other-planning-considerations"></a>Weitere Überlegungen zur Planung

- Um die Ermittlung von der Appliance aus zu starten, müssen Sie jeden Hyper-V-Host auswählen. 
- Wenn Sie eine mehrinstanzenfähige Umgebung ausführen, können Sie zurzeit nur VMS ermitteln, die zu einem bestimmten Mandanten gehören. 

## <a name="prepare-for-assessment"></a>Vorbereiten für die Bewertung

Bereiten Sie Azure und Hyper-V für die Serverbewertung vor. 

1. Überprüfen Sie die [Hyper-V-Unterstützungsanforderungen und -Einschränkungen](migrate-support-matrix-hyper-v.md).
2. Richten sie Berechtigungen für Ihr Azure-Konto zur Interaktion mit Azure Migrate ein.
3. Bereiten Sie die Hyper-V-Hosts und VMs vor.

Befolgen Sie die Anweisungen in [diesem Tutorial](tutorial-prepare-hyper-v.md), um diese Einstellungen zu konfigurieren.

## <a name="create-a-project"></a>Erstellen eines Projekts

Führen Sie in Übereinstimmung mit ihren Planungsanforderungen folgende Schritte aus:

1. Erstellen Sie Azure Migrate-Projekte.
2. Fügen Sie den Projekten das Azure Migrate-Serverbewertungs-Tool hinzu.

[Weitere Informationen](how-to-add-tool-first-time.md)

## <a name="create-and-review-an-assessment"></a>Erstellen und Überprüfen einer Bewertung

1. Erstellen Sie Bewertungen für Hyper-V-VMS.
1. Überprüfen Sie die Bewertungen in Vorbereitung auf die Migrationsplanung.

[Erfahren Sie mehr](tutorial-assess-hyper-v.md) über das Erstellen und Überprüfen von Bewertungen.
    

## <a name="next-steps"></a>Nächste Schritte

In diesem Artikel führen Sie folgende Schritte aus:
 
> [!div class="checklist"] 
> * Planen von Azure Migrate-Bewertungen für Hyper-V-VMS in gewünschtem Umfang
> * Vorbereiten von Azure und Hyper-V auf die Bewertung
> * Erstellen eines Azure Migrate-Projekts und Ausführen von Bewertungen
> * Überprüfen der Bewertungen in Vorbereitung auf die Migrationsplanung.

[Erfahren Sie nun](concepts-assessment-calculation.md), wie Bewertungen berechnet und [geändert](how-to-modify-assessment.md) werden.
