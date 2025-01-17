---
title: Kopieren von Daten aus einer SAP-Tabelle mithilfe von Azure Data Factory | Microsoft-Dokumentation
description: Erfahren Sie, wie Daten aus einer SAP-Tabelle mithilfe einer Kopieraktivität in einer Azure Data Factory-Pipeline in unterstützte Senkendatenspeicher kopiert werden.
services: data-factory
documentationcenter: ''
author: linda33wj
manager: craigg
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.topic: conceptual
ms.date: 08/01/2018
ms.author: jingwang
ms.openlocfilehash: 18b5b941716fd2c6664c37f9e7c1ab2a37d07a88
ms.sourcegitcommit: 85b3973b104111f536dc5eccf8026749084d8789
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 08/01/2019
ms.locfileid: "68720645"
---
# <a name="copy-data-from-an-sap-table-by-using-azure-data-factory"></a>Kopieren von Daten aus einer SAP-Tabelle mithilfe von Azure Data Factory

In diesem Artikel wird beschrieben, wie Sie die Kopieraktivität in Azure Data Factory verwenden, um Daten aus einer SAP-Tabelle zu kopieren. Weitere Informationen finden Sie im Artikel [Übersicht über die Kopieraktivität](copy-activity-overview.md).

## <a name="supported-capabilities"></a>Unterstützte Funktionen

