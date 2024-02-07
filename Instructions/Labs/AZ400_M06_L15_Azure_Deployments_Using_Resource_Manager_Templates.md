---
lab:
  title: Bereitstellungen mit Azure Bicep-Vorlagen
  module: 'Module 06: Manage infrastructure as code using Azure and DSC'
---

# Bereitstellungen mit Azure Bicep-Vorlagen

# Lab-Handbuch für Kursteilnehmer

## Labanforderungen

- Für dieses Lab ist **Microsoft Edge** oder ein von [Azure DevOps unterstützter Browser](https://docs.microsoft.com/en-us/azure/devops/server/compatibility?view=azure-devops#web-portal-supported-browsers) erforderlich.

- **Einrichten einer Azure DevOps-Organisation**: Wenn Sie nicht bereits eine Azure DevOps-Organisation haben, die Sie für dieses Lab verwenden können, müssen Sie diese erstellen, indem Sie die unter [Erstellen einer Organisation oder Projektsammlung](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/create-organization?view=azure-devops) beschriebenen Anweisungen befolgen.

- Identifizieren Sie ein vorhandenes Azure-Abonnement, oder erstellen Sie ein neues Abonnement.

- Stellen Sie sicher, dass Sie über ein Microsoft-Konto oder ein Azure AD-Konto mit der Rolle „Besitzer“ im Azure-Abonnement und der Rolle „Globaler Administrator“ im Azure AD-Mandanten verfügen, der dem Azure-Abonnement zugeordnet ist.

- [Visual Studio Code](https://code.visualstudio.com/). Diese wird als Teil der Voraussetzungen für diese Übung installiert.

## Übersicht über das Labor

In diesem Lab erstellen Sie eine Azure Bicep-Vorlage und modularisieren sie mithilfe des Azure Bicep Modules-Konzepts. Anschließend ändern Sie die Hauptbereitstellungsvorlage, um das Modul zu verwenden, und stellen schließlich alle Ressourcen in Azure bereit.

## Ziele

In diesem Lab lernen Sie Folgendes:

- Verstehen und Erstellen von Azure Bicep-Vorlagen
- Erstellen eines wiederverwendbaren Bicep-Moduls für Speicherressourcen
- Laden Sie die verknüpfte Vorlage auf Azure Blob Storage hoch und erzeugen Sie ein SAS-Token.
- Ändern der Hauptvorlage, um das Modul zu verwenden
- Modifizieren Sie die Hauptvorlage, um die Abhängigkeiten zu aktualisieren.
- Bereitstellen aller Ressourcen in Azure mithilfe von Azure Bicep-Vorlagen

## Geschätzte Zeit: 60 Minuten

## Anweisungen

### Übung 0: Konfigurieren der Voraussetzungen für das Lab

In dieser Übung richten Sie die Voraussetzungen für das Lab ein, die Visual Studio Code enthalten.

#### Übung 1: Installieren und Konfigurieren von Visual Studio Code

In dieser Aufgabe installieren Sie Visual Studio Code. Wenn Sie diese Voraussetzung bereits implementiert haben, können Sie direkt mit der nächsten Aufgabe fortfahren.

1. Wenn Sie Visual Studio Code noch nicht installiert haben, navigieren Sie im Webbrowserfenster Ihres Lab-Computers zur [Downloadseite von Visual Studio Code](https://code.visualstudio.com/), laden es herunter und führen die Installation aus.

### Übung 1: Erstellen und Bereitstellen von Azure Bicep-Vorlagen

In diesem Lab erstellen Sie eine Azure Bicep-Vorlage und ein Vorlagenmodul. Danach ändern Sie die Hauptbereitstellungsvorlage, um die verknüpfte Vorlage zu verwenden und die Abhängigkeiten zu aktualisieren, und stellen schließlich die Vorlagen in Azure bereit.

#### Aufgabe 1: Erstellen von Azure Bicep-Vorlagen

In dieser Aufgabe verwenden Sie Visual Studio Code, um eine Azure Bicep-Vorlage zu erstellen.

1. Starten Sie Visual Studio Code auf Ihrem Lab-Computer. Klicken Sie in Visual Studio Code auf das Menü **Datei** auf der obersten Ebene, wählen Sie im hierarchischen Menü **Einstellungen** und dann im Untermenü **Extensions** aus. Geben Sie in das Textfeld **Nach Extensions suchen** das Stichwort **Bicep** ein, wählen Sie die von Microsoft veröffentlichte Extension aus, und klicken Sie auf **Installieren**, um die Sprachunterstützung für Azure Bicep zu installieren.
1. Verbinden Sie sich in einem Webbrowser mit: **<https://github.com/Azure/azure-quickstart-templates/blob/master/quickstarts/microsoft.compute/vm-simple-windows/main.bicep>**. Klicken Sie auf die Option **Raw** für die Datei. Kopieren Sie den Inhal aus dem Codefenster und fügen Sie ihn in den Visual Studio Code-Editor ein.

   > **Hinweis**: Anstatt eine Vorlage von Grund auf neu zu erstellen, verwenden wir eine der [Azure-Schnellstartvorlagen](https://azure.microsoft.com/en-us/resources/templates/) mit dem Namen **Einfache Windows-VM bereitstellen**. Die Vorlagen können von GitHub heruntergeladen werden - [vm-simple-windows](https://github.com/Azure/azure-quickstart-templates/blob/master/quickstarts/microsoft.compute/vm-simple-windows).

1. Öffnen Sie auf Ihrem Lab-Computer den Datei-Explorer, und erstellen Sie den folgenden lokalen Ordner, um Vorlagen zu speichern:

   - **C:\\templates**

1. Wechseln Sie zurück zum Visual Studio Code-Fenster mit der main.bicep-Vorlage, klicken Sie auf das Menü **Datei** auf oberster Ebene und dann im Dropdownmenü auf **Speichern unter**. Speichern Sie die Vorlage als **main.bicep** im neu erstellten lokalen Ordner **C:\\templates**.
1. Überprüfen Sie die Vorlage, um ein besseres Verständnis der Struktur zu erhalten. Es gibt fünf Ressourcentypen, die in der Vorlage definiert werden:

   - Microsoft.Storage/storageAccounts
   - Microsoft.Network/publicIPAddresses
   - Microsoft.Network/virtualNetworks
   - Microsoft.Network/networkInterfaces
   - Microsoft.Compute/virtualMachines

1. Speichern Sie die Datei in Visual Studio Code erneut, aber wählen Sie diesmal **C:\\templates** als Ziel und **storage.bicep** als Dateinamen aus.

   > **Hinweis**: Wir haben jetzt zwei identische JSON-Dateien: **C:\\templates\\main.bicep** und **C:\\templates\\storage.bicep**.

#### Aufgabe 2: Erstellen eines Bicep-Moduls für Speicherressourcen

In dieser Aufgabe ändern Sie die Vorlagen, die Sie in der vorherigen Aufgabe gespeichert haben, sodass das Speichervorlagenmodul **storage.bicep** nur ein Speicherkonto erstellt. Es wird von der ersten Vorlage importiert. Das Speichervorlagenmodul muss einen Wert zurück an die Hauptvorlage **main.bicep** übergeben, und dieser Wert wird im Ausgabeelement des Speichervorlagenmoduls definiert.

1. Entfernen Sie in der im Visual Studio Code-Fenster angezeigten Datei **storage.bicep** unter dem **Ressourcen-Abschnitt** alle Ressourcenelemente mit Ausnahme der Ressource **storageAccounts**. Dies sollte dazu führen, dass ein Ressourcenabschnitt wie folgt aussieht:

   ```bicep
   resource storageAccount 'Microsoft.Storage/storageAccounts@2022-05-01' = {
     name: storageAccountName
     location: location
     sku: {
       name: 'Standard_LRS'
     }
     kind: 'Storage'
   }
   ```

1. Entfernen Sie als Nächstes alle Variablendefinitionen:

   ```bicep
   var storageAccountName = 'bootdiags${uniqueString(resourceGroup().id)}'
   var nicName = 'myVMNic'
   var addressPrefix = '10.0.0.0/16'
   var subnetName = 'Subnet'
   var subnetPrefix = '10.0.0.0/24'
   var virtualNetworkName = 'MyVNET'
   var networkSecurityGroupName = 'default-NSG'
   var securityProfileJson = {
     uefiSettings: {
       secureBootEnabled: true
       vTpmEnabled: true
     }
     securityType: securityType
   }
   var extensionName = 'GuestAttestation'
   var extensionPublisher = 'Microsoft.Azure.Security.WindowsAttestation'
   var extensionVersion = '1.0'
   var maaTenantName = 'GuestAttestation'
   var maaEndpoint = substring('emptyString', 0, 0)
   ```

1. Entfernen Sie als Nächstes alle Parameterwerte mit Ausnahme des Speicherorts, und fügen Sie den folgenden Parametercode hinzu, was zu folgendem Ergebnis führt:

   ```bicep
   @description('Location for all resources.')
   param location string = resourceGroup().location

   @description('Name for the storage account.')
   param storageAccountName string
   ```

1. Entfernen Sie als Nächstes am Ende der Datei die aktuelle Ausgabe, und fügen Sie einen neuen storageURI-Ausgabewert hinzu. Ändern Sie die Ausgabe, sodass sie wie unten aussieht:

   ```bicep
   output storageURI string = storageAccount.properties.primaryEndpoints.blob
   ```

1. Speichern Sie das Vorlagenmodul „storage.bicep“. Die Speichervorlage sollte nun wie folgt aussehen:

   ```bicep
   @description('Location for all resources.')
   param location string = resourceGroup().location

   @description('Name for the storage account.')
   param storageAccountName string

   resource storageAccount 'Microsoft.Storage/storageAccounts@2022-05-01' = {
     name: storageAccountName
     location: location
     sku: {
       name: 'Standard_LRS'
     }
     kind: 'Storage'
   }

   output storageURI string = storageAccount.properties.primaryEndpoints.blob
   ```

#### Aufgabe 3: Ändern der Hauptvorlage, sodass sie das Vorlagenmodul verwendet

In dieser Aufgabe ändern Sie die Hauptvorlage so, dass sie auf das Vorlagenmodul verweist, das Sie in der vorherigen Aufgabe erstellt haben.

1. Klicken Sie in Visual Studio Code im Dropdownmenü auf das Menü **Datei** auf oberster Ebene, wählen Sie im Dialogfeld „Datei öffnen“ die Option **Datei öffnen** aus, navigieren Sie zu **C:\\templates\\main.bicep**, wählen Sie sie aus und klicken Sie auf **Öffnen**.
1. Entfernen Sie in der Datei **main.bicep** im Ressourcenabschnitt das Speicherressourcenelement.

   ```bicep
   resource storageAccount 'Microsoft.Storage/storageAccounts@2022-05-01' = {
     name: storageAccountName
     location: location
     sku: {
       name: 'Standard_LRS'
     }
     kind: 'Storage'
   }
   ```

1. Fügen Sie als Nächstes den folgenden Code direkt an derselben Stelle hinzu, an der sich das neu gelöschte Speicherressourcenelement befand:

   ```bicep
   module storageModule './storage.bicep' = {
     name: 'linkedTemplate'
     params: {
       location: location
       storageAccountName: storageAccountName
     }
   }
   ```

1. Außerdem müssen wir den Verweis auf den Blob-URI des Speicherkontos in unserer VM-Ressource ändern, um stattdessen die Ausgabe des Moduls zu verwenden. Suchen Sie die VM-Ressource und ersetzen Sie den Abschnitt „diagnosticsProfile“ durch Folgendes:

   ```bicep
   diagnosticsProfile: {
     bootDiagnostics: {
       enabled: true
       storageUri: storageModule.outputs.storageURI
     }
   }
   ```

1. Überprüfen Sie die folgenden Details in der Hauptvorlage:

   - Ein Modul in der Hauptvorlage wird zur Verknüpfung mit einer anderen Vorlage verwendet.
   - Das Modul hat einen symbolischen Namen namens „storageModule“. Dieser Name wird zum Konfigurieren der Abhängigkeiten verwendet.
   - Sie können den Bereitstellungsmodus Inkrementell nur verwenden, wenn Sie Vorlagenmodule verwenden.
   - Ein relativer Pfad wird für Ihr Vorlagenmodul verwendet.
   - Verwenden Sie Parameter, um Werte von der Hauptvorlage an die Vorlagenmodule zu übergeben.

> **Hinweis**: Mit Azure ARM-Vorlagen hätten Sie ein Speicherkonto verwendet, um die verknüpfte Vorlage hochzuladen, um anderen die Verwendung zu erleichtern. Mit Azure Bicep-Modulen haben Sie die Möglichkeit, sie in die Azure Bicep Module Registry hochzuladen, die sowohl über öffentliche als auch private Registrierungsoptionen verfügt. Weitere Informationen finden Sie in der [Dokumentation zu Azure Bicep](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/modules#file-in-registry).

1. Speichern Sie die Vorlage.

#### Aufgabe 4: Bereitstellen von Ressourcen in Azure mithilfe von Vorlagenmodulen

> **Hinweis**: Sie können Vorlagen auf verschiedene Arten bereitstellen, z. B. über die lokal installierte Azure CLI oder über die Azure Cloud Shell oder über eine CI/CD-Pipeline. In diesem Lab verwenden Sie Azure CLI über dieAzure Cloud Shell.

> **Hinweis**: Im Gegensatz zu ARM-Vorlagen können Sie das Azure-Portal nicht verwenden, um Bicep-Vorlagen direkt bereitzustellen.

> **Hinweis**: Um Azure Cloud Shell zu verwenden, laden Sie sowohl die Dateien main.bicep und als auch storage.bicep in das Basiserzeichnis Ihrer Cloud Shell hoch.

> **Hinweis**: Derzeit wird die Bereitstellung von Bicep-Remotedateien von Azure CLI nicht unterstützt. Sie können die Bicep-Dateien erstellen, um den ARM-Vorlagen-JSON abzurufen und sie dann in ein Speicherkonto hochzuladen und dann remote bereitzustellen.

1. Klicken Sie auf dem Lab-Computer im Webbrowser, in dem das Azure-Portal angezeigt wird, auf das **Cloud Shell**-Symbol, um Cloud Shell zu öffnen.
   > **Hinweis**: Wenn die PowerShell-Sitzung vom Anfang dieser Übung noch aktiv ist, wechseln Sie zu Bash (nächster Schritt).
1. Klicken Sie im Cloud Shell-Bereich auf **PowerShell**, klicken Sie im Dropdownmenü auf **Bash** und wenn Sie dazu aufgefordert werden auf **Bestätigen**.
1. Klicken Sie im Cloud Shell-Bereich auf das Symbol **Dateien hochladen/herunterladen** und im Dropdownmenü auf **Hochladen**.
1. Navigieren Sie im Dialogfeld **Öffnen** zu **C:\\templates\\main.bicep**, und klicken Sie auf **Öffnen**.
1. Führen Sie die gleichen Schritte aus, um auch die Datei **C:\\templates\\storage.bicep** hochzuladen.
1. Führen Sie in einer **Bash**-Sitzung im Cloud Shell-Bereich Folgendes aus, um eine Bereitstellung mithilfe einer neu hochgeladenen Vorlage auszuführen:

   ```bash
   az deployment group what-if --name az400m06l15deployment --resource-group az400m06l15-RG --template-file main.bicep
   ```

1. Wenn Sie aufgefordert werden, den Wert für "adminUsername" anzugeben, geben Sie **Student** ein, und drücken Sie Taste **Enter** .
1. Wenn Sie aufgefordert werden, den Wert für "adminPassword" anzugeben, geben Sie **Pa55w.rd1234** ein, und drücken Sie die Taste **Enter** . (Kennworteingabe wird nicht angezeigt)
1. Überprüfen Sie das Ergebnis dieses Befehls, mit dem Ihre Bereitstellung überprüft wird, und teilen Sie uns mit, ob Fehler in Ihren Vorlagen vorliegen. Dies ist besonders bei der Bereitstellung von Vorlagen mit vielen Ressourcen und in geschäftskritischen Cloudumgebungen sehr wertvoll.

1. Führen Sie in einer **Bash**-Sitzung im Cloud Shell-Bereich Folgendes aus, um eine Bereitstellung mithilfe einer neu hochgeladenen Vorlage auszuführen:

   ```bash
   LOCATION='<region>'
   ```
   > **Hinweis**: Ersetzen Sie den Namen der Region durch eine Region in der Nähe Ihres Standorts. Wenn Sie nicht wissen, welche Speicherorte verfügbar sind, führen Sie den Befehl `az account list-locations -o table` aus.
  
   ```bash
   az group create --name az400m06l15-RG --location $LOCATION
   ```

   ```bash   
   az deployment group create --name az400m06l15deployment --resource-group az400m06l15-RG --template-file main.bicep
   ```

1. Wenn Sie aufgefordert werden, den Wert für "adminUsername" anzugeben, geben Sie **Student** ein, und drücken Sie Taste **Enter** .
1. Wenn Sie aufgefordert werden, den Wert für "adminPassword" anzugeben, geben Sie **Pa55w.rd1234** ein, und drücken Sie die Taste **Enter** . (Kennworteingabe wird nicht angezeigt)

1. Wenn Beim Ausführen des obigen Befehls Fehler beim Bereitstellen der Vorlage angezeigt werden, versuchen Sie Folgendes:

   - Wenn Sie über mehrere Azure-Abonnements verfügen, stellen Sie sicher, dass Sie den richtigen Abonnementkontext festgelegt haben, in der die Ressourcengruppe bereitgestellt wird.
   - Stellen Sie sicher, dass auf die verknüpfte Vorlage über den angegebenen URI zugegriffen werden kann.

> **Hinweis**: Im nächsten Schritt könnten Sie nun die übrigen Ressourcendefinitionen in der Hauptbereitstellungsvorlage modularisieren, z. B. die Ressourcendefinitionen für das Netzwerk und die VM.

> **Hinweis**: Wenn Sie die bereitgestellten Ressourcen nicht verwenden möchten, sollten Sie sie löschen, um damit verbundene Kosten zu vermeiden. Löschen Sie hierzu einfach die Ressourcengruppe **az400m06l15-RG**.

### Übung 2: Entfernen Sie die Azure-Laborressourcen

In dieser Übung entfernen Sie die in dieser Übung bereitgestellten Azure-Ressourcen, um unerwartete Gebühren zu beseitigen.

> **Hinweis**: Denken Sie daran, alle neu erstellten Azure-Ressourcen zu entfernen, die Sie nicht mehr verwenden. Durch das Entfernen nicht verwendeter Ressourcen wird sichergestellt, dass keine unerwarteten Kosten anfallen.

#### Übung 1: Entfernen der Azure Lab-Ressourcen

In dieser Aufgabe verwenden Sie Azure Cloud Shell, um die in dieser Übung bereitgestellten Azure-Ressourcen zu entfernen, um unnötige Gebühren zu vermeiden.

1. Öffnen Sie im Azure-Portal im **Cloud Shell**-Bereich die **Bash**-Sitzung.
1. Listen Sie alle Ressourcengruppen auf, die während der Labs in diesem Modul erstellt wurden, indem Sie den folgenden Befehl ausführen:

   ```bash
   az group list --query "[?starts_with(name,'az400m06l15-RG')].name" --output tsv
   ```

1. Löschen Sie alle Ressourcengruppen, die Sie während der praktischen Übungen in diesem Modul erstellt haben, indem Sie den folgenden Befehl ausführen:

   ```bash
   az group list --query "[?starts_with(name,'az400m06l15-RG')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
   ```

   > **Hinweis**: Der Befehl wird (dem --nowait-Parameter entsprechend) asynchron ausgeführt. Dies bedeutet, dass Sie zwar einen weiteren Azure CLI-Befehl in derselben Bash-Sitzung direkt im Anschluss ausführen können, es jedoch einige Minuten dauert, bis die Ressourcengruppen tatsächlich entfernt werden.

## Überprüfung

In diesem Lab haben Sie gelernt, wie Sie eine Azure Resource Manager-Vorlage erstellen, mithilfe einer verknüpften Vorlage modularisieren, die Hauptbereitstellungsvorlage ändern, um die verknüpfte Vorlage und aktualisierte Abhängigkeiten aufzurufen, und schließlich die Vorlagen in Azure bereitstellen.
