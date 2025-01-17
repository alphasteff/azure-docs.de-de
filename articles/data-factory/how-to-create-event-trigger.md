---
title: Erstellen von ereignisbasierten Triggern in Azure Data Factory | Microsoft-Dokumentation
description: Erfahren Sie, wie in Azure Data Factory ein Trigger erstellt wird, der eine Pipeline als Reaktion auf ein Ereignis ausführt.
services: data-factory
documentationcenter: ''
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.topic: conceptual
ms.date: 10/18/2018
author: sharonlo101
ms.author: shlo
manager: craigg
ms.openlocfilehash: bf4dc55d0ec17daf4c611563dd7aee3a06aa192b
ms.sourcegitcommit: 04ec7b5fa7a92a4eb72fca6c6cb617be35d30d0c
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 07/22/2019
ms.locfileid: "68384706"
---
# <a name="create-a-trigger-that-runs-a-pipeline-in-response-to-an-event"></a>Erstellen eines Triggers, der eine Pipeline als Reaktion auf ein Ereignis ausführt

Dieser Artikel beschreibt die ereignisbasierten Trigger, die Sie in Ihren Data Factory-Pipelines erstellen können.

Die ereignisgesteuerte Architektur (EDA) ist ein allgemeines Datenintegrationsmuster, das die Produktion, Erkennung, Verbrauch und Reaktion auf Ereignisse beinhaltet. In Datenintegrationsszenarios ist es häufig notwendig, dass Data Factory-Kunden Pipelines auf der Grundlage von Ereignissen auslösen, z. B. wenn Dateien zu einem Azure Storage-Konto hinzugefügt oder aus diesem gelöscht werden. Data Factory und [Azure Event Grid](https://azure.microsoft.com/services/event-grid/) sind jetzt integriert, sodass Sie bei einem Ereignis Pipelines auslösen können.

Das folgende Video enthält eine zehnminütige Einführung und Demonstration dieses Features:

> [!VIDEO https://channel9.msdn.com/Shows/Azure-Friday/Event-based-data-integration-with-Azure-Data-Factory/player]


> [!NOTE]
> Die in diesem Artikel beschriebene Integration basiert auf [Azure Event Grid](https://azure.microsoft.com/services/event-grid/). Stellen Sie sicher, dass Ihr Abonnement für den Event Grid-Ressourcenanbieter registriert ist. Weitere Informationen finden Sie unter [Ressourcenanbieter und -typen](../azure-resource-manager/resource-manager-supported-services.md#azure-portal).

## <a name="data-factory-ui"></a>Data Factory-Benutzeroberfläche

In diesem Abschnitt wird gezeigt, wie Sie über die Benutzeroberfläche von Azure Data Factory einen Trigger erstellen können.

1. Navigieren Sie zum **Bereich für die Erstellung**.

2. Klicken Sie in der linken unteren Ecke auf **Trigger**.

3. Klicken Sie auf **+ Neu**. Dann wird die Seitennavigationsleiste für die Triggererstellung geöffnet.

4. Wählen Sie als Triggertyp die Option **Ereignis** aus.

![Erstellen eines neuen Triggers](media/how-to-create-event-trigger/event-based-trigger-image1.png)

5. Wählen Sie aus der Dropdownliste für die Azure-Abonnements Ihr Storage-Konto aus, oder geben Sie die Ressourcen-ID des Storage-Kontos ein. Wählen Sie den Container aus, in dem die Ereignisse auftreten sollen. Die Containerauswahl ist optional, aber beachten Sie, dass die Auswahl aller Container zu einer hohen Anzahl von Ereignissen führen kann.

   > [!NOTE]
   > Der Trigger unterstützt derzeit nur Storage-Konten der Version 2 (Universell).

   > [!NOTE]
   > Aufgrund einer Einschränkung von Azure Event Grid unterstützt Azure Data Factory nur maximal 500 Trigger pro Storage-Konto.

6. Sie können die Eigenschaften **Blob path begins with** (Blobpfad beginnt mit) und **Blob path ends with** (Blobpfad endet mit) verwenden, um die Container, Ordner und Blobnamen anzugeben, für die Ereignisse empfangen werden sollen. Es muss mindestens eine dieser Eigenschaften für den Trigger definiert werden. Für die Eigenschaften **Blob-Pfad beginnt mit** und **Blob-Pfad endet mit** können Sie eine Vielzahl von Mustern verwenden, wie in den Beispielen im weiteren Verlauf dieses Artikels dargestellt.

    * **Blob path begins with** (Blobpfad beginnt mit): Der Blobpfad muss mit einem Ordnerpfad beginnen. Beispielsweise können die folgenden Werte verwendet werden: `2018/` und `2018/april/shoes.csv`. Dieses Feld kann nicht ausgewählt werden, wenn kein Container ausgewählt ist.
    * **Blob path ends with** (Blobpfad endet mit): Der Blobpfad muss auf einem Dateinamen oder einer Erweiterung enden. Beispielsweise können die folgenden Werte verwendet werden: `shoes.csv` und `.csv`. Die Angabe von Container- und Ordnernamen ist optional. Wenn Sie sich für die Angabe entscheiden, müssen die Namen aber durch ein `/blobs/`-Segment voneinander getrennt werden. Beispielsweise kann ein Container mit dem Namen „Bestellungen“ den Wert `/orders/blobs/2018/april/shoes.csv` haben. Wenn Sie einen Ordner in einem beliebigen Container angeben möchten, lassen Sie das vorangestellte Zeichen „/“ aus. Beispielsweise löst `april/shoes.csv` ein Ereignis für jede Datei mit dem Namen `shoes.csv` aus, die sich in einem beliebigen Container in einem Ordner mit dem Namen „April“ befindet. 

7. Wählen Sie aus, ob der Trigger auf ein **Blob created**-Ereignis (Blob erstellt), auf ein **Blob deleted**-Ereignis (Blob gelöscht) oder auf beides reagieren soll. Jedes Ereignis löst die Data Factory-Pipelines am angegebenen Speicherort aus, die dem Trigger zugeordnet sind.

    ![Konfigurieren des Ereignistriggers](media/how-to-create-event-trigger/event-based-trigger-image2.png)

8. Wenn Sie Ihren Trigger konfiguriert haben, klicken Sie auf **Weiter > Datenvorschau**. Dieser Anzeige können Sie die vorhandenen Blobs entnehmen, die ihrer Triggerkonfiguration entsprechen. Vergewissern Sie sich, dass Sie genaue Filter ausgewählt haben. Wenn Sie Filter auswählen, die zu weit gefasst sind, können diese auf eine große Anzahl an erstellten bzw. gelöschten Dateien zutreffen. Dafür können höhere Kosten anfallen. Sobald Sie Ihre Filterbedingungen überprüft haben, klicken Sie auf **Fertig stellen**.

    ![Datenvorschau für den Trigger](media/how-to-create-event-trigger/event-based-trigger-image3.png)

9. Sie können eine Pipeline an diesen Trigger anfügen, indem Sie zum Bereich „Pipeline“ navigieren und erst auf **Trigger hinzufügen** und dann auf **New/Edit** (Neu/Bearbeiten) klicken. Wenn die Seitennavigationsleiste angezeigt wird, klicken Sie auf die Dropdownliste **Choose trigger...** (Trigger auswählen...), und wählen Sie den von Ihnen erstellten Trigger aus. Klicken Sie auf **Weiter: Datenvorschau**, um zu überprüfen, ob die Konfiguration stimmt, und klicken Sie anschließend auf **Weiter**, um die Datenvorschau zu überprüfen.

10. Wenn Ihre Pipeline über Parameter verfügt, können Sie diese auf der Seitennavigationsleiste „Trigger run parameter“ (Parameter für die Triggerausführung) angeben. Der Trigger erfasst den Ordnerpfad und den Dateinamen des Blobs in den Eigenschaften `@triggerBody().folderPath` und `@triggerBody().fileName`. Um die Werte dieser Eigenschaften in einer Pipeline zu verwenden, müssen Sie die Eigenschaften Pipelineparametern zuordnen. Nach der Zuordnung der Eigenschaften zu Parametern können Sie über den Ausdruck `@pipeline().parameters.parameterName` auf die vom Trigger erfassten Werte in der gesamten Pipeline zugreifen. Wenn Sie fertig sind, klicken Sie auf **Fertig stellen**.

    ![Zuordnung von Eigenschaften zu Pipelineparametern](media/how-to-create-event-trigger/event-based-trigger-image4.png)

Im vorherigen Beispiel wurde der Trigger so konfiguriert, dass er ausgelöst wird, wenn im Container „sample-data“ (Beispieldaten) ein Blobpfad mit der Erweiterung „.csv“ im Ordner „event-testing“ (Ereignistest) erstellt wird. Die Eigenschaften **folderPath** and **fileName** erfassen den Speicherort des neuen Blobs. Wenn z. B. „MoviesDB.csv“ zum Pad „sample-data/event-testing“ (Beispieldaten/Ereignistest) hinzugefügt wird, hat `@triggerBody().folderPath` den Wert `sample-data/event-testing` und `@triggerBody().fileName` den Wert `moviesDB.csv`. Diese Werte werden im Beispiel den Pipelineparametern `sourceFolder` und `sourceFile` zugeordnet, die in der gesamten Pipeline jeweils als `@pipeline().parameters.sourceFolder` bzw. `@pipeline().parameters.sourceFile` verwendet werden können.

## <a name="json-schema"></a>JSON-Schema

Die folgende Tabelle enthält eine Übersicht über die Schemaelemente, die mit ereignisbasierten Triggern verknüpft sind:

| **JSON-Element** | **Beschreibung** | **Typ** | **Zulässige Werte** | **Erforderlich** |
| ---------------- | --------------- | -------- | ------------------ | ------------ |
| **scope** | Die Ressourcen-ID von Azure Resource Manager im Speicherkonto. | Zeichenfolge | Azure Resource Manager-ID | Ja |
| **events** | Der Ereignistyp, die diesen Trigger auslöst. | Array    | Microsoft.Storage.BlobCreated, Microsoft.Storage.BlobDeleted | Ja (beliebige Kombination dieser Werte). |
| **blobPathBeginsWith** | Der Blobpfad muss mit dem angegebenen Muster beginnen, damit der Trigger ausgelöst wird. `/records/blobs/december/` löst den Trigger beispielsweise nur für Blobs im Ordner `december` unter dem Container `records` aus. | Zeichenfolge   | | Geben Sie einen Wert für mindestens eine der folgenden Eigenschaften an: `blobPathBeginsWith` oder `blobPathEndsWith`. |
| **blobPathEndsWith** | Der Blobpfad muss mit dem angegebenen Muster enden, damit der Trigger ausgelöst wird. `december/boxes.csv` löst den Trigger beispielsweise nur für Blobs namens `boxes` aus, die sich in einem Ordner namens `december` befinden. | Zeichenfolge   | | Geben Sie einen Wert für mindestens eine der folgenden Eigenschaften an: `blobPathBeginsWith` oder `blobPathEndsWith`. |

## <a name="examples-of-event-based-triggers"></a>Beispiele für ereignisbasierte Trigger

Dieser Abschnitt enthält Beispiele für die Einstellungen für ereignisbasierte Trigger.

> [!IMPORTANT]
> Sie müssen das Segment `/blobs/` des Pfads wie in den folgenden Beispielen gezeigt immer dann einbeziehen, wenn Sie Container und Ordner, Container und Datei oder Container, Ordner und Datei angeben. Für **blobPathBeginsWith** fügt die Benutzeroberfläche für Data Factory automatisch das Segment `/blobs/` zwischen dem Ordner und dem Containernamen zum JSON-Code des Auslösers ein.

| Eigenschaft | Beispiel | BESCHREIBUNG |
|---|---|---|
| **Blobpfad beginnt mit** | `/containername/` | Empfängt Ereignisse für ein beliebiges Blob im Container. |
| **Blobpfad beginnt mit** | `/containername/blobs/foldername/` | Empfängt Ereignisse für ein beliebiges Blob im Container `containername` und Ordner `foldername`. |
| **Blobpfad beginnt mit** | `/containername/blobs/foldername/subfoldername/` | Sie können auch auf einen Unterordner verweisen. |
| **Blobpfad beginnt mit** | `/containername/blobs/foldername/file.txt` | Empfängt Ereignisse für ein Blob namens `file.txt` im Ordner `foldername` unter dem Container `containername`. |
| **Blobpfad endet mit** | `file.txt` | Empfängt Ereignisse für ein Blob namens `file.txt` unter einem beliebigen Pfad. |
| **Blobpfad endet mit** | `/containername/blobs/file.txt` | Empfängt Ereignisse für ein Blob namens `file.txt` unter dem Container `containername`. |
| **Blobpfad endet mit** | `foldername/file.txt` | Empfängt Ereignisse für ein Blob namens `file.txt` im Ordner `foldername` unter einem beliebigen Container. |

## <a name="next-steps"></a>Nächste Schritte
Detaillierte Informationen zu Triggern finden Sie unter [Pipelineausführung und -trigger](concepts-pipeline-execution-triggers.md#triggers).
