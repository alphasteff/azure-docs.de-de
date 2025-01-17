---
title: Widerrufen eines Freigabeabonnements in Azure Data Share (Vorschauversion)
description: Aufheben einer Abonnementfreigabe
author: madams0013
ms.service: data-share
ms.topic: conceptual
ms.date: 07/10/2019
ms.author: t-maadam
ms.openlocfilehash: fb5572aee513172d06b5a75c2d3b11885081f792
ms.sourcegitcommit: de47a27defce58b10ef998e8991a2294175d2098
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 07/15/2019
ms.locfileid: "67877384"
---
# <a name="how-to-revoke-a-consumers-share-subscription-in-azure-data-share-preview"></a>Widerrufen eines Freigabeabonnements eines Consumers in Azure Data Share (Vorschauversion)

In diesem Artikel wird erläutert, wie ein Freigabeabonnement für einen oder mehrere Consumer über Azure Data Share (Vorschauversion) widerrufen wird. Hierdurch wird der Consumer daran gehindert, weitere Momentaufnahmen auszulösen. Wenn der Consumer noch keine Momentaufnahme ausgelöst hat, werden die Daten nach dem Aufheben des Freigabeabonnements nicht mehr empfangen. Wenn zuvor eine Momentaufnahme ausgelöst wurde, verbleiben die letzten Daten im Konto des Consumers.

## <a name="navigate-to-a-sent-data-share"></a>Navigieren zu einer gesendeten Datenfreigabe

Navigieren Sie in Azure Data Share (Vorschauversion) zur gesendeten Freigabe, und klicken Sie auf die Registerkarte **Freigabeabonnements**.

![Aufheben eines Freigabeabonnements](./media/how-to/how-to-revoke-share-subscription/revoke-share-subscription.png) 

Aktivieren Sie die Felder neben den Empfängern, deren Freigabeabonnements Sie löschen möchten, und klicken Sie auf **Widerrufen**. Der Consumer erhält für seine Daten keine Aktualisierungen mehr.

## <a name="next-steps"></a>Nächste Schritte
Weitere Informationen zum [Überwachen Ihrer Datenfreigaben](how-to-monitor.md).