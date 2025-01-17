---
title: Parallele Ausführung von Tasks zur effizienten Nutzung von Computeressourcen – Azure Batch | Microsoft-Dokumentation
description: Steigern der Effizienz und Reduzieren von Kosten durch Verringern der Serverknotenzahl und Ausführen paralleler Aufgaben auf jedem Knoten in einem Azure Batch-Pool
services: batch
documentationcenter: .net
author: laurenhughes
manager: gwallace
editor: ''
ms.assetid: 538a067c-1f6e-44eb-a92b-8d51c33d3e1a
ms.service: batch
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: ''
ms.workload: big-compute
ms.date: 04/17/2019
ms.author: lahugh
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: cc6a607da2227ecf9acd6209e31b7aa0ef1c62d8
ms.sourcegitcommit: 4b431e86e47b6feb8ac6b61487f910c17a55d121
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 07/18/2019
ms.locfileid: "68323366"
---
# <a name="run-tasks-concurrently-to-maximize-usage-of-batch-compute-nodes"></a>Gleichzeitige Ausführung von Tasks zur optimalen Nutzung von Batch-Computeknoten 

Wenn Sie mehrere Aufgaben gleichzeitig in jedem Computeknoten in Ihrem Azure Batch-Pool ausführen, wird die Maximierung der Ressourcenauslastung auf einer kleineren Anzahl von Knoten im Pool möglich. Bei einigen Workloads kann dies zu kürzeren Auftragszeiten und niedrigeren Kosten führen.

In manchen Szenarien profitieren Sie davon, dass alle Ressourcen eines Knotens einer einzigen Aufgabe zugewiesen werden können. Dagegen profitieren Sie in vielen Situationen davon, diese Ressourcen von mehreren Aufgaben nutzen zu lassen:

* **Minimieren des Datenübertragung** , wenn Aufgaben Daten gemeinsam nutzen können. In diesem Szenario können Sie die Gebühren für die Datenübertragung deutlich reduzieren, indem Sie freigegebene Daten auf eine kleinere Anzahl Knoten kopieren und Aufgaben parallel an jedem Knoten ausführen. Dies trifft besonders dann zu, wenn die zu jedem Knoten zu kopierenden Daten zwischen verschiedenen geografischen Regionen übertragen werden müssen.
* **Maximieren der Speicherauslastung** , wenn Aufgaben viel Speicherplatz beanspruchen – jedoch nur für kurze Zeiträumen und zu unterschiedlichen Zeiten während der Ausführung. Sie können weniger, aber dafür größere Computeknoten mit mehr Arbeitsspeicher einsetzen, um solche Spitzen effizient zu bewältigen. Auf diesen Knoten werden dann mehrere Aufgaben parallel ausgeführt, doch jede Aufgabe kann den reichlich vorhandenen Arbeitsspeicher zu verschiedenen Zeiten nutzen.
* **Verringern der Beschränkungen der Knotenanzahl** , wenn die Kommunikation zwischen Knoten in einem Pool erforderlich ist. Derzeit sind Pools, die für die Kommunikation zwischen Knoten konfiguriert sind, auf 50 Computeknoten beschränkt. Wenn jeder Knoten in einem Pool Aufgaben parallel ausführen kann, kann also eine größere Anzahl von Aufgaben gleichzeitig ausgeführt werden.
* **Replizieren eines lokalen Computeclusters**, z.B. beim ersten Auslagern einer Compute-Umgebung in Azure. Wenn Ihre aktuelle lokale Lösung mehrere Aufgaben pro Computeknoten ausführt, können Sie die maximale Anzahl von Knotenaufgaben erhöhen, um diese Konfiguration besser widerzuspiegeln.

## <a name="example-scenario"></a>Beispielszenario
Um die Vorteile der parallelen Aufgabenausführung zu veranschaulichen, nehmen wir an, dass für Ihre Aufgabenanwendung CPU- und Speicheranforderungen vorliegen, sodass die Knotengröße [Standard\_D1](../cloud-services/cloud-services-sizes-specs.md) geeignet ist. Um den Auftrag in der geforderten Zeit abzuschließen, sind jedoch 1.000 dieser Knoten nötig.

Anstatt Standard\_D1-Knoten mit einem CPU-Kern zu verwenden, können Sie [Standard\_D14](../cloud-services/cloud-services-sizes-specs.md)-Knoten mit je 16 Kernen verwenden und so die parallele Ausführung von Aufgaben ermöglichen. Daher könnte *etwa ein Sechzehntel der Knoten* verwendet werden – statt 1.000 Knoten wären nur 63 erforderlich. Wenn große Anwendungsdateien oder Verweisdaten für jeden Knoten erforderlich sind, werden Auftragsdauer und Effizienz außerdem erneut verbessert, da die Daten nur auf 63 Knoten kopiert werden.

