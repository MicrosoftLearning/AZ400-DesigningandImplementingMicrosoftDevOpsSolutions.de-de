---
lab:
  title: Bereitstellungen mit Azure Bicep-Vorlagen
  module: 'Module 05: Manage infrastructure as code using Azure and DSC'
---

# Bereitstellungen mit Azure Bicep-Vorlagen

## Lab-Handbuch für Kursteilnehmer

## Labanforderungen

- Für dieses Lab ist **Microsoft Edge** oder ein von [Azure DevOps unterstützter Browser](https://docs.microsoft.com/azure/devops/server/compatibility) erforderlich.

- **Einrichten einer Azure DevOps-Organisation**: Wenn Sie nicht bereits eine Azure DevOps-Organisation haben, die Sie für dieses Lab verwenden können, müssen Sie diese erstellen, indem Sie die unter [Erstellen einer Organisation oder Projektsammlung](https://docs.microsoft.com/azure/devops/organizations/accounts/create-organization) beschriebenen Anweisungen befolgen.

- Identifizieren Sie ein vorhandenes Azure-Abonnement, oder erstellen Sie ein neues Abonnement.

- Stellen Sie sicher, dass Sie über ein Microsoft- oder Microsoft Entra-Konto mit der Rolle „Besitzer“ im Azure-Abonnement und der Rolle „Globaler Administrator“ im Microsoft Entra-Mandanten verfügen, der dem Azure-Abonnement zugeordnet ist. Ausführliche Informationen finden Sie in den Artikeln zum [Auflisten von Azure-Rollenzuweisungen mithilfe des Azure-Portals](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-list-portal) und [Anzeigen und Zuweisen von Administratorrollen in Azure Active Directory](https://docs.microsoft.com/azure/active-directory/roles/manage-roles-portal).

## Übersicht über das Labor

In diesem Lab erstellen Sie eine Azure Bicep-Vorlage und modularisieren sie mithilfe des Azure Bicep Modules-Konzepts. Anschließend ändern Sie die Hauptbereitstellungsvorlage, um das Modul zu verwenden, und stellen schließlich alle Ressourcen in Azure bereit.

## Ziele

In diesem Lab lernen Sie Folgendes:

- Grundlegendes zur Struktur einer Azure Bicep-Vorlage.
- Erstellen eines wiederverwendbaren Bicep-Moduls.
- Ändern der Hauptvorlage, um das Modul zu verwenden.
- Alle Ressourcen mithilfe von Azure YAML-Pipelines in Azure bereitstellen.

## Geschätzte Dauer: 45 Minuten

## Anweisungen

### Übung 0: Konfigurieren der Voraussetzungen für das Lab

In dieser Übung richten Sie die Voraussetzungen für das Lab ein, das aus einem neuen Azure DevOps-Projekt mit einem Repository basierend auf dem [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb) besteht.

#### Aufgabe 1: (überspringen, wenn fertig) Erstellen und Konfigurieren des Teamprojekts

In dieser Aufgabe erstellen Sie ein **eShopOnWeb** Azure DevOps-Projekt, das von mehreren Labs verwendet werden soll.

1. Öffnen Sie auf Ihrem Lab-Computer in einem Browserfenster Ihre Azure DevOps-Organisation. Klicken Sie auf **Neues Projekt**. Weisen Sie Ihrem Projekt den Namen **eShopOnWeb** zu, und lassen Sie die anderen Felder auf den Standardwerten. Klicken Sie auf **Erstellen**.

    ![Erstellen eines Projekts](images/create-project.png)

#### Aufgabe 2: (überspringen, wenn erledigt) Importieren von eShopOnWeb Git Repository

Bei dieser Aufgabe importieren Sie das eShopOnWeb Git-Repository, das von mehreren Labs verwendet wird.

1. Öffnen Sie auf Ihrem Lab-Computer in einem Browserfenster Ihre Azure DevOps-Organisation und das zuvor erstellte **eShopOnWeb**-Projekt. Klicken Sie auf **Repos>Dateien**, **Repository importieren**. Klicken Sie auf **Importieren**. Fügen Sie im Fenster **Git Repository importieren** die folgende URL <https://github.com/MicrosoftLearning/eShopOnWeb.git> ein, und klicken Sie auf **Importieren**:

    ![Importieren eines Repositorys](images/import-repo.png)

1. Das Repository ist wie folgt organisiert:
    - Der Ordner **.ado** enthält Azure DevOps-YAML-Pipelines.
    - Der Ordner **.devcontainer** enthält ein Containersetup für die Entwicklung mithilfe von Containern (entweder lokal in VS Code oder über GitHub Codespaces).
    - Der Ordner **infra** enthält eine Bicep&ARM-Infrastruktur als Codevorlagen, die in einigen Labszenarien verwendet werden.
    - Der Ordner **.github** enthält YAML GitHub-Workflow-Definitionen.
    - Der Ordner **src** enthält die .NET 8-Website, die in den Labszenarios verwendet wird.

### Übung 1: Verstehen einer Azure Bicep-Vorlage und Vereinfachen der Vorlage mithilfe eines wiederverwendbaren Moduls

In dieser Übung überprüfen Sie eine Azure Bicep-Vorlage und vereinfachen sie mithilfe eines wiederverwendbaren Moduls.

#### Aufgabe 1: Erstellen von Azure Bicep-Vorlagen

In dieser Aufgabe verwenden Sie Visual Studio Code, um eine Azure Bicep-Vorlage zu erstellen.

1. Navigieren Sie auf der Registerkarte Browser zu Ihrem Azure DevOps-Projekt, und navigieren Sie zu **Repos** und **Dateien**. Öffnen Sie den Ordner `infra` und suchen Sie die Datei `simple-windows-vm.bicep`.

   ![Simple-windows-vm.bicep-Datei](./images/m06/browsebicepfile.png)

1. Überprüfen Sie die Vorlage, um ein besseres Verständnis der Struktur zu erhalten. Es gibt Parameter mit Typen, Standardwerten und Validierung, einige Variablen und eine ganze Reihe von Ressourcen mit diesen Typen:

   - Microsoft.Storage/storageAccounts
   - Microsoft.Network/publicIPAddresses
   - Microsoft.Network/virtualNetworks
   - Microsoft.Network/networkInterfaces
   - Microsoft.Compute/virtualMachines

1. Achten Sie auf die Einfachheit der Ressourcendefinitionen und auf die Möglichkeit, in der gesamten Vorlage implizit auf symbolische Namen anstatt auf explizite `dependsOn` zu verweisen.

#### Aufgabe 2: Erstellen eines Bicep-Moduls für Speicherressourcen

In dieser Aufgabe erstellen Sie ein Speichervorlagenmodul **storage.bicep**, das nur ein Speicherkonto erstellt und von der Hauptvorlage importiert wird. Das Speichervorlagenmodul muss einen Wert zurück an die Hauptvorlage **main.bicep** übergeben, und dieser Wert wird im Ausgabeelement des Speichervorlagenmoduls definiert.

1. Zunächst muss die Speicherressource aus der Hauptvorlage entfernt werden. Klicken Sie in der oberen rechten Ecke Ihres Browserfensters auf die Schaltfläche **Bearbeiten**:

   ![Schaltfläche „Bearbeiten“](./images/m06/edit.png)

1. Löschen Sie nun die Speicherressource:

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

1. Committen Sie die Datei, auch wenn Sie damit noch nicht ganz fertig sind.

   ![Committen der Datei](./images/m06/commit.png)

1. Zeigen Sie als Nächstes mit der Maus auf den `Infra` Ordner, und klicken Sie auf das Auslassungssymbol, und wählen Sie **dann Neu** und **Datei** aus. Geben Sie **storage.bicep** als Namen ein und klicken Sie auf **Erstellen**.

   ![Menü „Neue Datei“](./images/m06/newfile.png)

1. Kopieren Sie nun den folgenden Codeausschnitt in die Datei und committen Sie Ihre Änderungen:

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

1. Navigieren Sie zurück zur Datei `simple-windows-vm.bicep` und klicken Sie noch einmal auf die Schaltfläche **Bearbeiten**.

1. Fügen Sie anschließend den folgenden Code nach den Variablen ein:

   ```bicep
   module storageModule './storage.bicep' = {
     name: 'linkedTemplate'
     params: {
       location: location
       storageAccountName: storageAccountName
     }
   }
   ```

1. Außerdem muss der Verweis auf den Blob-URI des Speicherkontos in der Ressource der virtuellen Maschine so geändert werden, dass stattdessen die Ausgabe des Moduls verwendet wird. Suchen Sie die VM-Ressource und ersetzen Sie den Abschnitt „diagnosticsProfile“ durch Folgendes:

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
   - Das Modul hat den symbolischen Namen `storageModule`. Dieser Name wird zum Konfigurieren der Abhängigkeiten verwendet.
   - Bei der Verwendung von Vorlagenmodulen können Sie nur den **Inkrementellen** Bereitstellungsmodus verwenden.
   - Ein relativer Pfad wird für Ihr Vorlagenmodul verwendet.
   - Verwenden Sie Parameter, um Werte von der Hauptvorlage an die Vorlagenmodule zu übergeben.

1. Commiten der Vorlage.

### Übung 2: Bereitstellen der Vorlagen in Azure mithilfe von YAML-Pipelines

In dieser Übung erstellen Sie eine Dienstverbindung und verwenden diese in einer Azure DevOps YAML-Pipeline, um Ihre Vorlage in Ihrer Azure-Umgebung bereitzustellen.

#### Aufgabe 1: (überspringen, wenn erledigt) Erstellen einer Dienstverbindung für die Bereitstellung

In dieser Aufgabe erstellen Sie mithilfe der Azure CLI ein Dienstprinzipal, der es Azure DevOps ermöglicht:

- Ressourcen in Ihrem Azure-Abonnement bereitstellen.
- Sie haben Lesezugriff auf die später erstellten Key Vault-Geheimnisse.

> **Hinweis**: Wenn Sie bereits über ein Dienstprinzipal verfügen, können Sie direkt mit der nächsten Aufgabe fortfahren.

Sie benötigen ein Dienstprinzipal, um Azure-Ressourcen über Azure Pipelines bereitzustellen. Da Sie Geheimnisse in einer Pipeline abrufen werden, müssen Sie dem Dienst bei der Erstellung des Azure Key Vault eine Berechtigung erteilen.

Ein Dienstprinzipal wird automatisch von Azure Pipelines erstellt, wenn Sie eine Verbindung zu einem Azure-Abonnement innerhalb einer Pipeline-Definition herstellen oder wenn Sie eine neue Dienstverbindung über die Projekteinstellungsseite erstellen (automatische Option). Sie können den Dienstprinzipal auch manuell über das Portal oder mithilfe von Azure CLI erstellen und ihn projektübergreifend wiederverwenden.

1. Starten Sie auf Ihrem Labcomputer einen Webbrowser, navigieren Sie zum [**Azure-Portal**](https://portal.azure.com), und melden Sie sich an. Verwenden Sie hierzu die Anmeldeinformationen eines Benutzerkontos, das in dem Abonnement, das Sie in diesem Lab verwenden, und das in dem in dem Microsoft Entra-Mandanten, der dem Abonnement zugeordnet ist, über die Rolle „Globaler Administrator“ verfügt.
1. Klicken Sie im Azure-Portal auf das Symbol **Cloud Shell**, das sich direkt rechts neben dem Textfeld für die Suche im oberen Bereich der Seite befindet.
1. Wählen Sie bei Aufforderung zur Auswahl von **Bash** oder **PowerShell** die Option **Bash** aus.

   >**Hinweis**: Wenn Sie **Cloud Shell** zum ersten Mal starten und die Meldung **Für Sie wurde kein Speicher bereitgestellt** angezeigt wird, wählen Sie das in diesem Lab verwendete Abonnement aus, und klicken Sie dann auf **Speicher erstellen**.

1. Führen Sie an der Eingabeaufforderung **Bash** im Bereich **Cloud Shell** die folgenden Befehle aus, um die Werte der Attribute „Azure-Abonnement-ID“ und „Abonnementname“ abzurufen:

    ```bash
    az account show --query id --output tsv
    az account show --query name --output tsv
    ```

    > **Hinweis**: Kopieren Sie beide Werte in eine Textdatei. Sie werden sie später in diesem Lab benötigen.

1. Führen Sie an der Eingabeaufforderung **Bash**im Bereich **Cloud Shell** den folgenden Befehl aus, um ein Dienstprinzipal zu erstellen (ersetzen Sie **myServicePrincipalName** durch eine beliebige eindeutige Zeichenfolge aus Buchstaben und Ziffern) und **mySubscriptionID** durch Ihre Azure subscriptionId :

    ```bash
    az ad sp create-for-rbac --name myServicePrincipalName \
                         --role contributor \
                         --scopes /subscriptions/mySubscriptionID
    ```

    > **Hinweis**: Der Befehl generiert eine JSON-Ausgabe. Kopieren Sie die Ausgabe in eine Textdatei. Sie benötigen diese später in diesem Lab.

1. Starten Sie als Nächstes auf dem Laborcomputer einen Webbrowser, navigieren Sie zum Azure DevOps **eShopOnWeb**-Projekt. Klicken Sie auf **Project Einstellungen>Dienstverbindungen (unter Pipelines)** und **Neue Dienstverbindung**.

    ![Neue Dienstverbindung](images/new-service-connection.png)

1. Wählen Sie im Bildschirm **Neue Dienstverbindung** die Option **Azure Resource Manager** und anschließend **Weiter** aus (Sie müssen möglicherweise scrollen).

1. Wählen Sie **Dienstprinzipal (manuell)** und klicken Sie auf **Weiter**.

1. Füllen Sie die leeren Felder mit den Informationen aus, die während der vorherigen Schritte gesammelt wurden:
    - Abonnement-ID und -Name.
    - Dienstprinzipal-ID (appId), Dienstprinzipalschlüssel (Kennwort) und Mandanten-ID (Mandant).
    - Geben Sie in **Name der Dienstverbindung** **azure subs** ein. Auf diesen Namen wird in YAML-Pipelines verwiesen, wenn eine Azure DevOps-Dienstverbindung erforderlich ist, um mit Ihrem Azure-Abonnement zu kommunizieren.

    ![Azure-Serviceverbindung](images/azure-service-connection.png)

1. Klicken Sie auf **Überprüfen und speichern**.

#### Aufgabe 2: Bereitstellen von Ressourcen in Azure durch YAML-Pipelines

1. Navigieren Sie zurück zum Bereich **Pipelines** im **Pipelinehub**.
1. Klicken Sie im Fenster **Erste Pipeline erstellen** auf **Pipeline erstellen**.

    > **Hinweis**: Wir verwenden den Assistenten, um eine neue YAML-Pipelinedefinition basierend auf unserem Projekt zu erstellen.

1. Klicken Sie im Bereich **Wo befindet sich Ihr Code?** auf die Option **Azure Repos Git (YAML)**.
1. Klicken Sie im Bereich **Ein Repository auswählen** auf **eShopOnWeb**.
1. Scrollen Sie auf dem Bildschirm **Pipeline konfigurieren** nach unten und wählen Sie die Option **Vorhandene Azure Pipelines-YAML-Datei** aus.
1. Geben Sie im Blatt **Auswählen einer vorhandenen YAML-Datei** die folgenden Parameter an:
   - Branch: **main**
   - Pfad: **.ado/eshoponweb-cd-windows-cm.yml**
1. Klicken Sie auf **Weiter**, um die Einstellungen zu speichern.
1. Wählen Sie im Abschnitt Variablen einen Namen für Ihre Ressourcengruppe aus, legen Sie den gewünschten Speicherort fest, und ersetzen Sie den Wert der Dienstverbindung durch eine Ihrer vorhandenen Dienstverbindungen, die Sie zuvor erstellt haben.
1. Klicken Sie in der oberen rechten Ecke auf die Schaltfläche **Speichern und ausführen**. Wenn der Commit-Dialog angezeigt wird, klicken Sie erneut auf **Speichern**.

   ![Speichern und Ausführen der YAML-Pipeline nach dem Vornehmen von Änderungen](./images/m06/saveandrun.png)

1. Warten Sie, bis die Bereitstellung abgeschlossen ist, und überprüfen Sie die Ergebnisse.
   ![Erfolgreiche Ressourcenbereitstellung in Azure mithilfe von YAML-Pipelines](./images/m06/deploy.png)

#### Aufgabe 3: Entfernen der Azure-Lab-Ressourcen.

In dieser Aufgabe verwenden Sie Azure Cloud Shell, um die in diesem Lab bereitgestellten Azure-Ressourcen zu entfernen, um unnötige Gebühren zu vermeiden.

1. Öffnen Sie im Azure-Portal die **Bash**-Shell-Sitzung im Bereich **Cloud Shell**.
1. Löschen Sie alle Ressourcengruppen, die Sie in den Übungen dieses Moduls erstellt haben, indem Sie den folgenden Befehl ausführen (ersetzen Sie den Namen der Ressourcengruppe durch den von Ihnen gewählten Namen):

   ```bash
   az group list --query "[?starts_with(name,'AZ400-EWebShop-NAME')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
   ```

   > **Hinweis**: Der Befehl wird (dem --nowait-Parameter entsprechend) asynchron ausgeführt. Dies bedeutet, dass Sie zwar einen weiteren Azure CLI-Befehl in derselben Bash-Sitzung direkt im Anschluss ausführen können, es jedoch einige Minuten dauert, bis die Ressourcengruppen tatsächlich entfernt werden.

## Überprüfung

In dieser Übung haben Sie erfahren, wie Sie eine Azure Bicep-Vorlage erstellen, mithilfe eines Vorlagenmoduls modularisieren, die Hauptbereitstellungsvorlage so ändern, dass sie das Modul und aktualisierte Abhängigkeiten verwendet, und schließlich die Vorlagen mithilfe von YAML-Pipelines in Azure bereitstellen.
