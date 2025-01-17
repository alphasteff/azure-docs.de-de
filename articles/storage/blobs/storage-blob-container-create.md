---
title: 'Erstellen oder Löschen eines Blobcontainers mit .NET: Azure Storage'
description: Erfahren Sie, wie Sie einen Blobcontainer in Ihrem Azure Storage-Konto mithilfe der .NET-Clientbibliothek erstellen oder löschen.
services: storage
author: tamram
ms.service: storage
ms.topic: article
ms.date: 07/10/2019
ms.author: tamram
ms.subservice: blobs
ms.openlocfilehash: fbeb6dc425709d26768a6185828b900a37c88fb6
ms.sourcegitcommit: b2db98f55785ff920140f117bfc01f1177c7f7e2
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 07/16/2019
ms.locfileid: "68235152"
---
# <a name="create-or-delete-a-container-in-azure-storage-with-net"></a>Erstellen oder Löschen eines Containers in Azure Storage mit .NET

Blobs in Azure Storage sind in Containern organisiert. Bevor Sie ein Blob hochladen können, müssen Sie zunächst einen Container erstellen. In diesem Artikel wird beschrieben, wie Container mit der [Azure Storage-Clientbibliothek für .NET](/dotnet/api/overview/azure/storage/client) erstellt und gelöscht werden.

## <a name="name-a-container"></a>Benennen eines Containers

Ein Containername muss ein gültiger DNS-Name sein, da er Teil des eindeutigen URI ist, der für die Adressierung des Containers oder seiner Blobs verwendet wird. Befolgen Sie diese Regeln, wenn Sie einen Container benennen:

- Containernamen können zwischen 3 und 63 Zeichen lang sein.
- Containernamen müssen mit einem Buchstaben oder einer Zahl beginnen und dürfen nur Kleinbuchstaben, Zahlen und Bindestriche (-) enthalten.
- Zwei oder mehr aufeinanderfolgende Bindestriche sind in Containernamen nicht zulässig.

Der URI für einen Container weist das folgende Format auf:

`https://myaccount.blob.core.windows.net/mycontainer`

## <a name="create-a-container"></a>Erstellen eines Containers

Um einen Container zu erstellen, rufen Sie eine der folgenden Methoden auf:

- [Erstellen](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.create)
- [CreateAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.createasync)
- [CreateIfNotExists](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.createifnotexists)
- [CreateIfNotExistsAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.createifnotexistsasync)

Die Methoden **Create** und **CreateAsync** lösen eine Ausnahme aus, wenn ein Container mit demselben Namen bereits vorhanden ist.

Die Methoden **CreateIfNotExists** und **CreateIfNotExistsAsync** geben einen booleschen Wert zurück, der angibt, ob der Container erstellt wurde. Wenn bereits ein Container mit dem gleichen Namen vorhanden ist, geben diese Methoden **FALSE** zurück, um anzugeben, dass kein neuer Container erstellt wurde.

Container werden direkt unterhalb des Speicherkontos erstellt. Es ist nicht möglich, einen Container unter einem anderen Container zu schachteln.

Im folgenden Beispiel wird ein Container asynchron erstellt:

```csharp
private static async Task<CloudBlobContainer> CreateSampleContainerAsync(CloudBlobClient blobClient)
{
    // Name the sample container based on new GUID, to ensure uniqueness.
    // The container name must be lowercase.
    string containerName = "container-" + Guid.NewGuid();

    // Get a reference to a sample container.
    CloudBlobContainer container = blobClient.GetContainerReference(containerName);

    try
    {
        // Create the container if it does not already exist.
        bool result = await container.CreateIfNotExistsAsync();
        if (result == true)
        {
            Console.WriteLine("Created container {0}", container.Name);
        }
    }
    catch (StorageException e)
    {
        Console.WriteLine("HTTP error code {0}: {1}",
                            e.RequestInformation.HttpStatusCode,
                            e.RequestInformation.ErrorCode);
        Console.WriteLine(e.Message);
    }

    return container;
}
```

## <a name="create-the-root-container"></a>Erstellen des Stammcontainers

Ein Stammcontainer dient als Standardcontainer für Ihr Speicherkonto. Jedes Speicherkonto kann über einen Stammcontainer verfügen, der den Namen *$root* tragen muss. Der Stammcontainer muss explizit erstellt oder gelöscht werden.