Sie können Daten aus einer SAP-Tabelle in beliebige unterstützte Senkendatenspeicher kopieren. Eine Liste der Datenspeicher, die als Quellen oder Senken für die Kopieraktivität unterstützt werden, finden Sie in der Tabelle [Unterstützte Datenspeicher](copy-activity-overview.md#supported-data-stores-and-formats).

Dieser SAP-Tabellenconnector unterstützt insbesondere Folgendes:

- Kopieren von Daten aus einer SAP-Tabelle in:

  - SAP ERP Central Component (SAP ECC), Version 7.01 oder höher (in einem aktuellen SAP Support Package Stack, das nach 2015 veröffentlicht wurde).
  - SAP Business Warehouse (SAP BW), Version 7.01 oder höher.
  - SAP S/4HANA.
  - Andere Produkte in SAP Business Suite, Version 7.01 oder höher.

- Kopieren von Daten aus einer transparenten SAP-Tabelle, einer in einem Pool zusammengefassten Tabelle, einer gruppierten Tabelle und einer Sicht.
- Kopieren von Daten mithilfe von Standardauthentifizierung oder Secure Network Communications (SNC), falls SNC konfiguriert ist.
- Herstellen einer Verbindung mit einem SAP-Anwendungsserver oder einem SAP-Nachrichtenserver.

## <a name="prerequisites"></a>Voraussetzungen

Zum Verwenden dieses SAP-Tabellenconnectors müssen Sie folgende Schritte ausführen:

- Richten Sie eine selbstgehostete Integration Runtime (Version 3.17 oder höher) ein. Weitere Informationen finden Sie unter [Erstellen und Konfigurieren einer selbstgehosteten Integration Runtime](create-self-hosted-integration-runtime.md).

- Laden Sie den [SAP Connector für Microsoft .NET 3.0](https://support.sap.com/en/product/connectors/msnet.html) (64 Bit) von der SAP-Website herunter, und installieren Sie ihn auf dem Computer mit der selbstgehosteten Integration Runtime. Während der Installation müssen Sie im Fenster **Optionale Einrichtungsschritte** die Option **Assemblys in GAC installieren** auswählen.

  ![Installieren des SAP-Connectors für .NET](./media/connector-sap-business-warehouse-open-hub/install-sap-dotnet-connector.png)

- Der SAP-Benutzer, der im SAP-Tabellenconnector von Data Factory verwendet wird, muss über die folgenden Berechtigungen verfügen:

  - Autorisierung für die Verwendung von RFC-Zielen (Remote Function Call).
  - Berechtigungen für die Aktivität „Execute“ des Autorisierungsobjekts „S_SDSAUTH“.

## <a name="get-started"></a>Erste Schritte

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

Die folgenden Abschnitte enthalten Details zu Eigenschaften, die zum Definieren der Data Factory-Entitäten speziell für den SAP-Tabellenconnector verwendet werden.

## <a name="linked-service-properties"></a>Eigenschaften des verknüpften Diensts

Folgende Eigenschaften werden für den mit SAP BW Open Hub verknüpften Dienst unterstützt:

| Eigenschaft | BESCHREIBUNG | Erforderlich |
|:--- |:--- |:--- |
| `type` | Die `type`-Eigenschaft muss auf `SapTable` festgelegt werden. | Ja |
| `server` | Der Name des Servers, auf dem sich die SAP-Instanz befindet.<br/>Verwenden Sie ihn zum Herstellen einer Verbindung mit einem SAP-Anwendungsserver. | Nein |
| `systemNumber` | Die Systemnummer des SAP-Systems.<br/>Verwenden Sie sie zum Herstellen einer Verbindung mit einem SAP-Anwendungsserver.<br/>Zulässiger Wert: Eine zweistellige Dezimalzahl, die als Zeichenfolge angegeben wird. | Nein |
| `messageServer` | Der Hostname des SAP-Nachrichtenservers.<br/>Verwenden Sie ihn zum Herstellen einer Verbindung mit einem SAP-Nachrichtenserver. | Nein |
| `messageServerService` | Der Dienstname oder die Portnummer des Nachrichtenservers.<br/>Verwenden Sie ihn bzw. sie zum Herstellen einer Verbindung mit einem SAP-Nachrichtenserver. | Nein |
| `systemId` | Die ID des SAP-Systems, in dem sich die Tabelle befindet.<br/>Verwenden Sie sie zum Herstellen einer Verbindung mit einem SAP-Nachrichtenserver. | Nein |
| `logonGroup` | Die Anmeldegruppe für das SAP-System.<br/>Verwenden Sie sie zum Herstellen einer Verbindung mit einem SAP-Nachrichtenserver. | Nein |
| `clientId` | Die ID des Clients im SAP-System.<br/>Zulässiger Wert: Eine dreistellige Dezimalzahl, die als Zeichenfolge angegeben wird. | Ja |
| `language` | Die Sprache, die das SAP-System verwendet.<br/>Der Standardwert ist `EN`.| Nein |
| `userName` | Der Name des Benutzers, der Zugriff auf den SAP-Server hat. | Ja |
| `password` | Das Kennwort für den Benutzer. Markieren Sie dieses Feld mit dem Typ `SecureString`, um es in Data Factory sicher zu speichern, oder [verweisen Sie auf ein in Azure Key Vault gespeichertes Geheimnis](store-credentials-in-key-vault.md). | Ja |
| `sncMode` | Der SNC-Aktivierungsindikator für den Zugriff auf den SAP-Server, auf dem sich die Tabelle befindet.<br/>Verwenden Sie ihn, wenn Sie mithilfe von SNC eine Verbindung zum SAP-Server herstellen möchten.<br/>Zulässige Werte sind `0` („off“, der Standardwert) oder `1` („on“). | Nein |
| `sncMyName` | Der SNC-Name des+ Initiators für den Zugriff auf den SAP-Server, auf dem sich die Tabelle befindet.<br/>Gilt, wenn `sncMode` „on“ ist. | Nein |
| `sncPartnerName` | Der SNC-Name des Kommunikationspartners für den Zugriff auf den SAP-Server, auf dem sich die Tabelle befindet.<br/>Gilt, wenn `sncMode` „on“ ist. | Nein |
| `sncLibraryPath` | Die Bibliothek des externen Sicherheitsprodukts für den Zugriff auf den SAP-Server, auf dem sich die Tabelle befindet.<br/>Gilt, wenn `sncMode` „on“ ist. | Nein |
| `sncQop` | Die anzuwendende Ebene von „Quality of Protection für SNC“.<br/>Gilt, wenn `sncMode` „on“ ist. <br/>Zulässige Werte sind `1` (Authentifizierung), `2` (Integrität), `3` (Datenschutz), `8` (Standard), `9` (Maximum). | Nein |
| `connectVia` | Die [Integration Runtime](concepts-integration-runtime.md), die zum Herstellen einer Verbindung mit dem Datenspeicher verwendet werden soll. Eine selbstgehostete Integration Runtime ist erforderlich, wie weiter oben unter [Voraussetzungen](#prerequisites) erwähnt wurde. |Ja |

**Beispiel 1: Herstellen einer Verbindung mit einem SAP-Anwendungsserver**

```json
{
    "name": "SapTableLinkedService",
    "properties": {
        "type": "SapTable",
        "typeProperties": {
            "server": "<server name>",
            "systemNumber": "<system number>",
            "clientId": "<client ID>",
            "userName": "<SAP user>",
            "password": {
                "type": "SecureString",
                "value": "<Password for SAP user>"
            }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

### <a name="example-2-connect-to-an-sap-message-server"></a>Beispiel 2: Herstellen einer Verbindung mit einem SAP-Nachrichtenserver

```json
{
    "name": "SapTableLinkedService",
    "properties": {
        "type": "SapTable",
        "typeProperties": {
            "messageServer": "<message server name>",
            "messageServerService": "<service name or port>",
            "systemId": "<system ID>",
            "logonGroup": "<logon group>",
            "clientId": "<client ID>",
            "userName": "<SAP user>",
            "password": {
                "type": "SecureString",
                "value": "<Password for SAP user>"
            }
        },
        "connectVia": {
            "referenceName": "<name of integration runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

### <a name="example-3-connect-by-using-snc"></a>Beispiel 3: Herstellen einer Verbindung mithilfe von SNC

```json
{
    "name": "SapTableLinkedService",
    "properties": {
        "type": "SapTable",
        "typeProperties": {
            "server": "<server name>",
            "systemNumber": "<system number>",
            "clientId": "<client ID>",
            "userName": "<SAP user>",
            "password": {
                "type": "SecureString",
                "value": "<Password for SAP user>"
            },
            "sncMode": 1,
            "sncMyName": "<SNC myname>",
            "sncPartnerName": "<SNC partner name>",
            "sncLibraryPath": "<SNC library path>",
            "sncQop": "8"
        },
        "connectVia": {
            "referenceName": "<name of integration runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

## <a name="dataset-properties"></a>Dataset-Eigenschaften

Eine vollständige Liste der Abschnitte und Eigenschaften zum Definieren von Datasets finden Sie unter [Datasets.](concepts-datasets-linked-services.md) Der folgende Abschnitt enthält eine Liste der Eigenschaften, die vom SAP-Tabellen-Dataset unterstützt werden.

Wenn Sie Daten aus oder in den verknüpften SAP BW Open Hub-Dienst kopieren möchten, werden die folgenden Eigenschaften unterstützt:

| Eigenschaft | BESCHREIBUNG | Erforderlich |
|:--- |:--- |:--- |
| `type` | Die `type`-Eigenschaft muss auf `SapTableResource` festgelegt werden. | Ja |
| `tableName` | Der Name der SAP-Tabelle, aus der Daten kopiert werden sollen. | Ja |

### <a name="example"></a>Beispiel

```json
{
    "name": "SAPTableDataset",
    "properties": {
        "type": "SapTableResource",
        "typeProperties": {
            "tableName": "<SAP table name>"
        },
        "schema": [],
        "linkedServiceName": {
            "referenceName": "<SAP table linked service name>",
            "type": "LinkedServiceReference"
        }
    }
}
```

## <a name="copy-activity-properties"></a>Eigenschaften der Kopieraktivität

Eine vollständige Liste der Abschnitte und Eigenschaften zum Definieren von Aktivitäten finden Sie unter [Pipelines.](concepts-pipelines-activities.md) Der folgende Abschnitt enthält eine Liste der Eigenschaften, die von der SAP-Tabellenquelle unterstützt werden.

### <a name="sap-table-as-a-source"></a>SAP-Tabelle als eine Quelle

Beim Kopieren von Daten aus einer SAP-Tabelle werden die folgenden Eigenschaften unterstützt:

| Eigenschaft                         | BESCHREIBUNG                                                  | Erforderlich |
| :------------------------------- | :----------------------------------------------------------- | :------- |
| `type`                             | Die `type`-Eigenschaft muss auf `SapTableSource` festgelegt werden.         | Ja      |
| `rowCount`                         | Die Anzahl der Zeilen, die abgerufen werden sollen.                              | Nein       |
| `rfcTableFields`                   | Die Felder (Spalten), die aus der SAP-Tabelle kopiert werden sollen. Beispiel: `column0, column1`. | Nein       |
| `rfcTableOptions`                  | Die Optionen zum Filtern der Zeilen in einer SAP-Tabelle. Beispiel: `COLUMN0 EQ 'SOMEVALUE'`. Sehen Sie sich dazu auch die Tabelle „SAP-Abfrageoperator“ weiter unten in diesem Artikel an. | Nein       |
| `customRfcReadTableFunctionModule` | Ein benutzerdefiniertes RFC-Funktionsmodul, das zum Lesen von Daten aus einer SAP-Tabelle verwendet werden kann.<br>Mit einem benutzerdefinierten RFC-Funktionsmodul können Sie festlegen, wie die Daten aus Ihrem SAP-System abgerufen und an Data Factory zurückgegeben werden sollen. Beim benutzerdefinierten Funktionsmodul muss eine Schnittstelle (für Import, Export, Tabellen) implementiert worden sein, die mit `/SAPDS/RFC_READ_TABLE2` vergleichbar ist. Dabei handelt es sich um die von Data Factory verwendete Standardschnittstelle. | Nein       |
| `partitionOption`                  | Der Partitionsmechanismus, der zum Lesen aus einer SAP-Tabelle verwendet wird. Folgende Optionen werden unterstützt: <ul><li>`None`</li><li>`PartitionOnInt` (normale ganze Zahl oder ganzzahlige Werte mit führenden Nullen, z.B. `0000012345`)</li><li>`PartitionOnCalendarYear` (4 Ziffern im Format „JJJJ“)</li><li>`PartitionOnCalendarMonth` (6 Ziffern im Format „JJJJMM“)</li><li>`PartitionOnCalendarDate` (8 Ziffern im Format „JJJJMMTT“)</li></ul> | Nein       |
| `partitionColumnName`              | Der Name der Spalte, die zum Partitionieren der Daten verwendet wird.                | Nein       |
| `partitionUpperBound`              | Der Höchstwert der in `partitionColumnName` angegebenen Spalte, der zum Fortsetzen der Partitionierung verwendet wird. | Nein       |
| `partitionLowerBound`              | Der Mindestwert der in `partitionColumnName` angegebenen Spalte, der zum Fortsetzen der Partitionierung verwendet wird. | Nein       |
| `maxPartitionsNumber`              | Die maximale Anzahl von Partitionen, in welche die Daten aufgeteilt werden können.     | Nein       |

>[!TIP]
>Wenn Ihre SAP-Tabelle eine große Menge an Daten enthält, z.B. mehrere Milliarden Zeilen, verwenden Sie `partitionOption` und `partitionSetting`, um die Daten in kleinere Partitionen aufzuteilen. In diesem Fall werden die Daten pro Partition gelesen, und jede Datenpartition wird über einen einzelnen RFC-Aufruf von Ihrem SAP-Server abgerufen.<br/>
<br/>
>Bei dem Beispiel von `partitionOption` als `partitionOnInt` wird die Anzahl der Zeilen in jeder Partition mit dieser Formel berechnet: (Gesamtzahl der Zeilen zwischen `partitionUpperBound` und `partitionLowerBound`)/`maxPartitionsNumber`.<br/>
<br/>
>Zum parallelen Ausführen von Partitionen, um das Kopieren zu beschleunigen, wird dringend empfohlen, `maxPartitionsNumber` zu einem Vielfachen des Werts der Eigenschaft `parallelCopies` zu machen. Weitere Informationen finden Sie unter [Paralleles Kopieren](copy-activity-performance.md#parallel-copy).

In `rfcTableOptions` können Sie die folgenden allgemeinen SAP-Abfrageoperatoren zum Filtern von Zeilen verwenden:

| Operator | BESCHREIBUNG |
| :------- | :------- |
| `EQ` | Ist gleich |
| `NE` | Ungleich |
| `LT` | Kleiner als |
| `LE` | Kleiner als oder gleich |
| `GT` | Größer als |
| `GE` | Größer oder gleich |
| `LIKE` | Wie in `LIKE 'Emma%'` |

### <a name="example"></a>Beispiel

```json
"activities":[
    {
        "name": "CopyFromSAPTable",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<SAP table input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "SapTableSource",
                "partitionOption": "PartitionOnInt",
                "partitionSettings": {
                     "partitionColumnName": "<partition column name>",
                     "partitionUpperBound": "2000",
                     "partitionLowerBound": "1",
                     "maxPartitionsNumber": 500
                 }
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

## <a name="data-type-mappings-for-an-sap-table"></a>Datentypzuordnungen für eine SAP-Tabelle

Wenn Sie Daten aus einer SAP-Tabelle kopieren, werden die folgenden Zuordnungen von den SAP-Tabellendatentypen zu den Azure Data Factory-Zwischendatentypen verwendet. Informationen dazu, wie die Kopieraktivität das Quellschema und den Datentyp zur Senke zuordnet, finden Sie unter [Schema- und Datentypzuordnungen](copy-activity-schema-and-type-mapping.md).

| SAP ABAP-Typ | Data Factory-Zwischendatentyp |
|:--- |:--- |
| `C` (Zeichenfolge) | `String` |
| `I` (Integer) | `Int32` |
| `F` (Float) | `Double` |
| `D` (Date) | `String` |
| `T` (Time) | `String` |
| `P` (BCD Packed, Currency, Decimal, Qty) | `Decimal` |
| `N` (Numeric) | `String` |
| `X` (Binary und Raw) | `String` |

## <a name="next-steps"></a>Nächste Schritte

Eine Liste der Datenspeicher, die als Quellen und Senken für die Kopieraktivität in Azure Data Factory unterstützt werden, finden Sie unter [Unterstützte Datenspeicher](copy-activity-overview.md#supported-data-stores-and-formats).
