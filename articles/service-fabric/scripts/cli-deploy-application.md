---
title: Azure Service Fabric-CLI-Skript (sfctl) – Bereitstellungsbeispiel
description: Bereitstellen einer Anwendung in einem Azure Service Fabric-Cluster mithilfe der Azure Service Fabric-CLI
services: service-fabric
documentationcenter: ''
author: athinanthny
manager: chackdan
editor: ''
tags: azure-service-management
ms.assetid: ''
ms.service: service-fabric
ms.workload: multiple
ms.devlang: na
ms.topic: sample
ms.date: 04/16/2018
ms.author: atsenthi
ms.custom: mvc
ms.openlocfilehash: d25ed8d27c13e20641d1495c47fdcf6ce2edeb75
ms.sourcegitcommit: fe6b91c5f287078e4b4c7356e0fa597e78361abe
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 07/29/2019
ms.locfileid: "68592209"
---
# <a name="deploy-an-application-to-a-service-fabric-cluster"></a>Bereitstellen einer Anwendung in einem Service Fabric-Cluster

Dieses Beispielskript kopiert ein Anwendungspaket in einen Clusterimagespeicher, registriert den Anwendungstyp im Cluster und erstellt eine Anwendungsinstanz aus dem Anwendungstyp. Alle Standarddienste werden ebenfalls zu diesem Zeitpunkt erstellt.

Installieren Sie ggf. die [Service Fabric-CLI](../service-fabric-cli.md).

## <a name="sample-script"></a>Beispielskript

[!code-sh[main](../../../cli_scripts/service-fabric/deploy-application/deploy-application.sh "Deploy an application to a cluster")]

## <a name="clean-up-deployment"></a>Bereinigen der Bereitstellung

Nach Abschluss kann die Anwendung mit dem [Entfernungsskript](cli-remove-application.md) entfernt werden. Das Entfernungsskript löscht die Anwendungsinstanz, hebt die Registrierung des Anwendungstyps auf und löscht das Anwendungspaket aus dem Imagespeicher.

## <a name="next-steps"></a>Nächste Schritte

Weitere Informationen finden Sie in der [Dokumentation der Service Fabric-Befehlszeilenschnittstelle](../service-fabric-cli.md).

Zusätzliche Service Fabric-CLI-Beispiele für Azure Service Fabric finden Sie unter [Azure PowerShell-Beispiele](../samples-cli.md).
