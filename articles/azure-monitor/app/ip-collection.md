---
title: Sammlung von Azure Application Insights-IP-Adressen | Microsoft-Dokumentation
description: Grundlegendes zur Handhabung von IP-Adressen und der Geolocation mit Azure Application Insights
services: application-insights
author: mrbullwinkle
manager: carmonm
ms.assetid: 0e3b103c-6e2a-4634-9e8c-8b85cf5e9c84
ms.service: application-insights
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 07/24/2019
ms.author: mbullwin
ms.openlocfilehash: 4c60cb78c01d7e18801cbe43c8b767f622ef4b39
ms.sourcegitcommit: c72ddb56b5657b2adeb3c4608c3d4c56e3421f2c
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 07/24/2019
ms.locfileid: "68473019"
---
# <a name="geolocation-and-ip-address-handling"></a>Geolocation und Verarbeitung von IP-Adressen

In diesem Artikel wird erläutert, wie die Geolocationsuche und die IP-Adressen-Behandlung in Application Insights zusammen mit der Vorgehensweise zum Ändern des Standardverhaltens erfolgen.

## <a name="default-behavior"></a>Standardverhalten

IP-Adressen werden standardmäßig temporär gesammelt, aber nicht in Application Insights gespeichert. Der Standardprozess verläuft wie folgt:

