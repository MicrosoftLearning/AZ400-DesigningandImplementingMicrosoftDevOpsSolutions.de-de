---
lab:
  title: Bereitstellungen mit Azure Bicep-Vorlagen
  module: 'Module 06: Manage infrastructure as code using Azure and DSC'
---

# Bereitstellungen mit Azure Bicep-Vorlagen

## Lab-Handbuch für Kursteilnehmer

## Labanforderungen

- Für dieses Lab ist **Microsoft Edge** oder ein [von Azure DevOps unterstützter Browser](https://docs.microsoft.com/azure/devops/server/compatibility) erforderlich.

- **Einrichten einer Azure DevOps-Organisation**: Wenn Sie nicht bereits eine Azure DevOps-Organisation haben, die Sie für dieses Lab verwenden können, müssen Sie diese erstellen, indem Sie die unter [Erstellen einer Organisation oder Projektsammlung](https://docs.microsoft.com/azure/devops/organizations/accounts/create-organization) beschriebenen Anweisungen befolgen.

- Identifizieren Sie ein vorhandenes Azure-Abonnement, oder erstellen Sie ein neues Abonnement.

- Stellen Sie sicher, dass Sie über ein Microsoft- oder Microsoft Entra-Konto mit der Rolle „Besitzer“ im Azure-Abonnement und der Rolle „Globaler Administrator“ im Microsoft Entra-Mandanten verfügen, der dem Azure-Abonnement zugeordnet ist. Ausführliche Informationen finden Sie in den Artikeln zum [Auflisten von Azure-Rollenzuweisungen mithilfe des Azure-Portals](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-list-portal) und [Anzeigen und Zuweisen von Administratorrollen in Azure Active Directory](https://docs.microsoft.com/azure/active-directory/roles/manage-roles-portal).

## Übersicht über das Labor

In diesem Lab erstellen Sie eine Azure Bicep-Vorlage und modularisieren sie mithilfe des Azure Bicep Modules-Konzepts. Anschließend ändern Sie die Hauptbereitstellungsvorlage, um das Modul zu verwenden, und stellen schließlich alle Ressourcen in Azure bereit.

## Ziele

In diesem Lab lernen Sie Folgendes:

- Grundlegendes zur Struktur einer Azure Bicep-Vorlage.
- Erstellen Sie ein wiederverwendbares Bicep-Modul.
- Ändern der Hauptvorlage, um das Modul zu verwenden
- Stellen Sie alle Ressourcen mithilfe von Azure YAML-Pipelines in Azure bereit.

## Geschätzte Dauer: 45 Minuten

## Anweisungen

### Übung 0: Konfigurieren der Voraussetzungen für das Lab.

In dieser Übung richten Sie die Voraussetzungen für das Labor ein, das aus einem neuen Azure DevOps-Projekt mit einem Repository basierend auf dem [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb) besteht.

#### Aufgabe 1: (überspringen, wenn fertig) Erstellen und Konfigurieren des Teamprojekts

In dieser Aufgabe erstellen Sie ein **eShopOnWeb** Azure DevOps-Projekt, das von mehreren Laboren verwendet werden soll.

1. Öffnen Sie auf Ihrem Laborcomputer in einem Browserfenster Ihre Azure DevOps-Organisation. Klicken auf „Neues Projekt“ Weisen Sie Ihrem Projekt den Namen **"eShopOnWeb** " zu, und lassen Sie die anderen Felder standardmäßig. Klicken Sie auf **Erstellen**.

    ![Erstellen eines Projekts](images/create-project.png)

#### Aufgabe 2: (überspringen, wenn erledigt) Importieren von eShopOnWeb Git Repository

Bei dieser Aufgabe importieren Sie das eShopOnWeb Git-Repository, das von mehreren Labs verwendet wird.

1. Öffnen Sie auf Ihrem Laborcomputer in einem Browserfenster Ihre Azure DevOps-Organisation und das zuvor erstellte **eShopOnWeb-Projekt** . Klicken Sie auf **Repos>Files** , **Import a Repository**. Wählen Sie **Importieren** aus. Fügen Sie im **Fenster "Git Repository** importieren" die folgende URL https://github.com/MicrosoftLearning/eShopOnWeb.git  ein, und klicken Sie auf " **Importieren**":

    ![Importieren eines Repositorys](images/import-repo.png)

1. Das Repository ist wie folgt organisiert:
    - Der Ordner „.ado“ enthält Azure DevOps-YAML-Pipelines.
    - Der Ordner „.devcontainer“ enthält ein Containersetup für die Entwicklung mithilfe von Containern (entweder lokal in VS Code oder über GitHub Codespaces).
    - **Azure-Ordner** enthält Bicep&ARM-Infrastruktur als Codevorlagen, die in einigen Lab-Szenarien verwendet werden.
    - **GITHUB-Ordnercontainer-YAML-GitHub-Workflowdefinitionen** .
    - Der Ordner „src“ enthält die .NET 6-Website, die in den Labszenarien verwendet wird.

### Übung 1: Verstehen einer Azure Bicep-Vorlage und Vereinfachen der Vorlage mithilfe eines wiederverwendbaren Moduls

In dieser Übung überprüfen Sie eine Azure Bicep-Vorlage und vereinfachen sie mithilfe eines wiederverwendbaren Moduls.

#### Erstellen von Azure Bicep-Vorlagen

In dieser Aufgabe verwenden Sie Visual Studio Code, um eine Azure Bicep-Vorlage zu erstellen.

1. Navigieren Sie auf der Registerkarte "Browser" zu Ihrem Azure DevOps-Projekt, und navigieren Sie zu "Repos **" und **"** Dateien"**. Suchen und Öffnen der `.azure\bicep` Datei im `simple-windows-vm.bicep` Ordner.

   ![Datei "Simple-windows-vm.bicep"](./images/m06/browsebicepfile.png)

1. Überprüfen Sie die Vorlage, um ein besseres Verständnis der Struktur zu erhalten. Es gibt einige Parameter mit Typen, Standardwerten und Überprüfungen, einigen Variablen und einigen Ressourcen mit diesen Typen:

   - Microsoft.Storage/storageAccounts
   - Microsoft.Network/publicIPAddresses
   - Microsoft.Network/virtualNetworks
   - Microsoft.Network/networkInterfaces
   - Microsoft.Compute/virtualMachines

1. Achten Sie darauf, wie einfach die Ressourcendefinitionen sind und wie einfach symbolische Namen und nicht `dependsOn` explizit in der gesamten Vorlage referenzieren können.

#### Erstellen eines wiederverwendbaren Bicep-Moduls für Speicherressourcen

In dieser Aufgabe erstellen Sie ein Speichervorlagenmodul **storage.bicep**, das nur ein Speicherkonto erstellt und von der Standard Vorlage importiert wird. Das Speichervorlagenmodul muss einen Wert zurück an die Standard Vorlage übergeben, **Standard.bicep**, und dieser Wert wird im Ausgabeelement des Speichervorlagenmoduls definiert.

1. Zuerst müssen wir die Speicherressource aus unserer Standard-Vorlage entfernen. Klicken Sie in der oberen rechten Ecke des Browserfensters auf die **Schaltfläche "Bearbeiten** ":

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

1. Übernehmen Sie die Datei jedoch noch nicht.

   ![Committen der Datei](./images/m06/commit.png)

1. Zeigen Sie als Nächstes mit der Maus auf den Bicep-Ordner, und klicken Sie auf das Auslassungszeichensymbol, und wählen Sie **dann "Neu" und **"Datei**"** aus. Geben Sie als Namen **EFGetStarted** ein, und klicken Sie dann auf **Erstellen**.

   ![Menü "Neue Datei"](./images/m06/newfile.png)

1. Kopieren Sie nun den folgenden Codeausschnitt in die Datei, und übernehmen Sie Die Änderungen:

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

#### Ändern der Hauptvorlage, um das Modul zu verwenden

In dieser Aufgabe ändern Sie die Standard Vorlage so, dass sie auf das Vorlagenmodul verweist, das Sie in der vorherigen Aufgabe erstellt haben.

1. Navigieren Sie zurück zur `simple-windows-vm.bicep` Datei, und klicken Sie erneut auf die **Schaltfläche "Bearbeiten** ".

1. Fügen Sie nach dem Erstellen der -Variablen den folgenden Code hinzu:

   ```bicep
   module storageModule './storage.bicep' = {
     name: 'linkedTemplate'
     params: {
       location: location
       storageAccountName: storageAccountName
     }
   }
   ```

1. Außerdem müssen wir den Verweis auf den Blob-URI des Speicherkontos in unserer Vm-Ressource ändern, um stattdessen die Ausgabe des Moduls zu verwenden. Suchen Sie die Ressource des virtuellen Computers, und ersetzen Sie den Abschnitt Diagnose Profile durch Folgendes:

   ```bicep
   diagnosticsProfile: {
     bootDiagnostics: {
       enabled: true
       storageUri: storageModule.outputs.storageURI
     }
   }
   ```

1. Die folgenden Angaben werden in der Vorlage angefordert.

   - Eine -Ressource in der Hauptvorlage wird zum Verknüpfen mit einer anderen Vorlage verwendet.
   - Das Modul hat einen symbolischen Namen namens `storageModule`. Dieser Name wird zum Konfigurieren der Abhängigkeit verwendet.
   - Beim Aufrufen verknüpfter Vorlagen können Sie nur den Bereitstellungsmodus **Inkrementell** verwenden.
   - Ein relativer Pfad wird für Ihr Vorlagenmodul verwendet.
   - Übergeben Sie mit  Werte aus der Hauptvorlage an die verknüpfte Vorlage.

1. Commit für die Vorlage.

### Übung 2: Bereitstellen der Vorlagen in Azure mithilfe von YAML-Pipelines

In dieser Übung erstellen Sie eine Dienstverbindung und verwenden sie in einer Azure DevOps YAML-Pipeline, um Ihre Vorlage in Ihrer Azure-Umgebung bereitzustellen.

#### Aufgabe 1: (überspringen, falls erledigt) Erstellen eines Dienst-Verbinden ion für die Bereitstellung

In dieser Aufgabe erstellen Sie einen Dienstprinzipal mithilfe der Azure CLI, der Azure DevOps zulässt:

- Bereitstellen von Ressourcen in einem Azure-Abonnement
- Haben Sie Lesezugriff auf die später erstellten Schlüsseltresorschlüssel.

> **Hinweis**: Wenn Sie bereits über einen Dienstprinzipal verfügen, können Sie direkt mit der nächsten Aufgabe fortfahren.

Sie benötigen einen Dienstprinzipal, um Azure-Ressourcen aus Azure-Pipelines bereitzustellen. Da wir geheime Schlüssel in einer Pipeline abrufen werden, müssen wir dem Dienst die Berechtigung erteilen, wenn wir den Azure Key Vault erstellen.

Ein Dienstprinzipal wird automatisch von Azure Pipelines erstellt, wenn Sie eine Verbindung mit einem Azure-Abonnement innerhalb einer Pipelinedefinition herstellen oder wenn Sie eine neue Dienst-Verbinden ion über die Projekteinstellungsseite (automatische Option) erstellen. Sie können den Dienstprinzipal auch manuell über das Portal erstellen oder Azure CLI verwenden und es für alle Projekte wiederverwenden.

1. Starten Sie auf Ihrem Labcomputer einen Webbrowser, navigieren Sie zum [**Azure-Portal**](https://portal.azure.com), und melden Sie sich an. Verwenden Sie hierzu die Anmeldeinformationen eines Benutzerkontos, das in dem Abonnement, das Sie in diesem Lab verwenden, über die Rolle „Besitzer“ und in dem Azure AD-Mandanten, der dem Azure-Abonnement zugeordnet ist, über die Rolle „Globaler Administrator“ verfügt.
1. Öffnen Sie im Azure-Portal den Bereich **Cloud Shell**, indem Sie auf das Symbolleistensymbol rechts neben dem Textfeld für die Suche klicken.
1. Wählen Sie bei Aufforderung zur Auswahl von **Bash** oder **PowerShell** die Option **Bash** aus.

   >**Hinweis**: Wenn Sie **Cloud Shell** zum ersten Mal starten und die Meldung **Für Sie wurde kein Speicher bereitgestellt** angezeigt wird, wählen Sie das in diesem Lab verwendete Abonnement aus, und klicken Sie dann auf **Speicher erstellen**.

1. Führen Sie in der **Bash-Eingabeaufforderung** im **Cloud Shell-Bereich** die folgenden Befehle aus, um die Werte der Azure-Abonnement-ID und der Abonnementnamenattribute abzurufen:

    ```bash
    az account show --query id --output tsv
    az account show --query name --output tsv
    ```

    > **Hinweis**: Kopieren Sie beide Werte in eine Textdatei. Sie werden sie später in diesem Lab benötigen.

1. Führen Sie in der Bash-Eingabeaufforderung** im **Cloud Shell-Bereich** den folgenden Befehl aus, um einen Dienstprinzipal zu erstellen (ersetzen Sie den **myServicePrincipalName** durch eine beliebige Zeichenfolge aus Buchstaben und Ziffern) und **mySubscriptionID** durch Ihre Azure subscriptionId**:

    ```bash
    az ad sp create-for-rbac --name myServicePrincipalName \
                         --role contributor \
                         --scopes /subscriptions/mySubscriptionID
    ```

    > **Hinweis**: Der Befehl generiert eine JSON-Ausgabe. Speichern der Ausgabe in einer Textdatei Sie benötigen ihn später in diesem Lab.

1. Starten Sie als Nächstes auf dem Laborcomputer einen Webbrowser, navigieren Sie zum Azure DevOps **eShopOnWeb-Projekt** . Klicken Sie auf **Project Einstellungen>Service-Verbinden ionen (unter Pipelines)** und **Verbinden ion** neuer Dienst.

    ![Neue Dienstverbindung](images/new-service-connection.png)

1. Wählen Sie im Bildschirm **Neue Dienstverbindung** die Option **Azure Resource Manager** und anschließend **Weiter** aus.

1. Wählen Sie **den Dienstprinzipal (manuell)** aus, und klicken Sie auf **"Weiter"**.

1. Füllen Sie die leeren Felder mit den informationen aus, die während der vorherigen Schritte gesammelt wurden:
    - Abonnement-ID oder -Name.
    - Dienstprinzipal-ID (appId), Dienstprinzipalschlüssel (Kennwort) und Mandanten-ID (Mandant).
    - Geben Sie unter **"Dienstverbindungsname****" Azure-Untere ein**. Auf diesen Namen wird in YAML-Pipelines verwiesen, wenn ein Azure DevOps Service Verbinden ion erforderlich ist, um mit Ihrem Azure-Abonnement zu kommunizieren.

    ![Azure-Serviceverbindung](images/azure-service-connection.png)

1. Klicken Sie auf **Überprüfen und speichern**.

#### Aufgabe 2: Bereitstellen von Ressourcen in Azure durch YAML-Pipelines
1. Navigieren Sie zurück zum **Bereich "Pipelines** " im **Pipelinehub** .
1. Klicken Sie im **Fenster "Erste Pipeline** erstellen" auf **"Pipeline erstellen"**.

    > **Hinweis**: Wir verwenden den Assistenten, um eine neue YAML-Pipelinedefinition basierend auf unserem Projekt zu erstellen.

1. Klicken Sie im **Bereich "Wo befindet sich Ihr Code?** " auf die **Option "Azure Repos Git (YAML)** ".
1. Klicken Sie im **Bereich "Repository** auswählen" auf **"EShopOnWeb"**.
1. Wählen Sie auf dem Bildschirm **Pipeline konfigurieren** die Option **Vorhandene Azure Pipelines-YAML-Datei** aus.
1. Geben Sie im **Blatt "Auswählen einer vorhandenen YAML-Datei** " die folgenden Parameter an:
   - Branch: **main**
   - Pfad: **.ado/eshoponweb-cd-windows-cm.yml**
1. Klicken Sie auf **Konfigurieren**, um die Einstellungen zu speichern.
1. Wählen Sie im Abschnitt "Variablen" einen Namen für Ihre Ressourcengruppe aus, legen Sie den gewünschten Speicherort fest, und ersetzen Sie den Wert der Dienstverbindung durch eine Ihrer vorhandenen Dienstverbindungen, die Sie zuvor erstellt haben.
1. Klicken Sie im oberen rechten Corder auf die **Schaltfläche "Speichern und ausführen** ", und klicken Sie dann erneut auf "Speichern", und führen Sie **den Vorgang erneut aus** .

   ![Speichern und Ausführen der YAML-Pipeline nach Dem Vornehmen von Änderungen](./images/m06/saveandrun.png)

1. Warten Sie, bis die Bereitstellung abgeschlossen ist, und überprüfen Sie die Ergebnisse.
   ![Erfolgreiche Ressourcenbereitstellung in Azure mithilfe von YAML-Pipelines](./images/m06/deploy.png)

#### Übung 3: Entfernen der Azure-Lab-Ressourcen.

In dieser Aufgabe verwenden Sie Azure Cloud Shell, um die in dieser Übung bereitgestellten Azure-Ressourcen zu entfernen, um unnötige Gebühren zu vermeiden.

1. Öffnen Sie im Azure-Portal im **Cloud Shell**-Bereich die **Bash**-Sitzung.
1. Löschen Sie alle Ressourcengruppen, die Sie während der praktischen Übungen in diesem Modul erstellt haben, indem Sie den folgenden Befehl ausführen:

   ```bash
   az group list --query "[?starts_with(name,'AZ400-EWebShop-NAME')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
   ```

   > **Hinweis**: Der Befehl wird (dem --nowait-Parameter entsprechend) asynchron ausgeführt. Dies bedeutet, dass Sie zwar einen weiteren Azure CLI-Befehl in derselben Bash-Sitzung direkt im Anschluss ausführen können, es jedoch einige Minuten dauert, bis die Ressourcengruppen tatsächlich entfernt werden.

## Überprüfung

In dieser Übung haben Sie erfahren, wie Sie eine Azure Bicep-Vorlage erstellen, mithilfe eines Vorlagenmoduls modularisieren, die Standard Bereitstellungsvorlage so ändern, dass sie das Modul und aktualisierte Abhängigkeiten verwendet, und schließlich die Vorlagen mithilfe von YAML-Pipelines in Azure bereitstellen.
