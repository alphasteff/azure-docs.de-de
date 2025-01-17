---
title: Planen eines virtuellen Netzwerks für Azure HDInsight
description: Erfahren Sie, wie Sie die Bereitstellung von Azure Virtual Network planen, um HDInsight mit anderen Cloudressourcen oder Ressourcen in Ihrem Rechenzentrum zu verbinden.
author: hrasheed-msft
ms.author: hrasheed
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 07/23/2019
ms.openlocfilehash: 9906fe5de9c24f1b1a8c3f713fa772e56ed4e13f
ms.sourcegitcommit: a874064e903f845d755abffdb5eac4868b390de7
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 07/24/2019
ms.locfileid: "68441958"
---
# <a name="plan-a-virtual-network-for-azure-hdinsight"></a>Planen eines virtuellen Netzwerks für Azure HDInsight

Dieser Artikel enthält Hintergrundinformationen zur Verwendung von [Azure Virtual Network](../virtual-network/virtual-networks-overview.md) mit Azure HDInsight. Außerdem werden Entwurfs- und Implementierungsentscheidungen erörtert, die vor der Implementierung eines virtuellen Netzwerks für Ihren HDInsight-Cluster getroffen werden müssen. Nachdem die Planungsphase abgeschlossen ist, können Sie mit dem [Create virtual networks for Azure HDInsight clusters (Erstellen von virtuellen Netzwerken für Azure HDInsight-Cluster)](hdinsight-create-virtual-network.md) fortfahren. Weitere Informationen zu den IP-Adressen für die HDInsight-Verwaltung, die für die ordnungsgemäße Konfiguration von Netzwerksicherheitsgruppen und benutzerdefinierten Routen erforderlich sind, finden Sie unter [HDInsight management IP addresses (IP-Adressen für die HDInsight-Verwaltung)](hdinsight-management-ip-addresses.md).

Die Nutzung eines Azure Virtual Network ermöglicht die folgenden Szenarien:

