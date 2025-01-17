---
title: Verwenden von Azure Dev Spaces für die Interaktion mit Windows-Containern
titleSuffix: Azure Dev Spaces
services: azure-dev-spaces
ms.service: azure-dev-spaces
author: zr-msft
ms.author: zarhoads
ms.date: 07/25/2019
ms.topic: conceptual
description: Erfahren Sie, wie Sie Azure Dev Spaces für einen vorhandenen Cluster mit Windows-Containern ausführen.
keywords: Azure Dev Spaces, Dev Spaces, Docker, Kubernetes, Azure, AKS, Azure Kubernetes Service, Container, Windows-Container
ms.openlocfilehash: 2110636b331f0cf4e74c77f41726ead5bf80a64f
ms.sourcegitcommit: a0b37e18b8823025e64427c26fae9fb7a3fe355a
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 07/25/2019
ms.locfileid: "68501596"
---
# <a name="use-azure-dev-spaces-to-interact-with-windows-containers"></a>Verwenden von Azure Dev Spaces für die Interaktion mit Windows-Containern

Sie können Azure Dev Spaces für neue und vorhandene Kubernetes-Namespaces aktivieren. Azure Dev Spaces führt Dienste aus, die in Linux-Containern ausgeführt werden, und instrumentiert diese. Diese Dienste können auch mit Anwendungen interagieren, die in Windows-Containern im selben Namespace ausgeführt werden. In diesem Artikel erfahren Sie, wie Sie mithilfe von Azure Dev Spaces Dienste in einem Namespace mit vorhandenen Windows-Containern ausführen.

## <a name="set-up-your-cluster"></a>Einrichten Ihres Clusters

In diesem Artikel wird davon ausgegangen, dass Sie bereits über einen Cluster mit Linux- und Windows-Knotenpools verfügen. Wenn Sie einen Cluster mit Linux- und Windows-Knotenpools erstellen müssen, finden Sie [hier][windows-container-cli] entsprechende Anweisungen.

Stellen Sie unter Verwendung des Kubernetes-Befehlszeilenclients ([kubectl][kubectl]) eine Verbindung mit Ihrem Cluster her. Mit dem Befehl [az aks get-credentials][az-aks-get-credentials] können Sie `kubectl` für die Verbindungsherstellung mit Ihrem Kubernetes-Cluster konfigurieren. Mit diesem Befehl werden die Anmeldeinformationen heruntergeladen, und die Kubernetes-Befehlszeilenschnittstelle wird für deren Verwendung konfiguriert.

```azurecli-interactive
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
```

Verwenden Sie zum Überprüfen der Verbindung mit Ihrem Cluster den Befehl [kubectl get][kubectl-get], um eine Liste der Clusterknoten zu erhalten.

```azurecli-interactive
kubectl get nodes
```

Die folgende Beispielausgabe zeigt einen Cluster mit einem Windows- und einem Linux-Knoten. Stellen Sie sicher, dass der Status für jeden Knoten *Bereit* lautet, bevor Sie fortfahren.

```console
NAME                                STATUS   ROLES   AGE    VERSION
aks-nodepool1-12345678-vmssfedcba   Ready    agent   13m    v1.14.1
aksnpwin987654                      Ready    agent   108s   v1.14.1
```

Wenden Sie einen [Taint][using-taints] auf die Windows-Knoten an. Der Taint auf Ihren Windows-Knoten verhindert, dass Azure Dev Spaces die Ausführung von Linux-Containern auf Ihren Windows-Knoten plant. Mit dem folgenden Beispielbefehl wird ein Taint auf den Windows-Knoten *aksnpwin987654* aus dem vorherigen Beispiel angewendet.

```azurecli-interactive
kubectl taint node aksnpwin987654 sku=win-node:NoSchedule
```

## <a name="run-your-windows-service"></a>Ausführen Ihres Windows-Diensts

Führen Sie den Windows-Dienst in Ihrem AKS-Cluster aus, und vergewissern Sie sich, dass der Status *Wird ausgeführt* lautet. In diesem Artikel wird anhand einer [Beispielanwendung][sample-application] die Ausführung eines Windows- und eines Linux-Diensts in Ihrem Cluster veranschaulicht.

Klonen Sie die Beispielanwendung von GitHub, und navigieren Sie in das Verzeichnis `dev-spaces/samples/existingWindowsBackend/mywebapi-windows`:

```console
git clone https://github.com/Azure/dev-spaces
cd dev-spaces/samples/existingWindowsBackend/mywebapi-windows
```

Die Beispielanwendung verwendet [Helm][helm-installed], um den Windows-Dienst auf Ihrem Cluster auszuführen. Installieren Sie Helm in Ihrem Cluster, und erteilen Sie ihm die entsprechenden Berechtigungen:

```console
helm init --wait
kubectl create serviceaccount --namespace kube-system tiller
kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller
kubectl patch deploy --namespace kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}'
``` 

Navigieren Sie zum Verzeichnis `charts`, und führen Sie den Windows-Dienst aus:

```console
cd charts/
helm install . --namespace dev
```

Der oben stehende Befehl verwendet Helm, um den Windows-Dienst im Namespace *dev* auszuführen. Wenn kein Namespace mit dem Namen *dev* vorhanden ist, wird er erstellt.

Überprüfen Sie mit dem Befehl `kubectl get pods`, ob der Windows-Dienst in Ihrem Cluster ausgeführt wird. 

```console
$ kubectl get pods --namespace dev --watch
NAME                     READY   STATUS              RESTARTS   AGE
myapi-4b9667d123-1a2b3   0/1     ContainerCreating   0          47s
...
myapi-4b9667d123-1a2b3   1/1     Running             0          98s
```

