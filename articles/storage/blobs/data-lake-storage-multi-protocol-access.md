---
title: Multiprotokollzugriff für Azure Data Lake Storage | Microsoft-Dokumentation
description: Verwenden von BLOB-APIs und Anwendungen, die BLOB-APIs mit Azure Data Lake Storage Gen2 verwenden.
services: storage
author: normesta
ms.subservice: data-lake-storage-gen2
ms.service: storage
ms.topic: conceptual
ms.date: 07/17/2019
ms.author: normesta
ms.openlocfilehash: f384fb738fe719b8e622e8d61502e6acba2bbf31
ms.sourcegitcommit: da0a8676b3c5283fddcd94cdd9044c3b99815046
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 07/18/2019
ms.locfileid: "68314375"
---
# <a name="multi-protocol-access-on-azure-data-lake-storage"></a>Multiprotokollzugriff für Azure Data Lake Storage

BLOB-APIs können jetzt für Konten verwendet werden, die über einen hierarchischen Namespace verfügen. Dadurch werden das gesamte Ökosystem von Tools, Anwendungen und Diensten sowie alle BLOB Storage-Features für Konten mit einem hierarchischen Namespace verfügbar.

Bis vor Kurzem mussten Sie eventuell jeweils eigene Speicherlösungen für den Objektspeicher und den Analysespeicher verwalten. Die Ursache hierfür war die beschränkte Unterstützung des Ökosystems durch Azure Data Lake Storage Gen2. Der Zugriff auf BLOB-Dienstfeatures, z. B. die Diagnoseprotokollierung, war ebenfalls beschränkt. Eine fragmentierte Speicherlösung ist schwierig zu verwalten, da für verschiedene Szenarien Daten zwischen Konten verschoben werden müssen. Dies ist nicht mehr erforderlich.

> [!NOTE]
> Der Zugriff auf mehrere Protokolle in Data Lake Storage ist in der Public Preview verfügbar, jedoch nur in den Regionen **USA, Westen 2** und **USA, Westen-Mitte**. Informationen zu den Einschränkungen finden Sie im Artikel [Bekannte Probleme](data-lake-storage-known-issues.md). Informationen zum Registrieren für die Vorschauversion finden Sie auf [dieser Seite](https://aka.ms/blobinteropsignup).

## <a name="use-the-entire-ecosystem-of-applications-tools-and-services"></a>Verwenden des gesamten Ökosystems von Anwendungen, Tools und Diensten

Wenn Sie sich für die Vorschauversion des Zugriffs auf mehrere Protokolle in Data Lake Storage registrieren, können Sie mithilfe des gesamten Ökosystems von Tools, Anwendungen und Diensten mit Ihren sämtlichen Daten arbeiten. Dies umfasst Azure-Dienste, z. B. [Azure Stream Analytics](https://docs.microsoft.com/azure/stream-analytics/stream-analytics-introduction), [IoT Hub](https://docs.microsoft.com/azure/iot-hub/), [Power BI](https://docs.microsoft.com/power-bi/desktop-data-sources) und viele andere. 

Dies beinhaltet auch Tools und Anwendungen von Drittanbietern. Sie können sie auf Konten mit einem hierarchischen Namespace verweisen, ohne sie ändern zu müssen. Diese Anwendungen *erfordern keine Änderungen*, selbst wenn sie BLOB-APIs aufrufen, da BLOB-APIs jetzt für Daten in Konten mit einem hierarchischen Namespace verwendet werden können.

> [!NOTE]
> Informationen zu den Einschränkungen finden Sie im Artikel [Bekannte Probleme](data-lake-storage-known-issues.md).

## <a name="use-all-blob-storage-features"></a>Verwenden sämtlicher BLOB Storage-Features

BLOB Storage-Features, z. B. [Diagnoseprotokollierung](../common/storage-analytics-logging.md), [Zugriffsebenen](storage-blob-storage-tiers.md) und [Richtlinien für die Azure Blob Storage-Lebenszyklusverwaltung](storage-lifecycle-management-concepts.md) können jetzt für Konten mit einem hierarchischen Namespace verwendet werden. Daher können Sie hierarchische Namespaces in Ihren BLOB Storage-Konten aktivieren, ohne den Zugriff auf diese wichtigen Features zu verlieren. 

> [!NOTE]
> Informationen zu den Einschränkungen finden Sie im Artikel [Bekannte Probleme](data-lake-storage-known-issues.md).

## <a name="how-multi-protocol-access-on-data-lake-storage-works"></a>Funktionsweise des Multiprotokollzugriffs für Azure Data Lake Storage

BLOB-APIs und Data Lake Storage Gen2-APIs können in Speicherkonten mit einem hierarchischen Namespace für dieselben Daten verwendet werden. Data Lake Storage Gen2 leitet BLOB-APIs über den hierarchischen Namespace weiter, sodass Sie die Vorteile von erstklassigen Verzeichnisvorgängen und POSIX-konformen Zugriffssteuerungslisten (Access Control Lists, ACLs) nutzen können. 

![Konzept des Multiprotokollzugriffs für Azure Data Lake Storage](./media/data-lake-storage-interop/interop-concept.png) 

Vorhandene Tools und Anwendungen, die die BLOB-API verwenden, profitieren von diesen Vorteilen automatisch. Entwickler müssen sie nicht ändern. Data Lake Storage Gen2 wendet konsistent ACLs auf Verzeichnis- und Dateiebene an, unabhängig von dem Protokoll, das von den Tools und Anwendungen für den Zugriff auf die Daten verwendet wird.   

## <a name="next-steps"></a>Nächste Schritte

Siehe [Bekannte Probleme](data-lake-storage-known-issues.md)




