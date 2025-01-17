---
title: Cognitive Services und maschinelles Lernen
titlesuffix: Azure Cognitive Services
description: Erfahren Sie, wo Azure Cognitive Services mit anderen Azure-Angeboten für maschinelles Lernen kompatibel ist.
services: cognitive-services
author: diberry
manager: nitinme
ms.service: cognitive-services
ms.topic: conceptual
ms.date: 07/23/2019
ms.author: diberry
ms.openlocfilehash: 15faacccfa1e0f7dd759f719c02e106918d735bb
ms.sourcegitcommit: c556477e031f8f82022a8638ca2aec32e79f6fd9
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 07/23/2019
ms.locfileid: "68414689"
---
# <a name="cognitive-services-and-machine-learning"></a>Cognitive Services und maschinelles Lernen

Cognitive Services bietet Funktionen für maschinelles Lernen, mit denen allgemeine Probleme gelöst werden können, etwa ein Analysieren von Text auf emotionale Stimmungslagen oder Analysieren von Bildern, um Objekte oder Gesichter zu erkennen. Sie benötigen keine speziellen Kenntnisse zu maschinellem Lernen oder Data Science, um diese Dienste zu verwenden. 

[Cognitive Services](welcome.md) ist eine Gruppe von Diensten, die jeweils unterschiedliche generalisierte Vorhersagefunktionen unterstützen. Die Dienste sind in verschiedene Kategorien eingeteilt, anhand denen Sie den richtigen Dienst finden können. 