## <a name="enable-azure-dev-spaces"></a>Aktivieren von Azure Dev Spaces

Aktivieren Sie Azure Dev Spaces in demselben Namespace, den Sie zum Ausführen des Windows-Diensts verwendet haben. Der folgende Befehl aktiviert Azure Dev Spaces im Namespace *dev*:

```console
az aks use-dev-spaces -g myResourceGroup -n myAKSCluster --space dev --yes
```

## <a name="update-your-windows-service-for-dev-spaces"></a>Aktualisieren Ihres Windows-Diensts für Azure Dev Spaces

Wenn Sie Azure Dev Spaces für einen vorhandenen Namespace mit Containern aktivieren, die bereits ausgeführt werden, testet und instrumentiert Azure Dev Spaces standardmäßig alle neuen Container, die in diesem Namespace ausgeführt werden. Azure Dev Spaces testet und instrumentiert außerdem alle neuen Container, die für Dienste erstellt wurden, die bereits im Namespace ausgeführt werden. Um zu verhindern, dass Azure Dev Spaces einen in Ihrem Namespace ausgeführten Container instrumentiert, fügen Sie den Header *no-proxy* (Kein Proxy) zur `deployment.yaml` hinzu.

Fügen Sie `azds.io/no-proxy: "true"` zur Datei `existingWindowsBackend/mywebapi-windows/charts/templates/deployment.yaml` hinzu:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  ...
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    ...
  template:
    metadata:
      labels:
        app.kubernetes.io/name: {{ include "mywebapi.name" . }}
        app.kubernetes.io/instance: {{ .Release.Name }}
        azds.io/no-proxy: "true"
```

Verwenden Sie `helm list`, um die Bereitstellung für den Windows-Dienst aufzulisten:

```cmd
$ helm list
NAME            REVISION    UPDATED                     STATUS      CHART           APP VERSION NAMESPACE
gilded-jackal   1           Wed Jul 24 15:45:59 2019    DEPLOYED    mywebapi-0.1.0  1.0         dev  
```

Im oben stehenden Beispiel lautet der Name Ihrer Bereitstellung *gilded-jackal*. Aktualisieren Sie den Windows-Dienst mit der neuen Konfiguration unter Verwendung von `helm upgrade`:

```cmd
$ helm upgrade gilded-jackal . --namespace dev
Release "gilded-jackal" has been upgraded.
```

Da Sie die Datei `deployment.yaml` aktualisiert haben, wird Ihr Dienst von Azure Dev Spaces nicht getestet und instrumentiert.

## <a name="run-your-linux-application-with-azure-dev-spaces"></a>Ausführen Ihrer Linux-Anwendung mit Azure Dev Spaces

Navigieren Sie zum Verzeichnis `webfrontend`, und führen Sie Ihre Linux-Anwendung unter Verwendung der Befehle `azds prep` und `azds up` in Ihrem Cluster aus.

```console
cd ../../webfrontend-linux/
azds prep --public
azds up
```

Der Befehl `azds prep --public` generiert das Helm-Diagramm und die Dockerfile-Dateien für Ihre Anwendung. Der Befehl `azds up` führt den Dienst im-Namespace aus.

```console
$ azds up
Using dev space 'dev' with target 'myAKSCluster'
Synchronizing files...4s
Installing Helm chart...11s
Waiting for container image build...6s
Building container image...
Step 1/12 : FROM mcr.microsoft.com/dotnet/core/sdk:2.2
...
Step 12/12 : ENTRYPOINT ["/bin/bash", "/entrypoint.sh"]
Built container image in 36s
Waiting for container...2s
Service 'webfrontend' port 'http' is available at http://dev.webfrontend.abcdef0123.eus.azds.io/
Service 'webfrontend' port 80 (http) is available via port forwarding at http://localhost:57648
```

Sie können die Ausführung des Diensts verfolgen, indem Sie die öffentliche URL öffnen, die in der Ausgabe des Befehls „azds up“ angezeigt wird. In diesem Beispiel lautet die öffentliche URL `http://dev.webfrontend.abcdef0123.eus.azds.io/`. Navigieren Sie in einem Browser zum Dienst, und klicken Sie oben auf *Info*. Vergewissern Sie sich, dass eine Meldung vom Dienst *mywebapi* angezeigt wird, in der die vom Container verwendete Windows-Version enthalten ist.

![Beispiel-App mit Windows-Version aus mywebapi](../media/run-dev-spaces-windows-containers/sample-app.png)

## <a name="next-steps"></a>Nächste Schritte

Informieren Sie sich darüber, wie Azure Dev Spaces Sie bei der Entwicklung komplexerer containerübergreifender Anwendungen unterstützt und wie Sie die gemeinsame Entwicklung vereinfachen können, indem Sie in verschiedenen Bereichen mit verschiedenen Versionen oder Branches Ihres Codes arbeiten.

> [!div class="nextstepaction"]
> [Schnellstart: Entwicklung im Team mit Java unter Kubernetes mithilfe von Azure Dev Spaces][team-development-qs]

[kubectl]: https://kubernetes.io/docs/user-guide/kubectl/
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[helm-installed]: https://github.com/helm/helm/blob/master/docs/install.md
[sample-application]: https://github.com/Azure/dev-spaces/tree/master/samples/existingWindowsBackend
[team-development-qs]: ../quickstart-team-development.md

[az-aks-get-credentials]: /cli/azure/aks?view=azure-cli-latest#az-aks-get-credentials
[team-development]: ../team-development-netcore.md
[using-taints]: ../../aks/use-multiple-node-pools.md#schedule-pods-using-taints-and-tolerations
[windows-container-cli]: ../../aks/windows-container-cli.md
