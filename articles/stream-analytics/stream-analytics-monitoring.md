---
title: Grundlegendes zur Auftragsüberwachung in Azure Stream Analytics
description: In diesem Artikel wird das Überwachen von Azure Stream Analytics-Aufträgen im Azure-Portal beschrieben.
services: stream-analytics
author: mamccrea
ms.author: mamccrea
ms.reviewer: jasonh
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 6/21/2018
ms.custom: seodec18
ms.openlocfilehash: a4fc6db300a24d483e88efe668645d2cdb62496b
ms.sourcegitcommit: 6a42dd4b746f3e6de69f7ad0107cc7ad654e39ae
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 07/07/2019
ms.locfileid: "67612275"
---
# <a name="understand-stream-analytics-job-monitoring-and-how-to-monitor-queries"></a>Grundlegendes zur Stream Analytics-Auftragsüberwachung und zum Überwachen von Abfragen

## <a name="introduction-the-monitor-page"></a>Einführung: Seite „Monitor“
Im Azure-Portal werden wichtige Leistungsmetriken angezeigt, die zum Überwachen der Leistung Ihrer Abfragen und Aufträge sowie für die Problembehandlung verwendet werden können. Navigieren Sie zum Anzeigen dieser Metriken zum Stream Analytics-Auftrag, dessen Metriken Sie interessieren, und zeigen Sie auf der Seite „Übersicht“ den Abschnitt **Überwachung** an.  

![Link zur Stream Analytics-Auftragsüberwachung](./media/stream-analytics-monitoring/02-stream-analytics-monitoring-block.png)

Das Fenster wird wie folgt angezeigt:

![Dashboard der Stream Analytics-Auftragsüberwachung](./media/stream-analytics-monitoring/01-stream-analytics-monitoring.png)  

## <a name="metrics-available-for-stream-analytics"></a>Verfügbare Metriken für Stream Analytics
| Metrik                 | Definition                               |
| ---------------------- | ---------------------------------------- |
| Eingabeereignisse im Rückstand       | Anzahl der Eingabeereignisse, die im Rückstand sind. Ein Wert ungleich Null für diese Metrik bedeutet, dass Ihr Auftrag nicht in der Lage ist, mit der Anzahl der eingehenden Ereignisse Schritt zu halten. Wenn dieser Wert langsam ansteigt oder konstant ungleich Null ist, sollten Sie Ihren Auftrag erweitern. Weitere Informationen finden Sie unter [Verstehen und Anpassen von Streamingeinheiten](stream-analytics-streaming-unit-consumption.md). |
| Konvertierungsfehler | Anzahl der Ausgabeereignisse, die nicht in das erwartete Ausgabeschema konvertiert werden konnten. Die Fehlerrichtlinie kann auf „Drop“ geändert werden, um Ereignisse zu löschen, bei denen dieses Szenario auftritt. |
| Frühe Eingabeereignisse       | Ereignisse, deren Anwendungszeitstempel mehr als fünf Minuten vor dem Zeitpunkt ihres Eintreffens liegt. |
| Fehlerhafte Funktionsanforderungen | Anzahl der fehlerhaften Aufrufe an die Azure Machine Learning-Funktion (falls vorhanden) |
| Funktionsereignisse        | Anzahl der an die Azure Machine Learning-Funktion gesendeten Ereignisse (falls vorhanden) |
| Funktionsanforderungen      | Anzahl der Aufrufe an die Azure Machine Learning-Funktion (falls vorhanden) |
| Eingabefehler bei Deserialisierung       | Anzahl der Eingabeereignisse, die nicht deserialisiert werden konnten.  |
| Eingabeereignisbytes      | Vom Stream Analytics-Auftrag erhaltene Datenmenge, in Byte. Kann verwendet werden, um sicherzustellen, dass Ereignisse an die Eingabequelle gesendet werden. |
| Eingabeereignisse           | Anzahl der Datensätze, die aus den Eingabeereignissen deserialisiert wurden. Diese Anzahl umfasst nicht eingehende Ereignisse, die zu Deserialisierungsfehlern führen. |
| Empfangene Eingabequellen       | Anzahl der vom Auftrag empfangenen Nachrichten. Für Event Hub besteht eine Nachricht aus einem einzelnen EventData. Für ein Blob ist eine Nachricht ein einzelnes Blob. Beachten Sie, dass die Eingabequellen vor der Deserialisierung gezählt werden. Bei Deserialisierungsfehlern können die Eingabequellen größer sein als die Eingabeereignisse. Andernfalls können sie kleiner als oder gleich den Eingabeereignissen sein, da jede Nachricht mehrere Ereignisse enthalten kann. |
| Ereignisse bei verspäteter Eingabe      | Ereignisse, die später als das konfigurierte Toleranzfenster für Eingangsverzögerung eingetroffen sind. Weitere Informationen finden Sie unter [Überlegungen zur Ereignisreihenfolge in Azure Stream Analytics](stream-analytics-out-of-order-and-late-events.md). |
| Ereignisse für falsche Reihenfolge    | Anzahl der Ereignisse, die in falscher Reihenfolge empfangen und anhand der Richtlinie für die Ereignissortierung entweder verworfen oder mit einem angepassten Zeitstempel versehen wurden. Dies kann von der Konfiguration der Einstellung „Toleranzfenster für Fehlordnung“ beeinflusst werden. |
| Ausgabeereignisse          | Vom Stream Analytics-Auftrag an das Ausgabeziel gesendete Datenmenge, ausgedrückt in der Anzahl von Ereignissen. |
| Laufzeitfehler         | Gesamtanzahl von Fehlern im Zusammenhang mit der Abfrageverarbeitung (mit Ausnahme von Fehlern, die beim Untersuchen von Ereignissen oder beim Ausgeben von Ergebnissen ermittelt werden) |
| Speichereinheitnutzung in %       | Die Auslastung der Streamingeinheiten, die einem Auftrag über seine Registerkarte „Skalieren“ zugewiesen sind. Sollte dieser Indikator 80 % erreichen oder darüber steigen, ist die Wahrscheinlichkeit hoch, dass die Ereignisverarbeitung verzögert wird oder keine Fortschritte mehr macht. |
| Wasserzeichenverzögerung       | Die maximale Wasserzeichenverzögerung für alle Partitionen aller Ausgaben im Auftrag. |

