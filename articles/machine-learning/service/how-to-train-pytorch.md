---
title: Trainieren und Registrieren von PyTorch-Modellen
titleSuffix: Azure Machine Learning service
description: In diesem Artikel erfahren Sie, wie Sie ein PyTorch-Modell mit Azure Machine Learning Service trainieren und registrieren.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: maxluk
author: maxluk
ms.reviewer: peterlu
ms.date: 06/18/2019
ms.custom: seodec18
ms.openlocfilehash: d9c953eeecedf14a8f3fae43c5d4713252d58b4c
ms.sourcegitcommit: 64798b4f722623ea2bb53b374fb95e8d2b679318
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 07/11/2019
ms.locfileid: "67840080"
---
# <a name="train-and-register-pytorch-models-at-scale-with-azure-machine-learning-service"></a>Trainieren und Registrieren von PyTorch-Modellen in großem Umfang mit Azure Machine Learning Service

In diesem Artikel erfahren Sie, wie Sie ein PyTorch-Modell mit Azure Machine Learning Service trainieren und registrieren. Es basiert auf dem [Transfer Learning Tutorial von PyTorch](https://pytorch.org/tutorials/beginner/transfer_learning_tutorial.html), das einen DNN-Klassifizierer (Deep Neural Network, tiefes neuronales Netzwerk) für Bilder von Hühnern und Puten aufbaut.

[PyTorch](https://pytorch.org/) ist ein Open-Source-Computing-Framework, das häufig zum Aufbau von Deep Neural Networks (DNN) verwendet wird. Azure Machine Learning Service bietet Ihnen die Möglichkeit, Ihre Open-Source-Trainingsaufträge mithilfe elastischer Cloudcomputeressourcen schnell zu erweitern. Sie können auch Ihre Trainingsdurchläufe, Versionsmodelle, Bereitstellungsmodelle und vieles mehr verfolgen.

Egal, ob Sie ein PyTorch-Modell von Grund auf entwickeln oder ein bestehendes Modell in die Cloud bringen, Azure Machine Learning Service kann Ihnen helfen, produktionsreife Modelle zu erstellen.

## <a name="prerequisites"></a>Voraussetzungen

Führen Sie diesen Code in einer dieser Umgebungen aus:

 - Azure Machine Learning Notebook VM: keine Downloads oder Installationen erforderlich

    - Vervollständigen Sie den Schnellstart des [Cloud-basierten Notebooks](quickstart-run-cloud-notebook.md), um einen dedizierten Notebookserver zu erstellen, der mit dem SDK und dem Beispielrepository vorinstalliert ist.
    - Suchen Sie im Beispielordner auf dem Notebookserver ein fertiges und erweitertes Notebook. Dazu navigieren Sie zum folgenden Verzeichnis: **how-to-use-azureml > training-with-deep-learning > train-hyperparameter-tune-deploy-with-pytorch**. 
 
 - Ihr eigener Jupyter Notebook-Server

    - [Installieren Sie das Azure Machine Learning SDK für Python](setup-create-workspace.md#sdk).
    - [Erstellen einer Konfigurationsdatei für den Arbeitsbereich](setup-create-workspace.md#write-a-configuration-file)
    - [Herunterladen der Beispielskriptdateien](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/training-with-deep-learning/train-hyperparameter-tune-deploy-with-pytorch) `pytorch_train.py`
     
    Auf der GitHub-Seite mit Beispielen finden Sie außerdem eine fertige [Jupyter Notebook-Version](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/training-with-deep-learning/train-hyperparameter-tune-deploy-with-pytorch/train-hyperparameter-tune-deploy-with-pytorch.ipynb) dieser Anleitung. Das Notebook umfasst erweiterte Abschnitte, in denen die intelligente Hyperparameteroptimierung, die Modellimplementierung und Notebook-Widgets behandelt werden.

## <a name="set-up-the-experiment"></a>Einrichten des Experiments

In diesem Abschnitt wird das Trainingsexperiment eingerichtet, indem die erforderlichen Python-Pakete geladen, ein Arbeitsbereich initialisiert, ein Experiment erstellt und die Trainingsdaten und -skripts hochgeladen werden.

### <a name="import-packages"></a>Importieren von Paketen

Importieren Sie zunächst die erforderlichen Python-Bibliotheken.

```Python
import os
import shutil

from azureml.core.workspace import Workspace
from azureml.core import Experiment

from azureml.core.compute import ComputeTarget, AmlCompute
from azureml.core.compute_target import ComputeTargetException
from azureml.train.dnn import PyTorch
```

### <a name="initialize-a-workspace"></a>Initialisieren eines Arbeitsbereichs

Der [Azure Machine Learning Service-Arbeitsbereich](concept-workspace.md) ist für den Dienst die Ressource der obersten Ebene. Er stellt den zentralen Ort für die Arbeit mit allen erstellten Artefakten dar. Im Python SDK können Sie auf die Arbeitsbereichsartefakte zugreifen, indem Sie ein [`workspace`](https://docs.microsoft.com/python/api/azureml-core/azureml.core.workspace.workspace?view=azure-ml-py)-Objekt erstellen.

Erstellen Sie ein Arbeitsbereichsobjekt aus der Datei `config.json`, die im [Abschnitt „Voraussetzungen“](#prerequisites) erstellt wurde.

```Python
ws = Workspace.from_config()
```

### <a name="create-an-experiment"></a>Erstellen eines Experiments

Erstellen Sie ein Experiment und einen Ordner, in dem Ihre Trainingsskripts gespeichert werden. In diesem Beispiel erstellen Sie ein Experiment mit dem Namen „pytorch-birds“.

```Python
project_folder = './pytorch-birds'
os.makedirs(project_folder, exist_ok=True)

experiment_name = 'pytorch-birds'
experiment = Experiment(ws, name=experiment_name)
```

### <a name="get-the-data"></a>Abrufen von Daten

Das Dataset besteht aus etwa 120 Trainingsbildern, jeweils von Puten und Hühnern, mit 100 Validierungsbildern für jede Klasse. Im Rahmen unseres Trainingsskripts `pytorch_train.py` laden wir das Dataset herunter und extrahieren es. Die Bilder sind eine Teilmenge des [Open Images v5 Dataset](https://storage.googleapis.com/openimages/web/index.html).

### <a name="prepare-training-scripts"></a>Vorbereiten von Trainingsskripts

In diesem Tutorial ist das Trainingsskript `pytorch_train.py` bereits bereitgestellt. In der Praxis können Sie jedes benutzerdefinierte Trainingsskript, wie es ist, verwenden und mit dem Azure Machine Learning Service ausführen.

Laden Sie das Pytorch-Trainingsskript `pytorch_train.py` hoch.

```Python
shutil.copy('pytorch_train.py', project_folder)
```

Wenn Sie jedoch die Nachverfolgungs- und Metrikfunktionen von Azure Machine Learning Service nutzen möchten, müssen Sie Ihr Trainingsskript mit ein wenig Code ergänzen. Beispiele für die Metrikennachverfolgung finden Sie in `pytorch_train.py`.

## <a name="create-a-compute-target"></a>Erstellen eines Computeziels

Erstellen Sie ein Computeziel, auf dem der PyTorch-Auftrag ausgeführt werden soll. In diesem Beispiel erstellen Sie einen GPU-fähigen Azure Machine Learning-Computercluster.

```Python
cluster_name = "gpucluster"

try:
    compute_target = ComputeTarget(workspace=ws, name=cluster_name)
    print('Found existing compute target')
except ComputeTargetException:
    print('Creating a new compute target...')
    compute_config = AmlCompute.provisioning_configuration(vm_size='STANDARD_NC6', 
                                                           max_nodes=4)

    compute_target = ComputeTarget.create(ws, cluster_name, compute_config)

    compute_target.wait_for_completion(show_output=True, min_node_count=None, timeout_in_minutes=20)
```

Weitere Informationen zu Computezielen finden Sie im Artikel [Was ist ein Computeziel?](concept-compute-target.md).

## <a name="create-a-pytorch-estimator"></a>Erstellen eines PyTorch-Estimators

Der [PyTorch-Estimator](https://docs.microsoft.com/python/api/azureml-train-core/azureml.train.dnn.pytorch?view=azure-ml-py) bietet eine einfache Möglichkeit, einen PyTorch-Trainingsauftrag auf einem Rechenziel zu starten.

Der PyTorch-Estimator wird durch die generische [`estimator`](https://docs.microsoft.com/python/api/azureml-train-core/azureml.train.estimator.estimator?view=azure-ml-py)-Klasse implementiert, die zur Unterstützung beliebiger Frameworks verwendet werden kann. Weitere Informationen zum Trainieren von Modellen mit dem generischen Estimator finden Sie unter [Trainieren von Azure Machine Learning-Modellen mit einem Estimator](how-to-train-ml-models.md).

Wenn für Ihr Trainingsskript zusätzliche Pip- oder Conda-Pakete ausgeführt werden müssen, können Sie die Pakete auf dem resultierenden Docker-Image installieren, indem Sie die Namen mit dem `pip_packages`-Argument und `conda_packages`-Argument übergeben.

```Python
script_params = {
    '--num_epochs': 30,
    '--output_dir': './outputs'
}

estimator = PyTorch(source_directory=project_folder, 
                    script_params=script_params,
                    compute_target=compute_target,
                    entry_script='pytorch_train.py',
                    use_gpu=True,
                    pip_packages=['pillow==5.4.1'])
```

## <a name="submit-a-run"></a>Initiieren einer Ausführung

Das [Run-Objekt](https://docs.microsoft.com/python/api/azureml-core/azureml.core.run%28class%29?view=azure-ml-py) bildet die Schnittstelle zum Ausführungsverlauf, während der Auftrag ausgeführt wird und nachdem er abgeschlossen wurde.

```Python
run = experiment.submit(estimator)
run.wait_for_completion(show_output=True)
```

Die Ausführung durchläuft die folgenden Phasen:

- **Vorbereitung**: Gemäß dem PyTorch-Estimator wird ein Docker-Image erstellt. Das Image wird in die Containerregistrierung des Arbeitsbereichs hochgeladen und für spätere Ausführungen zwischengespeichert. Darüber hinaus werden Protokolle in den Ausführungsverlauf gestreamt, mit deren Hilfe der Status überwacht werden kann.

- **Skalierung**: Der Cluster versucht ein zentrales Hochskalieren, wenn der Batch KI-Cluster mehr Knoten zur Ausführung benötigt, als derzeit verfügbar sind.

- **Running**: Alle Skripts im Skriptordner werden auf das Computeziel hochgeladen, Datenspeicher werden bereitgestellt oder kopiert, und das „entry_script“ wird ausgeführt. Ausgaben aus „stdout“ und dem Ordner „./logs“ werden in den Ausführungsverlauf gestreamt und können zur Überwachung der Ausführung verwendet werden.

- **Nachbearbeitung**: Der Ordner „./outputs“ der Ausführung wird in den Ausführungsverlauf kopiert.

## <a name="register-or-download-a-model"></a>Registrieren oder Herunterladen eines Modells

Sobald Sie das Modell trainiert haben, können Sie es in Ihrem Arbeitsbereich registrieren. Die Modellregistrierung bietet die Möglichkeit, Ihre Modelle in Ihrem Arbeitsbereich zu speichern und zu versionieren, um die [Modellverwaltung und -bereitstellung](concept-model-management-and-deployment.md) zu vereinfachen.

```Python
model = run.register_model(model_name='pt-dnn', model_path='outputs/')
```

Mit dem Run-Objekt können Sie auch eine lokale Kopie des Modells herunterladen. Im Trainingsskript `pytorch_train.py` wird das Modell durch ein Speicherobjekt von PyTorch persistent in einem lokalen Ordner (lokal für das Computeziel) gespeichert. Sie können das Run-Objekt verwenden, um eine Kopie herunterzuladen.

```Python
# Create a model folder in the current directory
os.makedirs('./model', exist_ok=True)

for f in run.get_file_names():
    if f.startswith('outputs/model'):
        output_file_path = os.path.join('./model', f.split('/')[-1])
        print('Downloading from {} to {} ...'.format(f, output_file_path))
        run.download_file(name=f, output_file_path=output_file_path)
```

## <a name="distributed-training"></a>Verteiltes Training

Der [`PyTorch`](https://docs.microsoft.com/python/api/azureml-train-core/azureml.train.dnn.pytorch?view=azure-ml-py)-Estimator unterstützt auch verteiltes Training für CPU- und GPU-Cluster. Sie können ganz einfach verteilte PyTorch-Aufträge ausführen. Azure Machine Learning Service verwaltet die Orchestrierung für Sie.

### <a name="horovod"></a>Horovod
[Horovod](https://github.com/uber/horovod) ist ein Open-Source-Allreduce-Framework, das von Uber für verteiltes Training entwickelt wurde. Es ermöglicht die einfache Erstellung von verteilten GPU-PyTorch-Aufträgen.

Um Horovod zu verwenden, geben Sie im PyTorch-Konstruktor ein [`MpiConfiguration`](https://docs.microsoft.com/python/api/azureml-core/azureml.core.runconfig.mpiconfiguration?view=azure-ml-py)-Objekt für den `distributed_training`-Parameter an. Dieser Parameter stellt sicher, dass die Horovod-Bibliothek installiert ist, die Sie in Ihrem Trainingsskript verwenden.


```Python
from azureml.train.dnn import PyTorch

estimator= PyTorch(source_directory=project_folder,
                      compute_target=compute_target,
                      script_params=script_params,
                      entry_script='script.py',
                      node_count=2,
                      process_count_per_node=1,
                      distributed_training=MpiConfiguration(),
                      framework_version='1.13',
                      use_gpu=True)
```
Horovod und alle zugehörigen Abhängigkeiten werden automatisch installiert. Sie können das entsprechende Modul daher wie folgt in das Trainingsskript `train.py` importieren:

```Python
import torch
import horovod
```
## <a name="export-to-onnx"></a>Exportieren nach ONNX

Um Rückschlüsse mit der [ONNX-Runtime](concept-onnx.md) zu optimieren, konvertieren Sie Ihr trainiertes PyTorch-Modell in dass ONNX-Format. Rückschlüsse oder Modellbewertungen bilden die Phase, in der das bereitgestellte Modell für die Vorhersage verwendet wird, meist für Produktionsdaten. Ein Beispiel finden Sie im [Tutorial](https://github.com/onnx/tutorials/blob/master/tutorials/PytorchOnnxExport.ipynb).

## <a name="next-steps"></a>Nächste Schritte

In diesem Artikel haben Sie ein PyTorch-Modell in Azure Machine Learning Service trainiert und registriert. Um zu erfahren, wie Sie ein Modell bereitstellen, fahren Sie mit unserem Artikel zur Modellbereitstellung fort.

> [!div class="nextstepaction"]
> [Wie und wo Modelle bereitgestellt werden](how-to-deploy-and-where.md)
* [Erfassen einer Ausführungsmetrik während des Trainings](how-to-track-experiments.md)
* [Optimieren von Hyperparametern](how-to-tune-hyperparameters.md)
* [Bereitstellen eines trainierten Modells](how-to-deploy-and-where.md)
