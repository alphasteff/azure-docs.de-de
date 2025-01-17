---
title: Ablaufsteuerung des Unterhaltungslernmoduls – Microsoft Cognitive Services | Microsoft-Dokumentation
titleSuffix: Azure
description: Informationen zur Ablaufsteuerung des Unterhaltungslernmoduls.
services: cognitive-services
author: nitinme
manager: nolachar
ms.service: cognitive-services
ms.subservice: conversation-learner
ms.topic: article
ms.date: 04/30/2018
ms.author: nitinme
ROBOTS: NOINDEX
ms.openlocfilehash: 3ec839c1a930ffbe73989149360f1b02866a3c50
ms.sourcegitcommit: ad9120a73d5072aac478f33b4dad47bf63aa1aaa
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 08/01/2019
ms.locfileid: "68705311"
---
## <a name="control-flow"></a>Ablaufsteuerung

Dieses Dokument beschreibt die Ablaufsteuerung des Unterhaltungslernmoduls (Conversation Learner, CL), wie im Diagramm unten dargestellt.

![](media/controlflow.PNG)

1. Benutzer geben einen Begriff oder eine Formulierung in den Bot ein, z.B. „Wie ist das Wetter in Düsseldorf?“
1. CL übergibt die Benutzereingabe einem Machine Learning-Modell, das Entitäten extrahiert
   - Dieses Modell wird vom Unterhaltungslernmodul erstellt und auf www.luis.ai gehostet
1. Alle extrahierten Entitäten und der Eingabetext des Benutzers werden an die Rückrufmethode zur Entitätserkennung im Code des Bots übergeben.
    - Dieser Code kann Entitätswerte festlegen/löschen/bearbeiten
1. Das neuronale Netzwerk des Unterhaltungslernmoduls übernimmt dann die Ausgabe der Entitätsextrahierung und die Benutzereingabe und bewertet alle im Bot definierten Aktionen
   - In diesem Beispiel ist die Aktion mit der höchsten Wahrscheinlichkeit die Bereitstellung der Wettervorhersage:

     ![](media/controlflow_forecast.PNG)

1. Die ausgewählte Aktion erfordert in diesem Fall einen API-Aufruf, um die Wettervorhersage abzurufen. 
1. Diese API, die zuvor mithilfe der CL.AddCallback-Methode registriert wurde, wird dann aufgerufen.  Das Ergebnis dieser API wird dann als Nachricht an den Benutzer zurückgegeben – z.B. „Sonnig mit einer Höchsttemperatur von 19 Grad“.
1. Dann erfolgt ein Aufruf des neuronalen Netzwerks, um auf der Grundlage des vorherigen Schritts die nächste Aktion zu bestimmen.
1. Das neuronale Netzwerk sagt dann den nächsten Satz möglicher Aktionen vorher, und die ausgewählte Aktion wird dem Benutzer angezeigt, in diesem Fall „Haben Sie weitere Wünsche?“

## <a name="next-steps"></a>Nächste Schritte

> [!div class="nextstepaction"]
> [Einsetzen von Conversation Learner](./how-to-teach-cl.md)