Sie können auf ein Blob verweisen, das im Stammcontainer gespeichert ist, ohne den Namen des Stammcontainers einzubeziehen. Der Stammcontainer ermöglicht das Verweisen auf ein Blob auf der obersten Ebene der Speicherkontohierarchie. Beispielsweise können Sie wie folgt auf ein Blob verweisen, das sich im Stammcontainer befindet:

`https://myaccount.blob.core.windows.net/default.html`

Im folgenden Beispiel wird der Stammcontainer synchron erstellt:

```csharp
private static void CreateRootContainer(CloudBlobClient blobClient)
{
    try
    {
        // Create the root container if it does not already exist.
        CloudBlobContainer container = blobClient.GetContainerReference("$root");
        if (container.CreateIfNotExists())
        {
            Console.WriteLine("Created root container.");
        }
        else
        {
            Console.WriteLine("Root container already exists.");
        }
    }
    catch (StorageException e)
    {
        Console.WriteLine("HTTP error code {0}: {1}",
                            e.RequestInformation.HttpStatusCode,
                            e.RequestInformation.ErrorCode);
        Console.WriteLine(e.Message);
    }
}
```

## <a name="delete-a-container"></a>Löschen eines Containers

Um einen Container in .NET zu löschen, verwenden Sie eine der folgenden Methoden:

- [Löschen](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.delete)
- [DeleteAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.deleteasync)
- [DeleteIfExists](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.deleteifexists)
- [DeleteIfExistsAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.deleteifexistsasync)

Die Methoden **Delete** und **DeleteAsync** lösen eine Ausnahme aus, wenn der Container nicht vorhanden ist.

Die Methoden **DeleteIfNotExists** und **DeleteIfNotExistsAsync** geben einen booleschen Wert zurück, der angibt, ob der Container gelöscht wurde. Wenn der angegebene Container nicht vorhanden ist, geben diese Methoden **FALSE** zurück, um anzugeben, dass der Container nicht gelöscht wurde.

Nachdem Sie einen Container gelöscht haben, können Sie mindestens 30 Sekunden lang (möglicherweise länger) keinen Container mit demselben Namen erstellen. Beim Löschen des Containers tritt beim Versuch, einen Container mit demselben Namen zu erstellen, ein Fehler mit dem HTTP-Fehlercode 409 (Konflikt) auf. Alle anderen Vorgänge für den Container oder die darin enthaltenen Blobs schlagen mit dem HTTP-Fehlercode 404 (Nicht gefunden) fehl, während der Container gelöscht wird.

Im folgenden Beispiel wird der angegebene Container gelöscht und die Ausnahme behandelt, wenn der Container nicht vorhanden ist:

```csharp
private static async Task DeleteSampleContainerAsync(CloudBlobClient blobClient, string containerName)
{
    CloudBlobContainer container = blobClient.GetContainerReference(containerName);

    try
    {
        // Delete the specified container and handle the exception.
        await container.DeleteAsync();
    }
    catch (StorageException e)
    {
        Console.WriteLine("HTTP error code {0}: {1}",
                            e.RequestInformation.HttpStatusCode,
                            e.RequestInformation.ErrorCode);
        Console.WriteLine(e.Message);
        Console.ReadLine();
    }
}
```

Im folgenden Beispiel wird gezeigt, wie alle Container gelöscht werden, die mit einem angegebenen Präfix beginnen. Im Beispiel wird die Lease unterbrochen, wenn eine Lease für den Container vorhanden ist.

```csharp
private static async Task DeleteContainersWithPrefixAsync(CloudBlobClient blobClient, string prefix)
{
    Console.WriteLine("Delete all containers beginning with the specified prefix");
    try
    {
        foreach (var container in blobClient.ListContainers(prefix))
        {
            Console.WriteLine("\tContainer:" + container.Name);
            if (container.Properties.LeaseState == LeaseState.Leased)
            {
                await container.BreakLeaseAsync(null);
            }

            await container.DeleteAsync();
        }

        Console.WriteLine();
    }
    catch (StorageException e)
    {
        Console.WriteLine(e.Message);
        Console.ReadLine();
        throw;
    }
}
```

[!INCLUDE [storage-blob-dotnet-resources](../../../includes/storage-blob-dotnet-resources.md)]

## <a name="see-also"></a>Weitere Informationen

- [Vorgang zum Erstellen eines Containers](/rest/api/storageservices/create-container)
- [Vorgang zum Löschen eines Containers](/rest/api/storageservices/delete-container)
