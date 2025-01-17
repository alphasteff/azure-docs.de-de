---
title: Trainieren und Registrieren von Keras-Modellen, die auf TensorFlow ausgeführt werden
titleSuffix: Azure Machine Learning service
description: In diesem Artikel erfahren Sie, wie Sie ein Keras-Modell, das auf TensorFlow ausgeführt wird, mit Azure Machine Learning Service trainieren und registrieren.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: maxluk
author: maxluk
ms.reviewer: peterlu
ms.date: 06/07/2019
ms.custom: seodec18
ms.openlocfilehash: 9d405b454d755e0c848e9422c8d4cf6e7c505b68
ms.sourcegitcommit: 64798b4f722623ea2bb53b374fb95e8d2b679318
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 07/11/2019
ms.locfileid: "67840048"
---
# <a name="train-and-register-keras-models-at-scale-with-azure-machine-learning-service"></a>Trainieren und Registrieren von Keras-Modellen in großem Umfang mit Azure Machine Learning Service

In diesem Artikel erfahren Sie, wie Sie ein Keras-Modell, das auf TensorFlow basiert, mit Azure Machine Learning Service trainieren und registrieren. Es verwendet das beliebte [MNIST-Dataset](http://yann.lecun.com/exdb/mnist/), um handschriftliche Ziffern mithilfe eines DNN (Deep Neural Network) zu klassifizieren, das mit der [Keras Python-Bibliothek](https://keras.io) erstellt wurde, die auf [TensorFlow](https://www.tensorflow.org/overview) ausgeführt wird.

Keras ist eine allgemeine API für neuronale Netzwerke, die zur Vereinfachung der Entwicklung auf anderen beliebten DNN-Frameworks ausgeführt werden kann. Azure Machine Learning Service bietet die Möglichkeit, Trainingsaufträge mithilfe elastischer Cloudcomputeressourcen schnell horizontal zu skalieren. Sie können auch Ihre Trainingsläufe, Versionsmodelle, Bereitstellungsmodelle und vieles mehr verfolgen.

Egal, ob Sie ein Keras-Modell von Grund auf entwickeln oder ein bestehendes Modell in die Cloud bringen, Azure Machine Learning Service kann Ihnen helfen, produktionsreife Modelle zu erstellen.

## <a name="prerequisites"></a>Voraussetzungen

Führen Sie diesen Code in einer dieser Umgebungen aus:

 - Azure Machine Learning Notebook VM: keine Downloads oder Installationen erforderlich

     - Führen Sie den [Schnellstart für cloudbasierte Notebooks](quickstart-run-cloud-notebook.md) durch, um einen dedizierten Notebookserver zu erstellen, auf dem das SDK und das Beispielrepository vorinstalliert sind.
    - Suchen Sie im Beispielordner auf dem Notebookserver ein fertiges und erweitertes Notebook. Dazu navigieren Sie zum folgenden Verzeichnis: **how-to-use-azureml > training-with-deep-learning > train-hyperparameter-tune-deploy-with-keras**. 
 
 - Ihr eigener Jupyter Notebook-Server

     - [Installieren des Azure Machine Learning SDK für Python](setup-create-workspace.md#sdk)
    - [Erstelle Sie eine Konfigurationsdatei für den Arbeitsbereich](setup-create-workspace.md#write-a-configuration-file)
    - [Laden Sie die Beispielskriptdateien](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/training-with-deep-learning/train-hyperparameter-tune-deploy-with-keras) `mnist-keras.py` und `utils.py` herunter.
     
    Auf der GitHub-Seite mit Beispielen finden Sie außerdem eine fertige [Jupyter Notebook-Version](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/training-with-deep-learning/train-hyperparameter-tune-deploy-with-keras/train-hyperparameter-tune-deploy-with-keras.ipynb) dieser Anleitung. Das Notebook umfasst erweiterte Abschnitte, in denen die intelligente Hyperparameteroptimierung, die Modellimplementierung und Notebook-Widgets behandelt werden.

## <a name="set-up-the-experiment"></a>Einrichten des Experiments

In diesem Abschnitt wird das Trainingsexperiment eingerichtet, indem die erforderlichen Python-Pakete geladen, ein Arbeitsbereich initialisiert, ein Experiment erstellt und die Trainingsdaten und -skripts hochgeladen werden.

### <a name="import-packages"></a>Importieren von Paketen

Importieren Sie zunächst die erforderlichen Python-Bibliotheken.

```Python
import os
import urllib
import shutil
import azureml

from azureml.core import Experiment
from azureml.core import Workspace, Run

from azureml.core.compute import ComputeTarget, AmlCompute
from azureml.core.compute_target import ComputeTargetException
```

### <a name="initialize-a-workspace"></a>Initialisieren eines Arbeitsbereichs

Der [Azure Machine Learning Service-Arbeitsbereich](concept-workspace.md) ist für den Dienst die Ressource der obersten Ebene. Er stellt den zentralen Ort für die Arbeit mit allen erstellten Artefakten dar. Im Python SDK können Sie auf die Arbeitsbereichsartefakte zugreifen, indem Sie ein [`workspace`](https://docs.microsoft.com/python/api/azureml-core/azureml.core.workspace.workspace?view=azure-ml-py)-Objekt erstellen.

Erstellen Sie ein Arbeitsbereichsobjekt aus der Datei `config.json`, die im [Abschnitt „Voraussetzungen“](#prerequisites) erstellt wurde.

```Python
ws = Workspace.from_config()
```

### <a name="create-an-experiment"></a>Erstellen eines Experiments

Erstellen Sie ein Experiment und einen Ordner, in dem Ihre Trainingsskripts gespeichert werden. In diesem Beispiel erstellen Sie ein Experiment mit dem Namen „keras-mnist“.

```Python
script_folder = './keras-mnist'
os.makedirs(script_folder, exist_ok=True)

exp = Experiment(workspace=ws, name='keras-mnist')
```

### <a name="upload-dataset-and-scripts"></a>Hochladen von Dataset und Skripts

Der [Datenspeicher](how-to-access-data.md) ist ein Ort, an dem Daten gespeichert und abgerufen werden können, indem die Daten auf dem Computeziel bereitgestellt oder dorthin kopiert werden. Jeder Arbeitsbereich stellt einen Standarddatenspeicher bereit. Laden Sie die Daten und Trainingsskripts in den Datenspeicher hoch, sodass während des Trainings leicht darauf zugegriffen werden kann.

1. Laden Sie das MNIST-Dataset lokal herunter.

    ```Python
    os.makedirs('./data/mnist', exist_ok=True)

    urllib.request.urlretrieve('http://yann.lecun.com/exdb/mnist/train-images-idx3-ubyte.gz', filename = './data/mnist/train-images.gz')
    urllib.request.urlretrieve('http://yann.lecun.com/exdb/mnist/train-labels-idx1-ubyte.gz', filename = './data/mnist/train-labels.gz')
    urllib.request.urlretrieve('http://yann.lecun.com/exdb/mnist/t10k-images-idx3-ubyte.gz', filename = './data/mnist/test-images.gz')
    urllib.request.urlretrieve('http://yann.lecun.com/exdb/mnist/t10k-labels-idx1-ubyte.gz', filename = './data/mnist/test-labels.gz')
    ```

1. Laden Sie das MNIST-Dataset in den Standarddatenspeicher hoch.

    ```Python
    ds = ws.get_default_datastore()
    ds.upload(src_dir='./data/mnist', target_path='mnist', overwrite=True, show_progress=True)
    ```

1. Laden Sie das Keras-Trainingsskript `keras_mnist.py` und die Hilfsdatei `utils.py` hoch.

    ```Python
    shutil.copy('./keras_mnist.py', script_folder)
    shutil.copy('./utils.py', script_folder)
    ```

## <a name="create-a-compute-target"></a>Erstellen eines Computeziels

Erstellen Sie ein Computeziel, auf dem der TensorFlow-Auftrag ausgeführt werden soll. In diesem Beispiel erstellen Sie einen GPU-fähigen Azure Machine Learning-Computercluster.

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

## <a name="create-a-tensorflow-estimator-and-import-keras"></a>Erstellen eines TensorFlow-Estimators und Importieren von Keras

Der [TensorFlow-Estimator](https://docs.microsoft.com/python/api/azureml-train-core/azureml.train.dnn.tensorflow?view=azure-ml-py) bietet eine einfache Möglichkeit, TensorFlow-Trainingsaufträge auf einem Computeziel zu starten. Da Keras auf TensorFlow ausgeführt wird, können Sie den TensorFlow-Estimator verwenden und die Keras-Bibliothek mit dem Argument `pip_packages` importieren.

Der TensorFlow-Estimator wird durch die generische [`estimator`](https://docs.microsoft.com//python/api/azureml-train-core/azureml.train.estimator.estimator?view=azure-ml-py)-Klasse implementiert, die zur Unterstützung beliebiger Frameworks verwendet werden kann. Weitere Informationen zum Trainieren von Modellen mit dem generischen Estimator finden Sie unter [Trainieren von Azure Machine Learning-Modellen mit einem Estimator](how-to-train-ml-models.md).

```Python
script_params = {
    '--data-folder': ds.path('mnist').as_mount(),
    '--batch-size': 50,
    '--first-layer-neurons': 300,
    '--second-layer-neurons': 100,
    '--learning-rate': 0.001
}

est = TensorFlow(source_directory=script_folder,
                 entry_script='keras_mnist.py',
                 script_params=script_params,
                 compute_target=compute_target,
                 pip_packages=['keras', 'matplotlib'],
                 use_gpu=True)
```

## <a name="submit-a-run"></a>Initiieren einer Ausführung

Das [Run-Objekt](https://docs.microsoft.com/python/api/azureml-core/azureml.core.run%28class%29?view=azure-ml-py) bildet die Schnittstelle zum Ausführungsverlauf, während der Auftrag ausgeführt wird und nachdem er abgeschlossen wurde.

```Python
run = exp.submit(est)
run.wait_for_completion(show_output=True)
```

Die Ausführung durchläuft die folgenden Phasen:

- **Vorbereitung**: Gemäß dem TensorFlow-Estimator wird ein Docker-Image erstellt. Das Image wird in die Containerregistrierung des Arbeitsbereichs hochgeladen und für spätere Ausführungen zwischengespeichert. Darüber hinaus werden Protokolle in den Ausführungsverlauf gestreamt, mit deren Hilfe der Status überwacht werden kann.

- **Skalierung**: Der Cluster versucht ein zentrales Hochskalieren, wenn der Batch KI-Cluster mehr Knoten zur Ausführung benötigt, als derzeit verfügbar sind.

- **Running**: Alle Skripts im Skriptordner werden auf das Computeziel hochgeladen, Datenspeicher werden bereitgestellt oder kopiert, und das „entry_script“ wird ausgeführt. Ausgaben aus „stdout“ und dem Ordner „./logs“ werden in den Ausführungsverlauf gestreamt und können zur Überwachung der Ausführung verwendet werden.

- **Nachbearbeitung**: Der Ordner „./outputs“ der Ausführung wird in den Ausführungsverlauf kopiert.

## <a name="register-the-model"></a>Registrieren des Modells

Sobald Sie das Modell trainiert haben, können Sie es in Ihrem Arbeitsbereich registrieren. Die Modellregistrierung bietet die Möglichkeit, Ihre Modelle in Ihrem Arbeitsbereich zu speichern und zu versionieren, um die [Modellverwaltung und -bereitstellung](concept-model-management-and-deployment.md) zu vereinfachen.

```Python
model = run.register_model(model_name='keras-dnn-mnist', model_path='outputs/model')
```

Sie können auch eine lokale Kopie des Modells herunterladen. Dies kann nützlich sein, um zusätzliche Arbeiten zur Modellüberprüfung lokal durchzuführen. Im Trainingsskript `mnist-keras.py` wird das Modell durch ein saver-Objekt von TensorFlow persistent in einem lokalen Ordner (lokal für das Computeziel) gespeichert. Mit dem Run-Objekt können Sie eine Kopie aus dem Datenspeicher herunterladen.

```Python
# Create a model folder in the current directory
os.makedirs('./model', exist_ok=True)

for f in run.get_file_names():
    if f.startswith('outputs/model'):
        output_file_path = os.path.join('./model', f.split('/')[-1])
        print('Downloading from {} to {} ...'.format(f, output_file_path))
        run.download_file(name=f, output_file_path=output_file_path)
```

## <a name="next-steps"></a>Nächste Schritte

In diesem Artikel haben Sie ein Keras-Modell in Azure Machine Learning Service trainiert und registriert. Um zu erfahren, wie Sie ein Modell bereitstellen, fahren Sie mit unserem Artikel zur Modellbereitstellung fort.

> [!div class="nextstepaction"]
> [Wie und wo Modelle bereitgestellt werden](how-to-deploy-and-where.md)