|Dienstkategorie|Zweck|
|--|--|
|[Entscheidung](https://azure.microsoft.com/services/cognitive-services/directory/decision/)|Erstellen Sie Apps, die Empfehlungen geben, um fundiert und effizient Entscheidungen treffen zu können.|
|[Sprache](https://azure.microsoft.com/services/cognitive-services/directory/lang/)|Ermöglichen Sie es Ihren Apps, natürliche Sprache mit vordefinierten Skripts zu verarbeiten, Stimmungen auszuwerten sowie zu trainieren, wie zu erkennen ist, was Benutzer wünschen.|
|[Suchen,](https://azure.microsoft.com/services/cognitive-services/directory/search/)|Fügen Sie Bing-Suche-APIs zu Ihren Apps hinzu, und profitieren Sie von der Möglichkeit, Milliarden von Webseiten, Bildern, Videos und Nachrichten mit einem einzigen API-Aufruf zu durchsuchen.|
|[Spracheingabe](https://azure.microsoft.com/services/cognitive-services/directory/speech/)|Konvertieren Sie Sprache in Text und Text in natürlich klingende Sprache. Übersetzen Sie von einer Sprache in eine andere, und aktivieren Sie Sprecherüberprüfung und -erkennung.|
|[Bildanalyse](https://azure.microsoft.com/services/cognitive-services/directory/vision/)|Erkennen, identifizieren, untertiteln, indizieren und verwalten Sie Ihre Bilder, Videos und Freihandschriftinhalte.|
||||

Verwenden Sie Cognitive Services, wenn Folgendes zutrifft:

* Sie können eine verallgemeinerte Lösung verwenden.
* Sie greifen aus einer REST-API oder einem SDK für Programmierung auf eine Lösung zu. 

Verwenden Sie eine andere Lösung für maschinelles Lernen, wenn Folgendes zutrifft:

* Sie müssen den Algorithmus auswählen, und Sie müssen für das Trainieren sehr spezielle Daten verwenden.

## <a name="what-is-machine-learning"></a>Was ist Machine Learning?

Machine Learning (maschinelles Lernen) ist ein Konzept, bei dem Sie Daten und einen Algorithmus zusammenführen, um einen bestimmten Aspekt zu lösen. Sobald der Algorithmus mit den jeweiligen Daten trainiert wurde, ist die Ausgabe ein Modell, das Sie mit anderen Daten erneut verwenden können. Das trainierte Modell bietet Erkenntnisse auf Basis der neuen Daten. 

Der Prozess zum Erstellen eines Machine Learning-Systems erfordert einige Kenntnisse zu Machine Learning oder Data Science.

Machine Learning wird über [Azure Machine Learning-Produkte und -Dienste](https://docs.microsoft.com/azure/architecture/data-guide/technology-choices/data-science-and-machine-learning?context=azure/machine-learning/studio/context/ml-context) bereitgestellt.

## <a name="what-is-a-cognitive-service"></a>Was ist ein Cognitive Service (kognitiver Dienst)?

Ein Cognitive Service stellt einige oder alle Komponenten einer Machine Learning-Lösung bereit: Daten, Algorithmus und trainiertes Modell. Zur Nutzung dieser Dienste sind allgemeine Kenntnisse über Ihre Daten erforderlich, ohne dass Sie Erfahrung mit maschinellem Lernen oder Data Science benötigen. Diese Dienste stellen sowohl REST-API(s) als auch sprachbasierte SDKs bereit. Daher müssen Sie über Programmiersprachenkenntnisse verfügen, um die Dienste verwenden zu können.

## <a name="how-are-cognitive-services-and-azure-machine-learning-aml-similar"></a>An welchen Stellen ähneln sich ein Cognitive Service und Azure Machine Learning?

Beide haben als Endziel das Anwenden von künstlicher Intelligenz (KI), um den Geschäftsbetrieb zu verbessern, obwohl sie sich in der Art und Weise unterscheiden, wie sie dies im jeweiligen Angebot bereitstellen. 

Im Grundsatz sind die Zielgruppen unterschiedlich:

* Cognitive Services sind für Entwickler vorgesehen, die keine Machine Learning-Erfahrung haben.
* Azure Machine Learning ist auf Data Scientists zugeschnitten. 

## <a name="how-is-a-cognitive-service-different-from-machine-learning"></a>Wie unterscheidet sich ein Cognitive Service von Machine Learning?

Ein Cognitive Service stellt Ihnen ein trainiertes Modell bereit. Dadurch werden Daten und ein Algorithmus verknüpft, die über eine REST-API oder ein SDK verfügbar sind. Abhängig von Ihrem Szenario können Sie diesen Dienst innerhalb von Minuten implementieren.  Ein Cognitive Service bietet Antworten auf allgemeine Probleme, etwa Schlüsselausdrücke in Texten oder Elementerkennung in Bildern. 

Machine Learning ist ein Prozess, für dessen erfolgreiche Implementierung üblicherweise ein längerer Zeitraum erforderlich ist. Diese Zeit wird für Datensammlung, Bereinigung, Transformation, Algorithmusauswahl, Modelltraining und Bereitstellung aufgewendet, um die gleiche Funktionalität zu erreichen, die von einem Cognitive Service bereitgestellt wird. Mit Machine Learning können Sie Antworten auf alle Arten von Problemen bereitstellen, einschließlich besonders spezieller Probleme. Diese mit Machine Learning zu lösenden Probleme erfordern Vertrautheit mit mindestens einem der folgenden Punkte: Thema, maschinelles Lernen, Data Science.

## <a name="what-kind-of-data-do-you-have"></a>Welche Art von Daten haben Sie?

Für Cognitive Services (als Gruppe von Diensten) können keine, einige oder alle benutzerdefinierten Daten für das trainierte Modell erforderlich sein. 

### <a name="no-additional-training-data-required"></a>Keine zusätzlichen Trainingsdaten erforderlich

Dienste, die ein vollständig trainiertes Modell bereitstellen, können als _Blackbox_ behandelt werden. Sie müssen weder wissen, wie die Dienste funktionieren, noch, welche Daten verwendet wurden, um sie zu trainieren. Sie lassen Ihre Daten in ein vollständig trainiertes Modell einfließen, um eine Vorhersage zu erhalten. 

### <a name="some-or-all-training-data-required"></a>Einige oder alle Trainingsdaten erforderlich

Einige Dienste ermöglichen es Ihnen, Ihre eigenen Daten einzubringen und dann ein Modell zu trainieren. Dadurch können Sie das Modell, für das die Daten und der Algorithmus des Diensts verwendet werden, mit ihren eigenen Daten erweitern. Die Ausgabe entspricht Ihren Anforderungen. Wenn Sie Ihre eigenen Daten einbringen, müssen Sie die Daten möglicherweise auf eine Weise markieren, die speziell für den Dienst gilt. Wenn Sie beispielsweise ein Modell zum Erkennen von Blumen trainieren, können Sie einen Katalog mit Blumenbildern zusammen mit der Position der Blume in jedem Bild bereitstellen, um das Modell zu trainieren. 

Ein Dienst kann Ihnen das Bereitstellen von Daten _ermöglichen_, um seine eigenen Daten zu verbessern. Ein Dienst kann von Ihnen _verlangen_, dass Sie Daten bereitstellen. 

### <a name="real-time-or-near-real-time-data-required"></a>Daten in Echtzeit oder nahezu in Echtzeit erforderlich

Ein Dienst benötigt möglicherweise Daten in Echtzeit oder nahezu in Echtzeit, um ein effektives Modell zu erstellen. Ein solcher Dienst verarbeiten beträchtliche Mengen von Modelldaten. 

## <a name="service-requirements-for-the-data-model"></a>Dienstanforderungen für das Datenmodell

In den folgenden Punkten ist jeder Dienst entsprechend der Art von Daten kategorisiert, die er zulässt oder erfordert.

|Cognitive Service|Keine Trainingsdaten erforderlich|Sie stellen einige oder alle Trainingsdaten bereit|Datensammlung in Echtzeit oder nahezu in Echtzeit|
|--|--|--|--|
|[Anomalieerkennung](./Anomaly-Detector/overview.md)|x|x|x|
|Bing-Suche |x|||
|[Maschinelles Sehen](./Computer-vision/Home.md)|x|||
|[Content Moderator](./Content-Moderator/overview.md)|x||x|
|[Custom Vision](./Custom-Vision-Service/home.md)||x||
|[Gesichtserkennung](./Face/Overview.md)|x|x||
|[Formularerkennung](./form-recognizer/overview.md)||x||
|[Plastischer Reader](./immersive-reader/overview.md)|x|||
|[Freihanderkennung](./Ink-recognizer/overview.md)|x|x||
|[Language Understanding (LUIS)](./LUIS/what-is-luis.md)||x||
|[Personalisierung](./personalizer/what-is-personalizer.md)|x*|x*|x|
|[QnA Maker](./QnAMaker/Overview/overview.md)||x||
|[Sprechererkennung](./speaker-recognition/home.md)||x||
|[Text-zu-Sprache (Text-to-Speech, TTS)](speech-service/text-to-speech.md)|x|x||
|[Spracherkennung (Speech-to-Text, STT)](/speech-service/speech-to-text.md)|x|x||
|[Sprachübersetzung](speech-service/speech-translation.md)|x|||
|[Textanalyse](./text-analytics/overview.md)|x|||
|[Textübersetzung](./translator/translator-info-overview.md)|x|||
|[Textübersetzung – Custom Translator]()||x||

*Personalisierung benötigt für ein Auswerten Ihrer Richtlinien und Daten nur Trainingsdaten, die vom Dienst erfasst wurden (weil der Dienst in Echtzeit funktioniert). Personalisierung benötigt für ein Vorab- oder Stapelverarbeitungstraining keine großen Datasets mit bereits erfassten Daten. 

## <a name="where-can-you-use-cognitive-services"></a>Wo können Sie Cognitive Services verwenden?
 
Die Dienste können in jeder Anwendung verwendet werden, in der REST-API- oder SDK-Aufrufe ausgeführt werden können. Beispiele für Anwendungen sind Websites, Bots, Anwendungen für virtuelle Realität oder Mixed reality sowie Desktop- und mobile Anwendungen. 

## <a name="how-is-cognitive-search-related-to-cognitive-services"></a>Wie hängen kognitive Suche und Cognitive Services zusammen?

[Azure Search](../search/search-what-is-azure-search.md) verwendet Cognitive Services, um diese Funktionalität bereitzustellen. Cognitive Services wird in Azure Search über [integrierte Fähigkeiten](../search/cognitive-search-predefined-skills.md) verfügbar gemacht, die einzelne APIs einschließen. Sie können eine kostenlose Ressource für exemplarische Vorgehensweisen verwenden, aber planen Sie für größere Volumen das Erstellen und Zuordnen einer [abrechenbaren Ressource](../search/cognitive-search-attach-cognitive-services.md) ein.

## <a name="how-can-you-use-cognitive-services"></a>Wie können Sie Cognitive Services verwenden?

Jeder Dienst stellt Informationen zu Ihren Daten bereit. Sie können Dienste kombinieren, um Lösungen zu verketten, z. B. Konvertieren von Sprache (Audio) in Text, Übersetzen des Texts in viele Sprachen und anschließendes Verwenden der übersetzten Sprachen, um Antworten aus einer Wissensdatenbank zu erhalten. Obwohl Cognitive Services genutzt werden können, intelligente Lösungen auf deren Basis zu erstellen, können sie auch mit herkömmlichen Machine Learning-Projekten kombiniert werden, um Modelle zu ergänzen oder den Entwicklungsprozess zu beschleunigen. 

Folgende Cognitive Services stellen exportierte Modelle für andere Machine Learning-Tools bereit:

|Cognitive Service|Modellinformationen|
|--|--|
|[Custom Vision](./custom-vision-service/home.md)|[Exportieren](./Custom-Vision-Service/export-model-python.md) für TensorFlow für Android, CoreML für iOS11, ONNX für Windows ML|


## <a name="next-steps"></a>Nächste Schritte

* Erstellen Sie Ihr Cognitive Service-Konto im [Azure-Portal](cognitive-services-apis-create-account.md) oder mit der [Azure-Befehlszeilenschnittstelle](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account-cli).
* Erfahren Sie, wie Sie sich bei einem Cognitive Service [authentifizieren](authentication.md).
* Verwenden Sie [Diagnoseprotokollierung](diagnostic-logging.md) zur Problemerkennung und zum Debuggen. 
* Stellen Sie einen Cognitive Service in einem Docker-[Container](cognitive-services-container-support.md) bereit.
* Bleiben Sie mit [Dienstupdates](https://azure.microsoft.com/updates/?product=cognitive-services) auf dem neuesten Stand.
