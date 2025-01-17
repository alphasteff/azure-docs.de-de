---
title: Ausführen von Azure Kubernetes Service
titleSuffix: Text Analytics - Azure Cognitive Services
description: Stellen Sie die Textanalysecontainer mit dem Bild der Standpunktanalyse beim Azure Kubernetes Service bereit, und testen Sie sie in einem Webbrowser.
services: cognitive-services
author: IEvangelist
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: conceptual
ms.date: 06/21/2019
ms.author: dapine
ms.openlocfilehash: 290a01e7e478f718607c0550702474cd31979a63
ms.sourcegitcommit: b49431b29a53efaa5b82f9be0f8a714f668c38ab
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 07/22/2019
ms.locfileid: "68377414"
---
# <a name="deploy-a-sentiment-analysis-container-to-azure-kubernetes-service"></a>Bereitstellen eines Containers für die Standpunktanalyse für Azure Kubernetes Service

Erfahren Sie, wie Sie den [Textanalyse](https://docs.microsoft.com/azure/cognitive-services/text-analytics/how-tos/text-analytics-how-to-install-containers)-Container von Azure Cognitive Services mit dem Bild der Standpunktanalyse für Azure Kubernetes Services (AKS) bereitstellen. Dieses Verfahren zeigt das Erstellen einer Textanalyseressource, das Erstellen eines zugehörigen Standpunktanalysebilds und die Orchestrierung der beiden über einen Browser. Die Verwendung von Containern kann Ihre Aufmerksamkeit von der Verwaltung der Infrastruktur auf die Anwendungsentwicklung lenken.

## <a name="prerequisites"></a>Voraussetzungen

Für dieses Verfahren müssen mehrere Tools lokal installiert und ausgeführt werden. Verwenden Sie nicht Azure Cloud Shell. Sie benötigen Folgendes:

* Ein Azure-Abonnement. Wenn Sie kein Azure-Abonnement besitzen, können Sie ein [kostenloses Konto](https://azure.microsoft.com/free/) erstellen, bevor Sie beginnen.
* Einen Text-Editor, z.B. [Visual Studio Code](https://code.visualstudio.com/download).
* Die [Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest) muss installiert sein.
* Die [Kubernetes CLI](https://kubernetes.io/docs/tasks/tools/install-kubectl/) muss installiert sein.
* Eine Azure-Ressource mit dem korrekten Tarif. Nicht alle Tarife können mit diesem Container verwendet werden:
    * Eine **Azure-Textanalyse**-Ressource nur in den Tarifen „F0“ oder „Standard“.
    * Eine **Azure Cognitive Services**-Ressource im S0-Tarif.

[!INCLUDE [Create a Cognitive Services Text Analytics resource](../includes/create-text-analytics-resource.md)]

[!INCLUDE [Create a Text Analytics container on Azure Kubernetes Service (AKS)](../../containers/includes/create-aks-resource.md)]

## <a name="deploy-a-text-analytics-container-to-an-aks-cluster"></a>Bereitstellen eines Textanalyse-Containers in einem AKS-Cluster

1. Öffnen Sie die Azure CLI, und melden Sie sich bei Azure an.

    ```azurecli
    az login
    ```

1. Melden Sie sich bei dem AKS-Cluster an. Ersetzen Sie `your-cluster-name` und `your-resource-group` durch die entsprechenden Werte.

    ```azurecli
    az aks get-credentials -n your-cluster-name -g -your-resource-group
    ```

    Nachdem dieser Befehl ausgeführt wurde, wird eine Meldung ähnlich der folgenden ausgegeben:

    ```console
    Merged "your-cluster-name" as current context in /home/username/.kube/config
    ```

    > [!WARNING]
    > Wenn Sie über mehrere Abonnements in Ihrem Azure-Konto verfügen und der `az aks get-credentials` Befehl mit einem Fehler zurückgegeben wird, ist ein häufiges Problem, dass Sie das falsche Abonnement verwenden. Stellen Sie den Kontext Ihrer Azure CLI-Sitzung so ein, dass Sie das gleiche Abonnement verwenden, mit dem Sie die Ressourcen erstellt haben, und versuchen Sie es erneut.
    > ```azurecli
    >  az account set -s subscription-id
    > ```

1. Öffnen Sie den Text-Editor Ihrer Wahl. In diesem Beispiel wird Visual Studio Code verwendet.

    ```azurecli
    code .
    ```

1. Erstellen Sie im Text-Editor eine neue Datei namens _sentiment.yaml_, und fügen Sie die folgende YAML darin ein. Achten Sie darauf, dass Sie `billing/value` und `apikey/value` durch Ihre eigenen Werte ersetzen.

    ```yaml
    apiVersion: apps/v1beta1
    kind: Deployment
    metadata:
      name: sentiment
    spec:
      template:
        metadata:
          labels:
            app: sentiment-app
        spec:
          containers:
          - name: sentiment
            image: mcr.microsoft.com/azure-cognitive-services/sentiment
            ports:
            - containerPort: 5000
            env:
            - name: EULA
              value: "accept"
            - name: billing
              value: # < Your endpoint >
            - name: apikey
              value: # < Your API Key >
     
    --- 
    apiVersion: v1
    kind: Service
    metadata:
      name: sentiment
    spec:
      type: LoadBalancer
      ports:
      - port: 5000
      selector:
        app: sentiment-app
    ```

1. Speichern Sie die Datei, und schließen Sie den Text-Editor.
1. Führen Sie den Kubernetes-Befehl `apply` mit _sentiment.yaml_ als Ziel aus:

    ```console
    kuberctl apply -f sentiment.yaml
    ```

    Nachdem der Befehl die Bereitstellungskonfiguration erfolgreich angewendet hat, wird eine Meldung ähnlich der folgenden Ausgabe angezeigt:

    ```
    deployment.apps "sentiment" created
    service "sentiment" created
    ```
1. Überprüfen Sie, ob der Pod bereitgestellt wurde:

    ```console
    kubectl get pods
    ```

    Die Ausgabe für den Ausführungsstatus des Pods:

    ```
    NAME                         READY     STATUS    RESTARTS   AGE
    sentiment-5c9ccdf575-mf6k5   1/1       Running   0          1m
    ```

1. Überprüfen Sie, ob der Dienst verfügbar ist, und rufen Sie die IP-Adresse ab.

    ```console
    kubectl get services
    ```

    Die Ausgabe für den Ausführungsstatus des _sentiment_-Diensts im POD:

    ```
    NAME         TYPE           CLUSTER-IP    EXTERNAL-IP      PORT(S)          AGE
    kubernetes   ClusterIP      10.0.0.1      <none>           443/TCP          2m
    sentiment    LoadBalancer   10.0.100.64   168.61.156.180   5000:31234/TCP   2m
    ```

[!INCLUDE [Verify the sentiment analysis container instance](../includes/verify-sentiment-analysis-container.md)]

## <a name="next-steps"></a>Nächste Schritte

* Verwenden weiterer [Cognitive Services-Container](../../cognitive-services-container-support.md)
* Verwenden des [Text Analytics Connected Service](../vs-text-connected-service.md)
