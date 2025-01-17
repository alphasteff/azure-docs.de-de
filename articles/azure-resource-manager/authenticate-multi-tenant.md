---
title: Mandantenübergreifende Authentifizierung – Azure Resource Manager
description: Beschreibt, wie Azure Resource Manager Authentifizierungsanforderungen mandantenübergreifend behandelt.
author: tfitzmac
ms.service: azure-resource-manager
ms.topic: conceptual
ms.date: 01/07/2019
ms.author: tomfitz
ms.openlocfilehash: 5370b9b6d6a8bee82f8feca6dbcbcd78a4c12193
ms.sourcegitcommit: b7a44709a0f82974578126f25abee27399f0887f
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 06/18/2019
ms.locfileid: "67205606"
---
# <a name="authenticate-requests-across-tenants"></a>Mandantenübergreifende Authentifizierung von Anforderungen

Beim Erstellen einer mandantenfähigen Anwendung müssen Sie möglicherweise Authentifizierungsanforderungen für Ressourcen, die sich in verschiedenen Mandanten befinden, bearbeiten. Ein häufiges Szenario ist, wenn ein virtueller Computer in einem Mandanten einem virtuellen Netzwerk in einem anderen Mandanten hinzugefügt werden muss. Der Azure Resource Manager stellt einen Headerwert für die Speicherung von zusätzlichen Token zur Verfügung, um die Anforderungen an verschiedene Mandanten zu authentifizieren.

## <a name="header-values-for-authentication"></a>Headerwerte für die Authentifizierung

Die Anforderung hat die folgenden Headerwerte für die Authentifizierung:

| Headername | BESCHREIBUNG | Beispielwert |
| ----------- | ----------- | ------------ |
| Authorization | Primäres Token | Bearer &lt;primary-token&gt; |
| x-ms-authorization-auxiliary | Zusätzliche Token | Bearer &lt;auxiliary-token1&gt;; EncryptedBearer &lt;auxiliary-token2&gt;; Bearer &lt;auxiliary-token3&gt; |

Der zusätzliche Header kann bis zu drei zusätzliche Token enthalten. 

Rufen Sie im Code Ihrer mehrmandantenfähigen App den Authentifizierungstoken für andere Mandanten ab und speichern Sie ihn in den Hilfsheadern. Alle Token müssen vom gleichen Benutzer oder der gleichen Anwendung stammen. Der Benutzer oder die Anwendung muss als Gast zu den anderen Mandanten eingeladen worden sein.

## <a name="processing-the-request"></a>Verarbeiten der Anforderung

Wenn Ihre App eine Anforderung an den Resource Manager sendet, wird diese unter der Identität des primären Tokens ausgeführt. Das primäre Token muss gültig und nicht abgelaufen sein. Dieses Token muss von einem Mandanten stammen, der das Abonnement verwalten kann.

Wenn die Anforderung auf eine Ressource von einem anderen Mandanten verweist, überprüft der Resource Manager die zusätzlichen Token, um festzustellen, ob die Anforderung bearbeitet werden kann. Alle zusätzliche Token im Header müssen gültig und nicht abgelaufen sein. Wenn Token abgelaufen sind, gibt Resource Manager Antwortcode 401 zurück. Die Antwort enthält die Client-ID und die Mandanten-ID des ungültigen Token. Wenn der Hilfsheader ein gültiges Token für den Mandanten enthält, wird die mandantenübergreifende Anforderung bearbeitet.

## <a name="next-steps"></a>Nächste Schritte
* Weitere Informationen zum Senden von Authentifizierungsanforderungen mit den Azure Resource Manager-APIs finden Sie unter [Verwenden der Resource Manager-Authentifizierungs-API für den Zugriff auf Abonnements](resource-manager-api-authentication.md).
* Weitere Informationen zu Token finden Sie unter [Azure Active Directory-Zugriffstoken](/azure/active-directory/develop/access-tokens).