## <a name="enable-parallel-task-execution"></a>Aktivieren der parallelen Aufgabenausführung
Sie konfigurieren die Computeknoten für die parallele Aufgabenausführung auf der Pool-Ebene. Legen Sie bei der Batch-Bibliothek für .NET die Eigenschaft [CloudPool.MaxTasksPerComputeNode][maxtasks_net] property when you create a pool. If you are using the Batch REST API, set the [maxTasksPerNode][rest_addpool]-Element im Anforderungstext während der Poolerstellung fest. In Azure Batch kann die Anzahl von Aufgaben pro Knoten auf die bis zu vierfache Anzahl von Kernknoten festgelegt werden.

Ist der Pool beispielsweise mit Knoten der Größe „Groß“ (vier Kerne) konfiguriert, kann für „ `maxTasksPerNode` ” 16 festgelegt werden. Unabhängig von der Anzahl von Kernen, über die der Knoten verfügt, können maximal 256 Aufgaben pro Knoten vorliegen. Ausführliche Informationen zur Anzahl der Kerne für jede Knotengröße finden Sie unter [Größen für Clouddienste](../cloud-services/cloud-services-sizes-specs.md). Weitere Informationen zu den Grenzen des Dienstes finden Sie in [Kontingente und Einschränkungen für den Azure Batch-Dienst](batch-quota-limit.md). Berücksichtigen Sie unbedingt den Wert `maxTasksPerNode`, wenn Sie für Ihren Pool eine [Formel für das automatische Skalieren][enable_autoscaling] erstellen.

> [!TIP]
> Beispielsweise könnte eine Formel zum Auswerten von `$RunningTasks` erheblich von einer Steigerung der Aufgaben pro Knoten betroffen sein. Weitere Informationen finden Sie unter [Automatisches Skalieren von Computeknoten in einem Azure Batch-Pool](batch-automatic-scaling.md) . Verteilung von Aufgaben
>
>

## <a name="distribution-of-tasks"></a>Wenn die Computeknoten in einem Pool Aufgaben parallel ausführen können, müssen Sie angeben, wie die Aufgaben auf die Knoten innerhalb des Pools verteilt werden sollen.
Mithilfe der [CloudPool.TaskSchedulingPolicy][task_schedule]-Eigenschaft können Sie angeben, dass Aufgaben gleichmäßig über alle Knoten im Pool zugewiesen werden sollen („Verteilen“).

Oder Sie können angeben, dass jedem Knoten so viele Aufgaben wie möglich zugewiesen werden sollen, bevor sie einem anderen Knoten im Pool zugewiesen werden („Packen“). Als ein Beispiel für die Bedeutung dieses Features betrachten Sie den Pool von [Standard\_D14](../cloud-services/cloud-services-sizes-specs.md)-Knoten (im vorstehenden Beispiel), das konfiguriert wird mit einem [CloudPool.MaxTasksPerComputeNode][maxtasks_net] value of 16. If the [CloudPool.TaskSchedulingPolicy][task_schedule] mit einem [ComputeNodeFillType][fill_type] von *Pack* konfiguriert wird, würde dadurch die Auslastung aller 16 Kerne der einzelnen Knoten maximiert und es einem [Pool für automatische Skalierung](batch-automatic-scaling.md) ermöglicht, nicht genutzte Knoten aus dem Pool (d.h. Knoten ohne zugewiesene Aufgaben) zu löschen.

Dies minimiert die Ressourcenverwendung und spart Geld. Beispiel für Batch .NET Dieser [Batch .NET][api_net] API code snippet shows a request to create a pool that contains four nodes with a maximum of four tasks per node. It specifies a task scheduling policy that will fill each node with tasks prior to assigning tasks to another node in the pool. For more information on adding pools by using the Batch .NET API, see [BatchClient.PoolOperations.CreatePool][poolcreate_net].

## <a name="batch-net-example"></a>Beispiel für Batch REST
Dieser [Batch REST][api_rest] API snippet shows a request to create a pool that contains two large nodes with a maximum of four tasks per node. For more information on adding pools by using the REST API, see [Add a pool to an account][rest_addpool]. Sie können das Element `maxTasksPerNode` und die [MaxTasksPerComputeNode][maxtasks_net]-Eigenschaft nur zum Zeitpunkt der Poolerstellung festlegen. Nach der Erstellung eines Pools können sie nicht mehr geändert werden.

```csharp
CloudPool pool =
    batchClient.PoolOperations.CreatePool(
        poolId: "mypool",
        targetDedicatedComputeNodes: 4
        virtualMachineSize: "standard_d1_v2",
        cloudServiceConfiguration: new CloudServiceConfiguration(osFamily: "5"));

pool.MaxTasksPerComputeNode = 4;
pool.TaskSchedulingPolicy = new TaskSchedulingPolicy(ComputeNodeFillType.Pack);
pool.Commit();
```

