---
title: Einführung in Azure-Warteschlangen – Azure Storage
description: Einführung in Azure-Warteschlangen
author: mhopkins-msft
ms.author: mhopkins
ms.date: 06/07/2019
ms.service: storage
ms.subservice: queues
ms.topic: overview
ms.reviewer: cbrooks
ms.openlocfilehash: 67e4874fcca93633140b7630ceadd273d1646f86
ms.sourcegitcommit: 85b3973b104111f536dc5eccf8026749084d8789
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 08/01/2019
ms.locfileid: "68721176"
---
# <a name="what-are-azure-queues"></a>Was sind Azure-Warteschlangen?

Azure Queue Storage ist ein Dienst für die Speicherung einer großen Anzahl von Nachrichten. Sie können überall auf der Welt über authentifizierte Aufrufe mithilfe von HTTP oder HTTPS auf Nachrichten zugreifen. Eine Warteschlangennachricht kann bis zu 64 KB groß sein. Eine Warteschlange kann Millionen Nachrichten enthalten, bis die maximale Kapazität eines Speicherkontos erreicht ist.

## <a name="common-uses"></a>Gängige Nutzungsszenarien

Warteschlangenspeicherungen werden hauptsächlich für folgende Zwecke verwendet:

* Erstellung eines Arbeits-Backlogs zur asynchronen Verarbeitung
* Weiterleitung von Nachrichten von einer Azure-Webrolle an eine Azure-Workerrolle

## <a name="queue-service-concepts"></a>Konzepte des Warteschlangendiensts

Der Warteschlangendienst umfasst die folgenden Komponenten:

![Konzepte des Warteschlangendiensts](./media/storage-queues-introduction/queue1.png)

* **URL-Format:** Warteschlangen können über das folgende URL-Format aufgerufen werden:

    `https://<storage account>.queue.core.windows.net/<queue>`
  
    Mit der folgenden URL kann eine der Warteschlangen im Diagramm adressiert werden:  
  
    `https://myaccount.queue.core.windows.net/images-to-download`

* **Speicherkonto:** Alle Zugriffe auf den Azure-Speicher erfolgen über ein Speicherkonto. Weitere Informationen zur Kapazität der Speicherkonten finden Sie unter [Azure Storage Scalability and Performance Targets](../common/storage-scalability-targets.md?toc=%2fazure%2fstorage%2fqueues%2ftoc.json) (Skalierbarkeits- und Leistungsziele für Microsoft Azure-Speicher, in englischer Sprache).

* **Warteschlange:** Eine Warteschlange enthält einen Satz von Nachrichten. Der Warteschlangenname **muss** ausschließlich aus Kleinbuchstaben bestehen. Informationen zum Benennen von Warteschlangen finden Sie unter [Benennen von Warteschlangen und Metadaten](https://msdn.microsoft.com/library/azure/dd179349.aspx).

* **Nachricht:** Eine Nachricht in einem beliebigen Format und mit einer Größe von bis zu 64 KB. Vor Version 2017-07-29 beträgt die maximale Gültigkeitsdauer sieben Tage. Für Version 2017-07-29 oder höhere Versionen kann die maximale Gültigkeitsdauer eine beliebige positive Zahl sein. Mit -1 wird angegeben, dass die Nachricht nicht abläuft. Wird dieser Parameter ausgelassen, beträgt die Standardgültigkeitsdauer sieben Tage.

## <a name="next-steps"></a>Nächste Schritte

* [Erstellen eines Speicherkontos](../storage-create-storage-account.md?toc=%2fazure%2fstorage%2fqueues%2ftoc.json)
* [Erste Schritte mit Azure Queue Storage mit .NET](storage-dotnet-how-to-use-queues.md)