IP-Adressen werden als Teil der Telemetriedaten an Application Insights gesendet. Wenn die IP-Adresse den Erfassungsendpunkt in Azure erreicht hat, wird sie zum Ausführen einer Geolocationsuche mithilfe von [GeoLite2 from MaxMind](https://dev.maxmind.com/geoip/geoip2/geolite2/) verwendet. Die Ergebnisse dieser Suche werden verwendet, um die Felder `client_City`, `client_StateOrProvince` und `client_CountryOrRegion` aufzufüllen. An diesem Punkt wird die IP-Adresse verworfen, und `0.0.0.0` wird in das Feld `client_IP` geschrieben.

* Browsertelemetrie: Die IP-Adresse des Absenders wird vorübergehend erfasst. Die IP-Adresse wird vom Erfassungsendpunkt berechnet.
* Servertelemetrie: Die Client-IP-Adresse wird vorübergehend vom Application Insights-Modul gesammelt. Sie wird nicht gesammelt, wenn `X-Forwarded-For` festgelegt ist.

Dieses Verhalten ist beabsichtigt und soll die unnötige Erfassung persönlicher Daten vermeiden. Die Erfassung persönlicher Daten sollte, wenn möglich, vermieden werden. 

## <a name="overriding-default-behavior"></a>Überschreiben des Standardverhaltens

Obwohl das Standardverhalten das Erfassen persönlicher Daten minimieren soll, bieten wir weiterhin die Flexibilität, IP-Adressdaten zu erfassen und zu speichern. Bevor Sie sich zur Speicherung persönlicher Daten wie IP-Adressen entscheiden, sollten Sie unbedingt sicherstellen, dass dadurch keine Complianceanforderungen oder lokale Bestimmungen verletzt werden, denen Sie möglicherweise unterliegen. Weitere Informationen zur Behandlung persönlicher Daten in Application Insights finden Sie im [Leitfaden zu persönlichen Daten](https://docs.microsoft.com/azure/azure-monitor/platform/personal-data-mgmt).

## <a name="storing-partial-ip-address-data"></a>Speichern partieller IP-Adressdaten

Die Eigenschaft `DisableIpMasking` der Application Insights-Komponente muss auf `true` festgelegt sein, um das Erfassen und Speichern partieller IP-Adressen zu aktivieren. Diese Eigenschaft kann entweder durch Azure Resource Manager-Vorlagen oder durch Aufrufen der REST-API festgelegt werden. Beim Aufzeichnen der IP-Adressen wird das letzte Oktett zurückgesetzt.


### <a name="azure-resource-manager-template"></a>Azure Resource Manager-Vorlage

```json
{
       "id": "/subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/microsoft.insights/components/<resource-name>",
       "name": "<resource-name>",
       "type": "microsoft.insights/components",
       "location": "westcentralus",
       "tags": {
              
       },
       "kind": "web",
       "properties": {
              "Application_Type": "web",
              "Flow_Type": "Redfield",
              "Request_Source": "IbizaAIExtension",
              // ...
              "DisableIpMasking": true
       }
}
```

### <a name="portal"></a>Portal 

Wenn Sie nur das Verhalten für eine einzelne Application Insights-Ressource ändern müssen, erreichen Sie dies am einfachsten über das Azure-Portal.  

1. Navigieren Sie zu Ihrer Application Insights-Ressource > **Einstellungen** > **Vorlage exportieren**. 

    ![Vorlage exportieren](media/ip-collection/export-template.png)

2. Klicken Sie auf **Bereitstellen**.

    ![Schaltfläche „Bereitstellen“ rot hervorgehoben](media/ip-collection/deploy.png)

3. Klicken Sie auf **Vorlage bearbeiten**. (Wenn Ihre Vorlage zusätzliche Eigenschaften oder Ressourcen enthält, die in dieser Beispielvorlage nicht angezeigt werden, sollten Sie im weiteren Verlauf sicherstellen, dass alle Ressourcen die Vorlagenbereitstellung als inkrementelle Änderung/Aktualisierung akzeptieren.)

    ![Vorlage bearbeiten](media/ip-collection/edit-template.png)

4. Nehmen Sie die folgenden Änderungen an der JSON-Datei für Ihre Ressource vor, und klicken Sie dann auf **Speichern**:

    ![Screenshot: Einfügen eines Kommas nach "IbizaAIExtension" und einer neuen Zeile darunter mit "DisableIpMasking": true](media/ip-collection/save.png)

    > [!NOTE]
    > Wenn eine Fehlermeldung mit folgendem Text angezeigt wird: _The resource group is in a location that is not supported by one or more resources in the template. Please choose a different resource group._ (Die Ressourcengruppe befindet sich an einem Speicherort, der von keiner Ressource in der Vorlage unterstützt wird. Wählen Sie eine andere Ressourcengruppe). Wählen Sie vorübergehend eine andere Ressourcengruppe aus der Dropdownliste aus, und wählen Sie dann erneut die ursprüngliche Ressourcengruppe aus, um den Fehler zu beheben.

5. Klicken Sie auf **Ich stimme zu** > **Kaufen**. 

    ![Vorlage bearbeiten](media/ip-collection/purchase.png)

    In diesem Fall wird nichts neues gekauft, sondern lediglich die Konfiguration der vorhandenen Application Insights-Ressource aktualisiert.

6. Nachdem die Bereitstellung abgeschlossen wurde, werden mit den ersten drei Oktetten, die mit der IP-Adresse aufgefüllt und bei denen das letzte Oktett zurückgesetzt wurde, neue Telemetriedaten aufgezeichnet.

    Wenn Sie die Vorlage erneut auswählen und bearbeiten, wird nur die Standardvorlage und nicht die neu hinzugefügte Eigenschaft und ihr zugehöriger Wert angezeigt. Wenn die IP-Adressdaten nicht angezeigt werden und Sie bestätigen möchten, dass `"DisableIpMasking": true` festgelegt ist, führen Sie folgenden PowerShell-Befehl aus: (Ersetzen Sie `Fabrikam-dev` durch die entsprechende Ressource und den Ressourcengruppennamen.)
    
    ```powershell
    # If you aren't using the cloud shell you will need to connect to your Azure account
    # Connect-AzAccount 
    $AppInsights = Get-AzResource -Name 'Fabrikam-dev' -ResourceType 'microsoft.insights/components' -ResourceGroupName 'Fabrikam-dev'
    $AppInsights.Properties
    ```
    
    Daraufhin wird eine Liste mit den Eigenschaften zurückgegeben. Eine der Eigenschaften sollte `DisableIpMasking: true` beinhalten. Wenn Sie vor dem Bereitstellen der neuen Eigenschaft mit Azure Resource Manager den PowerShell-Befehl ausführen, ist die Eigenschaft nicht vorhanden.

### <a name="rest-api"></a>REST-API

Die [REST-API](https://docs.microsoft.com/rest/api/azure/)-Payload, um die gleichen Änderungen vorzunehmen, sieht wie folgt aus:

```
PATCH https://management.azure.com/subscriptions/<sub-id>/resourceGroups/<rg-name>/providers/microsoft.insights/components/<resource-name>?api-version=2018-05-01-preview HTTP/1.1
Host: management.azure.com
Authorization: AUTH_TOKEN
Content-Type: application/json
Content-Length: 54

{
       "location": "<resource location>",
       "kind": "web",
       "properties": {
              "Application_Type": "web",
              "DisableIpMasking": true
       }
}
```

## <a name="telemetry-initializer"></a>Telemetrieinitialisierer

Wenn Sie die gesamte IP-Adresse und nicht nur die ersten drei Oktette aufzeichnen müssen, können Sie einen [Telemetrieinitialisierer](https://docs.microsoft.com/azure/azure-monitor/app/api-filtering-sampling#add-properties-itelemetryinitializer) verwenden, um die IP-Adresse in ein nicht maskiertes benutzerdefiniertes Feld zu kopieren.

### <a name="aspnetaspnet-core"></a>ASP.NET/ASP.NET Core

```csharp
using Microsoft.ApplicationInsights.Channel;
using Microsoft.ApplicationInsights.Extensibility;

namespace MyWebApp
{
    public class CloneIPAddress : ITelemetryInitializer
    {
        public void Initialize(ITelemetry telemetry)
        {
            if(!string.IsNullOrEmpty(telemetry.Context.Location.Ip))
            {
                telemetry.Context.Properties["client-ip"] = telemetry.Context.Location.Ip;
            }
        }
    }

}
```

### <a name="enable-telemetry-initializer-for-aspnet"></a>Aktivieren des Telemetrieinitialisierers für ASP.NET

```csharp
using Microsoft.ApplicationInsights.Extensibility;


namespace MyWebApp
{
    public class MvcApplication : System.Web.HttpApplication
    {
        protected void Application_Start()
        {
              //Enable your telemetry initializer:
              TelemetryConfiguration.Active.TelemetryInitializers.Add(new CloneIPAddress());
        }
    }
}

```

### <a name="enable-telemetry-initializer-for-aspnet-core"></a>Aktivieren des Telemetrieinitialisierers für ASP.NET Core

Sie können den Telemetrieinitialisierer für ASP.NET Core auf die gleiche Weise wie für ASP.NET erstellen, müssen zum Aktivieren des Initialisierers aber das folgende Beispiel als Verweis verwenden:

```csharp
 using Microsoft.ApplicationInsights.Extensibility;
 using CustomInitializer.Telemetry;
 public void ConfigureServices(IServiceCollection services)
{
    services.AddSingleton<ITelemetryInitializer, CloneIPAddress>();
}
```

### <a name="nodejs"></a>Node.js

```javascript
appInsights.defaultClient.addTelemetryProcessor((envelope) => {
    const baseData = envelope.data.baseData;
    if (appInsights.Contracts.domainSupportsProperties(baseData)) {
        const ipAddress = envelope.tags[appInsights.defaultClient.context.keys.locationIp];
        if (ipAddress) {
            baseData.properties["client-ip"] = ipAddress;
        }
    }
});
```

### <a name="client-side-javascript"></a>Clientseitiges JavaScript

Im Gegensatz zu serverseitigen SDKs berechnet das clientseitige JavaScript-SDK keine IP-Adresse. Die IP-Adressenberechnung für clientseitige Telemetriedaten wird standardmäßig am Erfassungsendpunkt in Azure bei der Ankunft von Telemetriedaten ausgeführt. Dies bedeutet, dass beim Senden clientseitiger Daten an einen Proxy und Weiterleiten an den Erfassungsendpunkt die IP-Adressenberechnung die IP-Adresse des Proxys und nicht des Clients angezeigt hat. Wenn kein Proxy verwendet wird, sollte dies kein Problem sein.

Wenn Sie IP-Adressen direkt auf der Clientseite berechnen möchten, müssen Sie eine eigene benutzerdefinierte Logik hinzufügen, um diese Berechnung auszuführen und das Ergebnis zum Festlegen des Tags `ai.location.ip` verwenden. Wenn `ai.location.ip` festgelegt ist, wird die IP-Adressenberechnung nicht vom Erfassungsendpunkt durchgeführt und die angegebene IP-Adresse wird berücksichtigt und zum Durchführen der Geolocationsuche verwendet. In diesem Szenario wird die IP-Adresse weiterhin standardmäßig zurückgesetzt. 

Sie können einen Telemetrieinitialisierer verwenden, der die von Ihnen in `ai.location.ip` bereitgestellten IP-Adressdaten in ein separates benutzerdefiniertes Feld kopiert, um die gesamte von der benutzerdefinierten Logik berechnete IP-Adresse beizubehalten. Im Gegensatz zu den serverseitigen SDKs wird, ohne sich auf Bibliotheken von Drittanbietern oder eine eigene benutzerdefinierte clientseitige IP-Adressensammlungslogik zu verlassen, die IP-Adresse vom clientseitigen SDK nicht berechnet.    


```javascript
appInsights.addTelemetryInitializer((item) => {
    const ipAddress = item.tags && item.tags["ai.location.ip"];
    if (ipAddress) {
        item.baseData.properties = {
            ...item.baseData.properties,
            "client-ip": ipAddress
        };
    }
});

```  

### <a name="view-the-results-of-your-telemetry-initializer"></a>Anzeigen der Ergebnisse des Telemetrieinitialisierers

Wenn Sie dann neuen Datenverkehr für Ihre Website auslösen und ungefähr 2 bis 5 Minuten für die Erfassung warten, können Sie eine Kusto-Abfrage ausführen, um zu überprüfen, ob die IP-Adressensammlung funktioniert:

```kusto
requests
| where timestamp > ago(1h) 
| project appName, operation_Name, url, resultCode, client_IP, customDimensions.["client-ip"]
```

Die neu gesammelten IP-Adressen sollten in der Spalte `customDimensions_client-ip` angezeigt werden. Bei der Standardspalte `client-ip` werden auch weiterhin alle vier Oktette entweder zurückgesetzt, oder es werden nur die ersten drei Oktette angezeigt, je nachdem, wie die IP-Adressensammlung auf der Komponentenebene konfiguriert ist. Wenn Sie nach dem Implementieren des Telemetrieinitialisierers lokal testen und der für `customDimensions_client-ip` angezeigte Wert `::1` ist, ist dies das zu erwartende Verhalten. `::1` stellt die Loopbackadresse in IPv6 dar. Dies entspricht `127.0.01` in IPv4 und ist das Ergebnis, das beim Testen von Localhost angezeigt wird.

## <a name="next-steps"></a>Nächste Schritte

* Weitere Informationen zur [Sammlung persönlicher Daten](https://docs.microsoft.com/azure/azure-monitor/platform/personal-data-mgmt) finden Sie in Application Insights.

* Erfahren Sie mehr über die Funktionsweise der [IP-Adressensammlung](https://apmtips.com/blog/2016/07/05/client-ip-address/) in Application Insights. (Dies ist ein älterer externer Blogbeitrag von einem unserer Techniker. Zwar wurde der Beitrag vor der Implementierung des aktuellen Standardverhaltens verfasst, bei dem die IP-Adresse als `0.0.0.0` aufgezeichnet wird, allerdings wird die Funktionsweise des integrierten `ClientIpHeaderTelemetryInitializer` ausführlicher erklärt.)