Sie können diese Metriken zum [Überwachen der Leistung des Stream Analytics-Auftrags](https://docs.microsoft.com/azure/stream-analytics/stream-analytics-set-up-alerts#scenarios-to-monitor) verwenden. 

## <a name="customizing-monitoring-in-the-azure-portal"></a>Anpassen der Überwachung im Azure-Portal
Sie können den Diagrammtyp, die angezeigten Metriken und den Uhrzeitbereich in den Einstellungen unter "Diagramm bearbeiten" anpassen. Weitere Informationen finden Sie unter [Anpassen der Überwachung](../monitoring-and-diagnostics/insights-how-to-customize-monitoring.md).

  ![Zeitdiagramm für die Stream Analytics-Abfrageüberwachung](./media/stream-analytics-monitoring/08-stream-analytics-monitoring.png)  


## <a name="latest-output"></a>Letzte Ausgabe
Ein weiterer interessanter Datenpunkt für die Überwachung Ihres Auftrags ist der Zeitpunkt der letzten Ausgabe, der auf der Übersichtsseite angezeigt wird.
Bei dieser Zeitangabe handelt es sich um den Anwendungszeitpunkt (d.h. den Zeitpunkt mit dem Zeitstempel aus den Ereignisdaten) der letzten Ausgabe Ihres Auftrags.

## <a name="get-help"></a>Hier erhalten Sie Hilfe
Um Hilfe zu erhalten, besuchen Sie unser [Azure Stream Analytics-Forum](https://social.msdn.microsoft.com/Forums/azure/home?forum=AzureStreamAnalytics)

## <a name="next-steps"></a>Nächste Schritte
* [Einführung in Azure Stream Analytics](stream-analytics-introduction.md)
* [Erste Schritte mit Azure Stream Analytics](stream-analytics-real-time-fraud-detection.md)
* [Skalieren von Azure Stream Analytics-Aufträgen](stream-analytics-scale-jobs.md)
* [Stream Analytics Query Language Reference (in englischer Sprache)](https://docs.microsoft.com/stream-analytics-query/stream-analytics-query-language-reference)
* [Referenz zur Azure Stream Analytics-Verwaltungs-REST-API](https://msdn.microsoft.com/library/azure/dn835031.aspx)