## <a name="batch-rest-example"></a>Codebeispiel
Das [ParallelNodeTasks][parallel_tasks_sample] project on GitHub illustrates the use of the [CloudPool.MaxTasksPerComputeNode][maxtasks_net]-Eigenschaft. Diese C#-Konsolenanwendung verwendet die [Batch-Bibliothek für .NET][api_net] zum Erstellen eines Pools mit einem oder mehreren Serverknoten.

```json
{
  "odata.metadata":"https://myaccount.myregion.batch.azure.com/$metadata#pools/@Element",
  "id":"mypool",
  "vmSize":"large",
  "cloudServiceConfiguration": {
    "osFamily":"4",
    "targetOSVersion":"*",
  }
  "targetDedicatedComputeNodes":2,
  "maxTasksPerNode":4,
  "enableInterNodeCommunication":true,
}
```

> [!NOTE]
> Sie führt eine konfigurierbare Anzahl an Aufgaben auf diesen Knoten aus, um eine variable Auslastung zu simulieren. Die Ausgabe der Anwendung gibt an, welcher Knoten welche Aufgabe ausgeführt hat.
>
>

## <a name="code-sample"></a>Die Anwendung liefert auch eine Zusammenfassung der Aufgabenparameter und der -dauer.
Die Zusammenfassung der Ausgabe von zwei verschiedenen Ausführungen der Beispielanwendung wird unten angezeigt.

Die erste Ausführung der Beispielanwendung zeigt, dass die Aufgabe bei Nutzung eines einzigen Knotens im Pool und einer Aufgabe pro Knoten mehr als 30 Minuten dauert. Die zweite Ausführung des Beispiels zeigt eine deutliche Verringerung der Aufgabendauer. Das liegt daran, dass der Pool mit vier Aufgaben pro Knoten konfiguriert wurde, was die parallele Aufgabenausführung in etwa einem Viertel der Zeit ermöglicht. In der Aufgabedauer in den oben aufgeführten Zusammenfassungen ist die Erstellung des Pools nicht berücksichtigt. Jede der oben aufgeführten Aufgaben wurde an bereits erstellte Pools übermittelt, deren Serverknoten sich zum Übermittlungszeitpunkt im *Idle* -Status befanden.

```
Nodes: 1
Node size: large
Max tasks per node: 1
Tasks: 32
Duration: 00:30:01.4638023
```

Nächste Schritte

```
Nodes: 1
Node size: large
Max tasks per node: 4
Tasks: 32
Duration: 00:08:48.2423500
```

Heat Map für Batch Explorer Der [Batch Explorer][batch_labs] ist ein kostenloses eigenständiges Clienttool mit zahlreichen Features, das Sie beim Erstellen, Debuggen und Überwachen von Azure Batch-Anwendungen unterstützt.

> [!NOTE]
> Der Batch Explorer enthält ein *Wärmebild*-Feature, das die Visualisierung der Taskausführung ermöglicht. Wenn Sie die Beispielanwendung [ParallelTasks][parallel_tasks_sample] ausführen, können Sie die Heat Map-Funktion für eine einfache Visualisierung der Ausführung paralleler Aufgaben auf jedem Knoten verwenden.
>
>

## <a name="next-steps"></a>Next steps
### <a name="batch-explorer-heat-map"></a>Batch Explorer Heat Map
<bpt id="p1">[</bpt>Batch Explorer<ept id="p1">][batch_labs]</ept> is a free, rich-featured, standalone client tool to help create, debug, and monitor Azure Batch applications. Batch Explorer contains a <bpt id="p1">*</bpt>Heat Map<ept id="p1">*</ept> feature that provides visualization of task execution. When you're executing the <bpt id="p1">[</bpt>ParallelTasks<ept id="p1">][parallel_tasks_sample]</ept> sample application, you can use the Heat Map feature to easily visualize the execution of parallel tasks on each node.


[api_net]: https://msdn.microsoft.com/library/azure/mt348682.aspx
[api_rest]: https://msdn.microsoft.com/library/azure/dn820158.aspx
[batch_labs]: https://azure.github.io/BatchExplorer/
[cloudpool]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudpool.aspx
[enable_autoscaling]: https://msdn.microsoft.com/library/azure/dn820173.aspx
[fill_type]: https://msdn.microsoft.com/library/microsoft.azure.batch.common.computenodefilltype.aspx
[github_samples]: https://github.com/Azure/azure-batch-samples
[maxtasks_net]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudpool.maxtaskspercomputenode.aspx
[rest_addpool]: https://msdn.microsoft.com/library/azure/dn820174.aspx
[parallel_tasks_sample]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/ArticleProjects/ParallelTasks
[poolcreate_net]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.pooloperations.createpool.aspx
[task_schedule]: https://msdn.microsoft.com/library/microsoft.azure.batch.cloudpool.taskschedulingpolicy.aspx