* Herstellen einer Verbindung mit HDInsight direkt über ein lokales Netzwerk
* Herstellen einer Verbindung für HDInsight mit Datenspeichern in einem Azure Virtual Network
* Direktes Zuweisen von [Apache Hadoop](https://hadoop.apache.org/)-Diensten, die nicht öffentlich über das Internet verfügbar sind. Zum Beispiel [Apache Kafka](https://kafka.apache.org/)-APIs oder die [Apache HBase](https://hbase.apache.org/)-Java-API.

> [!IMPORTANT]
> Wenn Sie einen HDInsight-Cluster in einem VNET erstellen, werden mehrere Netzwerkressourcen erstellt, z. B. NICs und Lastenausgleichsmodule. Löschen Sie diese Netzwerkressourcen **nicht**, da sie benötigt werden, damit Ihr Cluster mit dem VNET ordnungsgemäß funktioniert.
>
> Ab dem 28. Februar 2019 werden die Netzwerkressourcen (wie Netzwerkschnittstellenkarten, Lastenausgleichsmodule usw.) für in einem virtuellen Netzwerk NEU erstellte HDInsight-Cluster in der gleichen HDInsight-Clusterressourcengruppe bereitgestellt. Zuvor wurden diese Ressourcen in der Ressourcengruppe des virtuellen Netzwerks bereitgestellt. Für derzeit ausgeführte Cluster und Cluster, die ohne virtuelles Netzwerk erstellt wurden, ändert sich nichts.

## <a name="planning"></a>Planung

Sie müssen die folgenden Fragen beantworten, wenn Sie die Installation von HDInsight in einem virtuellen Netzwerk planen:

* Müssen Sie HDInsight in einem vorhandenen virtuellen Netzwerk installieren? Oder erstellen Sie ein neues Netzwerk?

    Falls Sie ein vorhandenes virtuelles Netzwerk verwenden, müssen Sie unter Umständen die Netzwerkkonfiguration ändern, bevor Sie HDInsight installieren können. Weitere Informationen finden Sie im Abschnitt [Hinzufügen von HDInsight zu einem vorhandenen virtuellen Netzwerk](#existingvnet).

* Möchten Sie das virtuelle Netzwerk mit HDInsight einem anderen virtuellen Netzwerk oder Ihrem lokalen Netzwerk hinzufügen?

    Um Ressourcen auf einfache Weise netzwerkübergreifend verwenden zu können, müssen Sie ggf. ein benutzerdefiniertes DNS erstellen und die DNS-Weiterleitung konfigurieren. Weitere Informationen finden Sie im Abschnitt [Verbinden von mehreren Netzwerken](#multinet).

* Möchten Sie ein- oder ausgehenden Datenverkehr für HDInsight beschränken oder umleiten?

    HDInsight muss über eine uneingeschränkte Kommunikation mit bestimmten IP-Adressen im Azure-Rechenzentrum verfügen. Es gibt auch mehrere Ports, für die die Clientkommunikation durch Firewalls zugelassen sein muss. Weitere Informationen finden Sie im Abschnitt [Steuern des Netzwerkdatenverkehrs](#networktraffic).

## <a id="existingvnet"></a>Hinzufügen von HDInsight zu einem vorhandenen virtuellen Netzwerk

Führen Sie die Schritte in diesem Abschnitt aus, um zu erfahren, wie Sie einen neuen HDInsight-Cluster einem vorhandenen Azure Virtual Network hinzufügen.

> [!NOTE]  
> Ein vorhandener HDInsight-Cluster kann in einem virtuellen Netzwerk nicht hinzugefügt werden.

1. Verwenden Sie das klassische oder das Resource Manager-Bereitstellungsmodell für das virtuelle Netzwerk?

    Für HDInsight 3.4 und höher ist ein virtuelles Resource Manager-Netzwerk erforderlich. In früheren Versionen von HDInsight war ein klassisches virtuelles Netzwerk erforderlich.

    Falls Ihr vorhandenes Netzwerk ein klassisches virtuelles Netzwerk ist, müssen Sie ein virtuelles Resource Manager-Netzwerk erstellen und die beiden Netzwerke dann verbinden. [Herstellen einer Verbindung zwischen klassischen VNets und neuen VNets](../vpn-gateway/vpn-gateway-connect-different-deployment-models-portal.md)

    Nach dem Verknüpfen kann die Installation von HDInsight im Resource Manager-Netzwerk mit Ressourcen im klassischen Netzwerk interagieren.

2. Verwenden Sie Netzwerksicherheitsgruppen, benutzerdefinierte Routen oder virtuelle Network Appliances, um Datenverkehr einzuschränken, der in das bzw. aus dem virtuellen Netzwerk fließt?

    Als verwalteter Dienst ist für HDInsight der uneingeschränkte Zugriff auf mehrere IP-Adressen im Azure-Rechenzentrum erforderlich. Aktualisieren Sie alle vorhandenen Netzwerksicherheitsgruppen oder benutzerdefinierten Routen, um die Kommunikation mit diesen IP-Adressen zuzulassen.
    
    HDInsight hostet mehrere Dienste, für die viele verschiedene Ports verwendet werden. Verhindern Sie, dass der Datenverkehr für diesen Port blockiert wird. Eine Liste mit Ports, die für Firewalls von virtuellen Geräten zugelassen werden müssen, finden Sie im Abschnitt „Sicherheit“.
    
    Verwenden Sie die folgenden Azure PowerShell- oder Azure CLI-Befehle, um Ihre vorhandene Sicherheitskonfiguration zu ermitteln:

    * Netzwerksicherheitsgruppen

        Ersetzen Sie `RESOURCEGROUP` durch den Namen der Ressourcengruppe, die das virtuelle Netzwerk enthält, und geben Sie dann diesen Befehl ein:
    
        ```powershell
        Get-AzNetworkSecurityGroup -ResourceGroupName  "RESOURCEGROUP"
        ```
    
        ```azurecli
        az network nsg list --resource-group RESOURCEGROUP
        ```

        Weitere Informationen finden Sie im Dokument zur [Problembehandlung bei Netzwerksicherheitsgruppen](../virtual-network/diagnose-network-traffic-filter-problem.md).

        > [!IMPORTANT]  
        > Netzwerksicherheitsgruppen-Regeln werden der Reihenfolge nach basierend auf der Regelpriorität angewendet. Die erste Regel, die mit dem Datenverkehrsmuster übereinstimmt, wird angewendet, und auf diesen Datenverkehr werden dann keine anderen Regeln angewendet. Sortieren Sie die Regeln von „weniger strikt“ bis „strikt“. Weitere Informationen finden Sie im Dokument [Filtern des Netzwerkdatenverkehrs mit Netzwerksicherheitsgruppen](../virtual-network/security-overview.md).

    * Benutzerdefinierte Routen

        Ersetzen Sie `RESOURCEGROUP` durch den Namen der Ressourcengruppe, die das virtuelle Netzwerk enthält, und geben Sie dann diesen Befehl ein:

        ```powershell
        Get-AzRouteTable -ResourceGroupName "RESOURCEGROUP"
        ```

        ```azurecli
        az network route-table list --resource-group RESOURCEGROUP
        ```

        Weitere Informationen finden Sie im Dokument [Problembehandlung bei Routen](../virtual-network/diagnose-network-routing-problem.md).

3. Erstellen Sie einen HDInsight-Cluster, und wählen Sie während der Konfiguration die virtuelle Azure-Netzwerk-Instanz aus. Den Vorgang zur Clustererstellung können Sie anhand der Schritte in den folgenden Dokumenten nachvollziehen:

    * [Erstellen von Linux-basierten Clustern in HDInsight mithilfe des Azure-Portals](hdinsight-hadoop-create-linux-clusters-portal.md)
    * [Erstellen von Linux-basierten Clustern in HDInsight mit Azure PowerShell](hdinsight-hadoop-create-linux-clusters-azure-powershell.md)
    * [Erstellen von HDInsight-Clustern mit der klassischen Azure CLI](hdinsight-hadoop-create-linux-clusters-azure-cli.md)
    * [Erstellen von Hadoop-Clustern in HDInsight mit Azure Resource Manager-Vorlagen](hdinsight-hadoop-create-linux-clusters-arm-templates.md)

   > [!IMPORTANT]  
   > Das Hinzufügen von HDInsight zu einem virtuellen Netzwerk ist ein optionaler Konfigurationsschritt. Achten Sie darauf, beim Konfigurieren des Clusters das virtuelle Netzwerk auszuwählen.

## <a id="multinet"></a>Verbinden von mehreren Netzwerken

Die größte Herausforderung bei der Konfiguration von mehreren Netzwerken ist die Namensauflösung zwischen den Netzwerken.

Azure ermöglicht die Namensauflösung für Azure-Dienste, die in einem virtuellen Netzwerk installiert sind. Diese integrierte Namensauflösung erlaubt HDInsight das Herstellen einer Verbindung mit den folgenden Ressourcen, indem ein vollqualifizierter Domänenname (FQDN) verwendet wird:

* Alle Ressourcen, die im Internet verfügbar sind. Beispiel: „microsoft.com“, „windowsupdate.com“.

* Alle Ressourcen, die sich in demselben Azure Virtual Network befinden, indem der __interne DNS-Name__ der Ressource verwendet wird. Beim Verwenden der Standardnamensauflösung sind dies Beispiele für interne DNS-Namen, die HDInsight-Workerknoten zugewiesen werden:

  * wn0-hdinsi.0owcbllr5hze3hxdja3mqlrhhe.ex.internal.cloudapp.net
  * wn2-hdinsi.0owcbllr5hze3hxdja3mqlrhhe.ex.internal.cloudapp.net

    Beide Knoten können direkt miteinander und mit anderen Knoten in HDInsight kommunizieren, indem interne DNS-Namen verwendet werden.

Bei der Standardnamensauflösung ist es für HDInsight __nicht__ zulässig, die Namen von Ressourcen in Netzwerken aufzulösen, die mit dem virtuellen Netzwerk verknüpft sind. Beispielsweise ist es eine gängige Vorgehensweise, Ihr lokales Netzwerk mit dem virtuellen Netzwerk zu verknüpfen. Allein mit der Standardnamensauflösung kann HDInsight nicht anhand des Namens auf Ressourcen im lokalen Netzwerk zugreifen. Das Gegenteil gilt ebenfalls: Ressourcen in Ihrem lokalen Netzwerk können anhand des Namens nicht auf Ressourcen im virtuellen Netzwerk zugreifen.

> [!WARNING]  
> Sie müssen den benutzerdefinierten DNS-Server erstellen und das virtuelle Netzwerk für seine Verwendung konfigurieren, bevor Sie den HDInsight-Cluster erstellen.

Die Ausführung der folgenden Aktionen ist erforderlich, um die Namensauflösung zwischen dem virtuellen Netzwerk und Ressourcen in verknüpften Netzwerken zu aktivieren:

1. Erstellen eines benutzerdefinierten DNS-Servers im Azure Virtual Network, in dem Sie die Installation von HDInsight planen

2. Konfigurieren des virtuellen Netzwerks zur Verwendung des benutzerdefinierten DNS-Servers

3. Suchen nach dem von Azure zugewiesenen DNS-Suffix für Ihr virtuelles Netzwerk. Dieser Wert ähnelt diesem Wert: `0owcbllr5hze3hxdja3mqlrhhe.ex.internal.cloudapp.net`. Informationen zum Ermitteln des DNS-Suffix finden Sie im Abschnitt [Beispiel: Benutzerdefiniertes DNS](hdinsight-create-virtual-network.md#example-dns).

4. Konfigurieren der Weiterleitung zwischen den DNS-Servern. Die Konfiguration richtet sich nach dem Typ des Remotenetzwerks.

   * Wenn das Remotenetzwerk ein lokales Netzwerk ist, können Sie das DNS wie folgt konfigurieren:
        
     * __Benutzerdefiniertes DNS__ (im virtuellen Netzwerk):

         * Leiten Sie Anforderungen für das DNS-Suffix des virtuellen Netzwerks an den rekursiven Azure-Resolver (168.63.129.16) weiter. Azure verarbeitet Anforderungen von Ressourcen im virtuellen Netzwerk.

         * Leiten Sie alle anderen Anforderungen an den lokalen DNS-Server weiter. Das lokale DNS verarbeitet alle anderen Anforderungen der Namensauflösung. Dies gilt auch für Internetressourcen, z.B. „Microsoft.com“.

     * __Lokales DNS:__ Leiten Sie Anforderungen für das DNS-Suffix des virtuellen Netzwerks an den benutzerdefinierten DNS-Server weiter. Der benutzerdefinierte DNS-Server leitet die Daten dann an den rekursiven Azure-Resolver weiter.

       Bei dieser Konfiguration werden Anforderungen von vollqualifizierten Domänennamen, die das DNS-Suffix des virtuellen Netzwerks enthalten, an den benutzerdefinierten DNS-Server weiter. Alle anderen Anforderungen (auch für öffentliche Internetadressen) werden vom lokalen DNS-Server verarbeitet.

   * Konfigurieren Sie das DNS wie folgt, wenn das Remotenetzwerk ein anderes Azure Virtual Network ist:

     * __Benutzerdefiniertes DNS__ (in jedem virtuellen Netzwerk):

         * Anforderungen für das DNS-Suffix der virtuellen Netzwerke werden an die benutzerdefinierten DNS-Server weitergeleitet. Das DNS in jedem virtuellen Netzwerk ist für das Auflösen von Ressourcen im jeweiligen Netzwerk verantwortlich.

         * Leiten Sie alle anderen Anforderungen an den rekursiven Azure-Resolver weiter. Der rekursive Resolver ist für das Auflösen von lokalen Ressourcen und Internetressourcen verantwortlich.

       Der DNS-Server für jedes Netzwerk leitet Anforderungen jeweils basierend auf dem DNS-Suffix an das andere Netzwerk weiter. Andere Anforderungen werden mit dem rekursiven Azure-Resolver aufgelöst.

     Ein Beispiel für die einzelnen Konfigurationen finden Sie im Abschnitt [Beispiel: Benutzerdefiniertes DNS](hdinsight-create-virtual-network.md#example-dns).

Weitere Informationen finden Sie im Dokument [Namensauflösung für virtuelle Computer und Rolleninstanzen](../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md).

## <a name="directly-connect-to-apache-hadoop-services"></a>Herstellen einer direkten Verbindung mit Apache Hadoop-Diensten

Sie können unter `https://CLUSTERNAME.azurehdinsight.net` eine Verbindung mit dem Cluster herstellen. Diese Adresse verwendet eine öffentliche IP-Adresse, die möglicherweise nicht erreichbar sind, wenn Sie NSGs verwendet haben, um eingehenden Datenverkehr aus dem Internet einzuschränken. Darüber hinaus können Sie bei der Bereitstellung des Clusters in einem VNET mithilfe des privaten Endpunkts `https://CLUSTERNAME-int.azurehdinsight.net` darauf zugreifen. Dieser Endpunkt wird in eine private IP-Adresse innerhalb des VNET für den Clusterzugriff aufgelöst.

Verwenden Sie die folgenden Schritte, um über das virtuelle Netzwerk eine Verbindung mit Apache Ambari und anderen Webseiten herzustellen:

1. Verwenden Sie eine der folgenden Methoden, um die internen vollqualifizierten Domänennamen (FQDNs) der HDInsight-Clusterknoten zu ermitteln:

    Ersetzen Sie `RESOURCEGROUP` durch den Namen der Ressourcengruppe, die das virtuelle Netzwerk enthält, und geben Sie dann diesen Befehl ein:

    ```powershell
    $clusterNICs = Get-AzNetworkInterface -ResourceGroupName "RESOURCEGROUP" | where-object {$_.Name -like "*node*"}

    $nodes = @()
    foreach($nic in $clusterNICs) {
        $node = new-object System.Object
        $node | add-member -MemberType NoteProperty -name "Type" -value $nic.Name.Split('-')[1]
        $node | add-member -MemberType NoteProperty -name "InternalIP" -value $nic.IpConfigurations.PrivateIpAddress
        $node | add-member -MemberType NoteProperty -name "InternalFQDN" -value $nic.DnsSettings.InternalFqdn
        $nodes += $node
    }
    $nodes | sort-object Type
    ```

    ```azurecli
    az network nic list --resource-group RESOURCEGROUP --output table --query "[?contains(name,'node')].{NICname:name,InternalIP:ipConfigurations[0].privateIpAddress,InternalFQDN:dnsSettings.internalFqdn}"
    ```

    Suchen Sie in der Liste mit den zurückgegebenen Knoten nach dem FQDN für die Hauptknoten, und verwenden Sie die FQDNs, um eine Verbindung mit Ambari und anderen Webdiensten herzustellen. Verwenden Sie beispielsweise `http://<headnode-fqdn>:8080`, um auf Ambari zuzugreifen.

    > [!IMPORTANT]  
    > Einige der auf den Hauptknoten gehosteten Dienste sind jeweils nur auf einem Knoten aktiv. Wenn Sie versuchen, auf einen Dienst auf einem der Hauptknoten zuzugreifen, und der Fehler 404 zurückgegeben wird, wechseln Sie zum anderen Hauptknoten.

2. Informationen dazu, wie Sie den Knoten und Port ermitteln, auf dem ein Dienst verfügbar ist, finden Sie im Dokument [Von HDInsight verwendete Ports und URIs](./hdinsight-hadoop-port-settings-for-services.md).

## <a id="networktraffic"></a>Steuern des Netzwerkdatenverkehrs

### <a name="techniques-for-controlling-inbound-and-outbound-traffic-to-hdinsight-clusters"></a>Verfahren zum Steuern von ein- und ausgehendem Datenverkehr für HDInsight-Cluster

Sie können den Netzwerkdatenverkehr in einem Azure Virtual Network mit den folgenden Methoden steuern:

* Mit **Netzwerksicherheitsgruppen** (NSGs) können Sie ein- und ausgehenden Datenverkehr für das Netzwerk filtern. Weitere Informationen finden Sie im Dokument [Filtern des Netzwerkdatenverkehrs mit Netzwerksicherheitsgruppen](../virtual-network/security-overview.md).

* **Virtuelle Netzwerkgeräte** (Network Virtual Appliances, NVAs) können nur mit ausgehendem Datenverkehr verwendet werden. Mit virtuellen Netzwerkgeräten wird die Funktionalität von Geräten, z. B. Firewalls und Routern, repliziert. Weitere Informationen finden Sie im Dokument [Network Appliances](https://azure.microsoft.com/solutions/network-appliances).

Als verwalteter Dienst benötigt HDInsight uneingeschränkten Zugriff auf die HDInsight-Integritäts- und -Verwaltungsdienste für eingehenden und ausgehenden Datenverkehr aus dem VNET. Beim Verwenden von NSGs und müssen Sie sicherstellen, dass diese Dienste weiterhin mit dem HDInsight-Cluster kommunizieren können.

![Diagramm der HDInsight-Entitäten, erstellt in einem benutzerdefinierten virtuellen Azure-Netzwerk](./media/hdinsight-virtual-network-architecture/vnet-diagram.png)

### <a name="hdinsight-with-network-security-groups"></a>HDInsight mit Netzwerksicherheitsgruppen

Wenn Sie planen, **Netzwerksicherheitsgruppen** zum Steuern des Netzwerkdatenverkehrs zu verwenden, sollten Sie vor dem Installieren von HDInsight die folgenden Aktionen durchführen:

1. Identifizieren Sie die Azure-Region, die Sie für HDInsight verwenden möchten.

2. Identifizieren Sie die IP-Adressen, die für HDInsight erforderlich sind. Weitere Informationen finden Sie unter [HDInsight management IP addresses (IP-Adressen für die HDInsight-Verwaltung)](hdinsight-management-ip-addresses.md).

3. Erstellen oder ändern Sie die Netzwerksicherheitsgruppen für das Subnetz, in dem Sie HDInsight installieren möchten.

    * __Netzwerksicherheitsgruppen__: Lassen Sie __eingehenden__ Datenverkehr über Port __443__ für die IP-Adressen zu. Dadurch wird sichergestellt, dass HDInsight-Verwaltungsdienste den Cluster außerhalb des virtuellen Netzwerks erreichen können.

Weitere Informationen zu Netzwerksicherheitsgruppen finden Sie in der [Übersicht über Netzwerksicherheitsgruppen](../virtual-network/security-overview.md).

### <a name="controlling-outbound-traffic-from-hdinsight-clusters"></a>Steuern des ausgehenden Datenverkehrs der HDInsight-Cluster

Weitere Informationen zum Steuern des ausgehenden Datenverkehrs von HDInsight-Clustern finden Sie unter [Konfigurieren von Einschränkungen des ausgehenden Netzwerkdatenverkehrs für Azure HDInsight-Cluster (Vorschau)](hdinsight-restrict-outbound-traffic.md).

#### <a name="forced-tunneling-to-on-premise"></a>Tunnelerzwingung für lokale Netzwerke

Die Tunnelerzwingung ist eine benutzerdefinierte Routingkonfiguration, bei der für den gesamten Datenverkehr eines Subnetzes die Weiterleitung in ein bestimmtes Netzwerk bzw. an einen bestimmten Standort erzwungen wird, z.B. Ihr lokales Netzwerk. HDInsight bietet __keine__ Unterstützung für die Tunnelerzwingung bei Datenverkehr zu lokalen Netzwerken. 

## <a id="hdinsight-ip"></a>Erforderliche IP-Adressen

Sie sollten sich den Abschnitt [HDInsight management IP addresses (IP-Adressen für die HDInsight-Verwaltung)](hdinsight-management-ip-addresses.md) ansehen, wenn Sie Netzwerksicherheitsgruppen oder benutzerdefinierte Routen zum Steuern von Datenverkehr verwenden.
    
## <a id="hdinsight-ports"></a>Erforderliche Ports

Wenn Sie eine **Firewall** verwenden und extern über bestimmte Ports auf den Cluster zugreifen möchten, müssen Sie möglicherweise Datenverkehr an den für Ihr Szenario erforderlichen Ports zulassen. Standardmäßig ist keine spezifische Whitelist von Ports erforderlich, solange der im vorherigen Abschnitt erläuterte Azure-Verwaltungsdatenverkehr für Cluster an Port 443 zulässig ist.

Eine Liste mit Ports für bestimmte Dienste finden Sie im Dokument [Ports für Apache Hadoop-Dienste in HDInsight](hdinsight-hadoop-port-settings-for-services.md).

Weitere Informationen zu Firewallregeln für virtuelle Geräte finden Sie im Dokument [Szenario für virtuelle Geräte](../virtual-network/virtual-network-scenario-udr-gw-nva.md).

## <a name="next-steps"></a>Nächste Schritte

* Codebeispiele und Beispiele für das Erstellen von virtuellen Azure-Netzwerken finden Sie unter [Create virtual networks for Azure HDInsight clusters (Erstellen von virtuellen Netzwerken für Azure HDInsight-Cluster)](hdinsight-create-virtual-network.md).
* Ein umfassendes Beispiel für die Konfiguration von HDInsight zum Herstellen einer Verbindung mit einem lokalen Netzwerk finden Sie unter [Connect HDInsight to an on-premises network](./connect-on-premises-network.md) (Verbinden von HDInsight mit einem lokalen Netzwerk).
* Informationen zum Konfigurieren von Apache HBase-Clustern in virtuellen Azure-Netzwerken finden Sie unter [Erstellen von Apache HBase-Clustern in HDInsight in Azure Virtual Network](hbase/apache-hbase-provision-vnet.md).
* Informationen zum Konfigurieren der Apache HBase-Georeplikation finden Sie unter [Einrichten der Apache HBase-Clusterreplikation in virtuellen Azure-Netzwerken](hbase/apache-hbase-replication.md).
* Weitere Informationen zu virtuellen Azure-Netzwerken finden Sie in der [Übersicht zu virtuellen Azure-Netzwerken](../virtual-network/virtual-networks-overview.md).
* Weitere Informationen zu Netzwerksicherheitsgruppen finden Sie in der [Übersicht zu Netzwerksicherheitsgruppen](../virtual-network/security-overview.md).
* Weitere Informationen zu benutzerdefinierten Routen finden Sie unter [Benutzerdefinierte Routen und IP-Weiterleitung](../virtual-network/virtual-networks-udr-overview.md).
