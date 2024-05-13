---
lab:
  title: Überwachen der Anwendungsleistung mit Azure Load Testing
  module: 'Module 08: Implement continuous feedback'
---

# Überwachen der Anwendungsleistung mit Azure Load Testing

## Lab-Handbuch für Kursteilnehmer

## Labanforderungen

- Für dieses Lab ist **Microsoft Edge** oder ein von [Azure DevOps unterstützter Browser](https://docs.microsoft.com/azure/devops/server/compatibility) erforderlich.

- **Einrichten einer Azure DevOps-Organisation**: Wenn Sie nicht bereits eine Azure DevOps-Organisation haben, die Sie für dieses Lab verwenden können, müssen Sie diese erstellen, indem Sie die unter [Erstellen einer Organisation oder Projektsammlung](https://docs.microsoft.com/azure/devops/organizations/accounts/create-organization) beschriebenen Anweisungen befolgen.

- Identifizieren Sie ein vorhandenes Azure-Abonnement, oder erstellen Sie ein neues Abonnement.

- Stellen Sie sicher, dass Sie über ein Microsoft- oder Microsoft Entra-Konto mit der Rolle „Besitzer“ im Azure-Abonnement und der Rolle „Globaler Administrator“ im Microsoft Entra-Mandanten verfügen, der dem Azure-Abonnement zugeordnet ist. Ausführliche Informationen finden Sie in den Artikeln zum [Auflisten von Azure-Rollenzuweisungen mithilfe des Azure-Portals](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-list-portal) und [Anzeigen und Zuweisen von Administratorrollen in Azure Active Directory](https://docs.microsoft.com/azure/active-directory/roles/manage-roles-portal#view-my-roles).

## Übersicht über das Labor

**Azure Load Testing** ist ein vollständig verwalteter Auslastungstestdienst, mit dem Sie eine hohe Auslastung generieren können. Der Dienst simuliert Datenverkehr für Ihre Anwendungen, unabhängig davon, wo sie gehostet werden. Fachkräfte in der Entwicklung und Qualitätssicherung sowie Tester*innen können damit die Leistung, Skalierbarkeit oder Kapazität einer Anwendung optimieren.
Erstellen Sie schnell einen Auslastungstest für Ihre Webanwendung mithilfe einer URL und ohne vorherige Kenntnisse von Testtools. Azure Load Testing abstrahiert Komplexität und Infrastruktur, um Ihre Auslastungstests nach Maß durchzuführen.
Für komplexere Auslastungstestszenarien können Sie einen Auslastungstest erstellen, indem Sie ein vorhandenes Apache JMeter-Testskript, ein beliebtes Open-Source-Tool für Auslastung und Leistung, wiederverwenden. Ihr Testplan kann beispielsweise aus mehreren Anwendungsanforderungen bestehen, Sie möchten Nicht-HTTP-Endpunkte aufrufen, oder Sie verwenden Eingabedaten und Parameter, um den Test dynamischer zu gestalten.

In diesem Lab erfahren Sie, wie Sie Azure Load Testing verwenden können, um Leistungstests mit einer Live ausgeführten Webanwendung mit verschiedenen Ladeszenarien zu simulieren. Schließlich erfahren Sie, wie Sie Azure Load Testing in Ihre CI/CD-Pipelines integrieren.

## Ziele

In diesem Lab lernen Sie Folgendes:

- Bereitstellen von Azure App Service-Webanwendungen.
- Erstellen und Ausführen einer YAML-basierten CI/CD-Pipeline.
- Bereitstellen von Azure Load Testing.
- Untersuchen der Leistung von Azure-Webanwendungen mithilfe von Azure Load Testing.
- Integrieren von Azure Load Testing in Ihre CI/CD-Pipelines.

## Geschätzte Zeit: 60 Minuten

## Anweisungen

### Übung 0: Konfigurieren der Voraussetzungen für das Lab

In dieser Übung richten Sie die Voraussetzungen für das Lab ein, das aus einem neuen Azure DevOps-Projekt mit einem Repository basierend auf dem [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb) besteht.

#### Aufgabe 1: (überspringen, wenn fertig) Erstellen und Konfigurieren des Teamprojekts

In dieser Aufgabe erstellen Sie ein **eShopOnWeb** Azure DevOps-Projekt, das von mehreren Labs verwendet werden soll.

1. Öffnen Sie auf Ihrem Lab-Computer in einem Browserfenster Ihre Azure DevOps-Organisation. Klicken Sie auf **Neues Projekt**. Geben Sie Ihrem Projekt den Namen **eShopOnWeb**, und wählen Sie **Scrum** in der Dropdownliste **Arbeitselementprozess** aus. Klicken Sie auf **Erstellen**.

    ![Erstellen eines Projekts](images/create-project.png)

#### Aufgabe 2: (überspringen, wenn erledigt) Importieren von eShopOnWeb Git Repository

Bei dieser Aufgabe importieren Sie das eShopOnWeb Git-Repository, das von mehreren Labs verwendet wird.

1. Öffnen Sie auf Ihrem Lab-Computer in einem Browserfenster Ihre Azure DevOps-Organisation und das zuvor erstellte **eShopOnWeb**-Projekt. Klicken Sie auf **Repos>Dateien**, **Importieren**. Fügen Sie im Fenster **Git Repository importieren** die folgende URL https://github.com/MicrosoftLearning/eShopOnWeb.git ein und klicken Sie auf **Importieren**:

    ![Importieren eines Repositorys](images/import-repo.png)

1. Das Repository ist wie folgt organisiert:
    - Der Ordner **.ado** enthält Azure DevOps-YAML-Pipelines
    - Der Ordner **.devcontainer** enthält ein Containersetup für die Entwicklung mithilfe von Containern (entweder lokal in VS Code oder über GitHub Codespaces).
    - Der Ordner **infra** enthält eine Bicep&ARM-Infrastruktur als Codevorlagen, die in einigen Labszenarien verwendet werden.
    - Der Ordner **.github** enthält YAML GitHub-Workflow-Definitionen.
    - Der Ordner **src** enthält die .NET 8-Website, die in den Labszenarien verwendet wird.

#### Schritt 2: Erstellen von Azure-Ressourcen

In dieser Aufgabe erstellen Sie eine Azure-Webanwendung mithilfe der Cloud Shell in Azure-Portal.

1. Starten Sie auf Ihrem Labcomputer einen Webbrowser, navigieren Sie zum [**Azure-Portal**](https://portal.azure.com), und melden Sie sich an. Verwenden Sie hierzu die Anmeldeinformationen eines Benutzerkontos, das in dem Abonnement, das Sie in diesem Lab verwenden, und das in dem in dem Microsoft Entra-Mandanten, der dem Abonnement zugeordnet ist, über die Rolle „Globaler Administrator“ verfügt.
1. Klicken Sie im Azure-Portal in der Symbolleiste auf das Symbol **Cloud Shell**, das sich direkt rechts neben dem Suchtextfeld befindet.
1. Wählen Sie bei Aufforderung zur Auswahl von **Bash** oder **PowerShell** die Option **Bash** aus.
    >**Hinweis**: Wenn Sie **Cloud Shell** zum ersten Mal starten und die Meldung **Für Sie wurde kein Speicher bereitgestellt** angezeigt wird, wählen Sie das in diesem Lab verwendete Abonnement aus, und klicken Sie dann auf **Speicher erstellen**.

1. Führen Sie in der **Bash**-Eingabeaufforderung im **Cloud Shell**-Bereich den folgenden Befehl aus, um eine Ressourcengruppe zu erstellen (ersetzen Sie den `<region>`-Platzhalter durch den Namen der Azure-Region, die Ihnen am nächsten kommt, z. B. "eastus").

    ```bash
    RESOURCEGROUPNAME='az400m09l16-RG'
    LOCATION='<region>'
    az group create --name $RESOURCEGROUPNAME --location $LOCATION
    ```

1. Um einen Windows-App-Service-Plan zu erstellen, führen Sie den folgenden Befehl aus:

    ```bash
    SERVICEPLANNAME='az400l16-sp'
    az appservice plan create --resource-group $RESOURCEGROUPNAME \
        --name $SERVICEPLANNAME --sku B3
    ```

1. Erstellen Sie eine Web-App mit einem eindeutigen Namen.

    ```bash
    WEBAPPNAME=az400eshoponweb$RANDOM$RANDOM
    az webapp create --resource-group $RESOURCEGROUPNAME --plan $SERVICEPLANNAME --name $WEBAPPNAME
    ```

    > **Anmerkung**: Tragen Sie den Namen der Web-App ein. Sie benötigen diese später in diesem Lab.

### Übung 1: Konfigurieren Sie CI/CD-Pipelines-as-Code mit YAML in Azure DevOps.

In dieser Übung werden Sie CI/CD-Pipelines-as-Code mit YAML in Azure DevOps konfigurieren.

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

#### Aufgabe 2: Hinzufügen einer YAML-Build- und Bereitstellungsdefinition

In dieser Aufgabe fügen Sie dem vorhandenen Projekt eine YAML-Builddefinition hinzu.

1. Navigieren Sie zurück zum Bereich **Pipelines** im **Pipelinehub**.
1. Klicken Sie auf **Neue Pipeline** (oder „Pipeline erstellen“, wenn dies die erste ist, die Sie erstellen).

    > **Hinweis**: Wir verwenden den Assistenten, um eine neue YAML-Pipelinedefinition basierend auf unserem Projekt zu erstellen.

1. Klicken Sie im Bereich **Wo befindet sich Ihr Code?** auf die Option **Azure Repos Git (YAML)**.
1. Klicken Sie im Bereich **Ein Repository auswählen** auf **eShopOnWeb**.
1. Scrollen Sie im Bereich **Pipeline konfigurieren** herunter und wählen Sie **Starterpipeline** aus.
1. **Wählen** Sie alle Zeilen aus der Starterpipeline aus, und löschen Sie sie.
1. **Kopieren** Sie die vollständige Vorlagenpipeline von unten. Beachten Sie, dass Sie **vor dem Speichern** der Änderungen Parameteränderungen vornehmen müssen:

    ```yml
    
    #Template Pipeline for CI/CD
    # trigger:
    # - main

    resources:
      repositories:
        - repository: self
          trigger: none

    stages:
    - stage: Build
      displayName: Build .Net Core Solution
      jobs:
      - job: Build
        pool:
          vmImage: ubuntu-latest
        steps:
        - task: DotNetCoreCLI@2
          displayName: Restore
          inputs:
            command: 'restore'
            projects: '**/*.sln'
            feedsToUse: 'select'

        - task: DotNetCoreCLI@2
          displayName: Build
          inputs:
            command: 'build'
            projects: '**/*.sln'

        - task: DotNetCoreCLI@2
          displayName: Publish
          inputs:
            command: 'publish'
            publishWebProjects: true
            arguments: '-o $(Build.ArtifactStagingDirectory)'

        - task: PublishBuildArtifacts@1
          displayName: Publish Artifacts ADO - Website
          inputs:
            pathToPublish: '$(Build.ArtifactStagingDirectory)'
            artifactName: Website

    - stage: Deploy
      displayName: Deploy to an Azure Web App
      jobs:
      - job: Deploy
        pool:
          vmImage: 'windows-2019'
        steps:
        - task: DownloadBuildArtifacts@1
          inputs:
            buildType: 'current'
            downloadType: 'single'
            artifactName: 'Website'
            downloadPath: '$(Build.ArtifactStagingDirectory)'

    ```

1. Setzen Sie den Cursor in eine neue Zeile am Ende der YAML-Definition. **Stellen Sie sicher, dass Sie den Cursor am Einzug der vorherigen Aufgabenebene positionieren**.

    > **Hinweis**: Dies ist der Ort, an dem neue Aufgaben hinzugefügt werden.

1. Klicken Sie auf der rechten Seite des Portals auf **Assistent anzeigen**. Suchen Sie in der Liste der Aufgaben nach der Aufgabe **Azure App Service-Bereitstellung**, und wählen Sie diese aus.
1. Geben Sie im Bereich **Azure-App Sevice-Bereitstellung** die folgenden Einstellungen an, und klicken Sie auf **Hinzufügen**:

    - wählen Sie in der Dropdown-Liste **Azure-Abonnement** die soeben erstellte Dienstverbindung.
    - Überprüfen Sie, dass **App Service-Typ** auf „Web-App unter Windows“ zeigt.
    - wählen Sie in der Dropdownliste **App Service-Name** den Namen der Web-App aus, die Sie zuvor im Lab bereitgestellt haben (**az400eshoponweb...).
    - **Aktualisieren** Sie im Textfeld **Paket oder Ordner** den Standardwert auf `$(Build.ArtifactStagingDirectory)/**/Web.zip`.
    - Erweitern Sie **Anwendungs- und Konfigurationseinstellungen**, und fügen Sie den Wert `-UseOnlyInMemoryDatabase true -ASPNETCORE_ENVIRONMENT Development` hinzu.
1. Bestätigen Sie die Einstellungen im Bereich Assistent, indem Sie auf die Schaltfläche **Hinzufügen** klicken.

    > **Hinweis**: Dadurch wird der YAML-Pipelinedefinition automatisch die Bereitstellungsaufgabe hinzugefügt.

1. Der Codeausschnitt, der dem Editor hinzugefügt wurde, sollte ähnlich wie unten aussehen und für die Parameter azureSubscription und WebappName Ihren Namen aufweisen:

    ```yml
        - task: AzureRmWebAppDeployment@4
          inputs:
            ConnectionType: 'AzureRM'
            azureSubscription: 'SERVICE CONNECTION NAME'
            appType: 'webApp'
            WebAppName: 'az400eshoponWeb369825031'
            packageForLinux: '$(Build.ArtifactStagingDirectory)/**/Web.zip'
            AppSettings: '-UseOnlyInMemoryDatabase true -ASPNETCORE_ENVIRONMENT Development'
    ```

    > **Hinweis**: Der **parameter packageForLinux** ist im Kontext dieses Labs irreführend, ist aber für Windows oder Linux gültig.

1. Bevor Sie die Aktualisierungen in der YML-Datei speichern, geben Sie ihnen einen eindeutigeren Namen. Oben im Fenster des YAML-Editors wird **EShopOnweb/azure-pipelines-#.yml** angezeigt. (wobei # eine Zahl ist, in der Regel 1, sie könnte aber in Ihrem Setup unterschiedlich sein.) Wählen Sie **diesen Dateinamen** aus, und benennen Sie ihn in **m09l16-pipeline.yml** um

1. Klicken Sie auf **Speichern**. Klicken Sie im Bereich **Speichern** erneut auf **Speichern**, um die Änderung direkt in den Masterzweig zu übertragen.

    > **Hinweis**: Da unser ursprüngliches CI-YAML nicht so konfiguriert wurde, dass automatisch ein neuer Build ausgelöst wird, müssen wir diesen manuell initiieren.

1. Navigieren Sie im linken Menü von Azure DevOps zu **Pipelines** und wählen Sie erneut **Pipelines**. Wählen Sie als Nächstes **Alle** aus, um alle Pipelinedefinitionen zu öffnen, nicht nur die zuletzt verwendeten.

    > **Hinweis**: Wenn Sie alle vorherigen Pipelines aus früheren Labübungen beibehalten haben, hat diese neue Pipeline möglicherweise den Standardsequenznamen **eShopOnWeb (#)** für die Pipeline wiederverwendet, wie im folgenden Screenshot gezeigt. Wählen Sie eine Pipeline aus (höchstwahrscheinlich die mit der höchsten Sequenznummer, wählen Sie „Bearbeiten“ aus, und überprüfen Sie, ob sie auf die Codedatei m09l16-pipeline.yml zeigt).

    ![Screenshot von Azure Pipelines mit eShopOnWeb-Ausführungen](images/m3/eshoponweb-m9l16-pipeline.png)

1. Bestätigen Sie die Ausführung dieser Pipeline, indem Sie im angezeigten Bereich auf **Ausführen** klicken, und bestätigen Sie dies, indem Sie erneut auf **Ausführen** klicken.
1. Beachten Sie die zwei dargestellten Phasen **Build .Net Core Solution** und **Deploy to Azure Web App**.
1. Warten Sie, bis die Pipeline gestartet wird.

1. **Ignorieren** Sie alle Warnungen, die während der Buildphase angezeigt werden. Warten Sie, bis die Buildphase erfolgreich abgeschlossen wurde. (Sie können die aktuelle Buildphase auswählen, um weitere Details aus den Protokollen anzuzeigen.)

1. Sobald die Bereitstellungsphase gestartet werden soll, werden die **erforderlichen Berechtigungen** sowie eine orangefarbene Statusleiste angezeigt:

    ```text
    This pipeline needs permission to access a resource before this run can continue to Deploy to an Azure Web App
    ```

1. Klicken Sie auf **Ansicht**
1. Klicken Sie im Bereich **Warten auf Überprüfung** auf **Zulassen**.
1. Überprüfen Sie die Meldung im Fenster **Popup zulassen** und bestätigen Sie mit **Zulassen**.
1. Damit wird die Bereitstellungsphase eingeleitet. Warten Sie, bis die Bereitstellung erfolgreich abgeschlossen wurden.

#### Aufgabe 2: Die bereitgestellte Website überprüfen

1. Wechseln Sie zurück zum Webbrowser-Fenster, in dem das Azure-Portal angezeigt wird, und navigieren Sie zu dem Blatt, auf dem die Eigenschaften der Azure-Webanwendung angezeigt werden.
1. Klicken Sie auf dem Azure-Web-App-Blatt auf **Übersicht** und im Übersichts-Blatt auf **Durchsuchen**, um Ihre Site in einer neuen Webbrowser-Registerkarte zu öffnen.
1. Stellen Sie sicher, dass die bereitgestellte Website auf der neuen Browserregisterkarte mit der E-Commerce-Website „eShopOnWeb“ wie erwartet geladen wird.

### Übung 2: Bereitstellen und Einrichten von Azure Load Testing

In dieser Übung stellen Sie eine Azure Load Testing-Ressource in Azure bereit und konfigurieren unterschiedliche Auslastungstestszenarien für Ihren live ausgeführten Azure-App Service.

#### Aufgabe 1: Bereitstellen von Azure Load Testing

In dieser Aufgabe werden Sie eine Azure Load Test-Ressource in Ihrem Azure-Abonnement bereitstellen.

1. Navigieren Sie im Azure-Portal (<https://portal.azure.com>) zu **Azure-Ressource erstellen**.
1. Geben Sie im Suchfeld „Suchdienste und Marktplatz“ **Azure Load Testing** ein.
1. Wählen Sie **Azure Load Testing** (veröffentlicht von Microsoft) aus den Suchergebnissen.
1. Klicken Sie auf der Seite Azure Load Testing auf **Erstellen**, um den Bereitstellungsprozess zu starten.
1. Geben Sie auf der Seite „Auslastungstestressource erstellen“ die erforderlichen Details für die Ressourcenbereitstellung an:
   - **Abonnement**: Wählen Sie Ihr Azure-Abonnement aus.
   - **Ressourcengruppe**: Wählen Sie die Ressourcengruppe aus, die Sie für die Bereitstellung des Web App Service in der vorherigen Übung verwendet haben.
   - **Name**: eShopOnWebLoadTesting
   - **Region**: Wählen Sie unter Region eine Region in Ihrer Nähe aus.

    > **Hinweis**: Der Azure Load Testing-Dienst ist nicht in allen Azure-Regionen verfügbar.

1. Klicken Sie auf **Überprüfen und Erstellen**, um Ihre Einstellungen validieren zu lassen.
1. Klicken Sie auf **Erstellen**, um diese zu bestätigen, und stellen Sie die Azure Load Testing-Ressource bereit.
1. Die Seite „Bereitstellung ist in Bearbeitung“ wird angezeigt. Warten Sie ein paar Minuten, bis die Bereitstellung erfolgreich abgeschlossen ist.
1. Klicken Sie auf der Seite „Bereitstellungsfortschritt“ auf **Zur Ressource wechseln**, um zur Azure Load Testing-Ressource **eShopOnWebLoadTesting** zu navigieren.

    > **Hinweis**: Wenn Sie die Seite oder das Azure-Portal während der Bereitstellung der Azure Load Testing-Ressource geschlossen haben, können Sie sie erneut mit dem Azure-Portal-Suchfeld oder in der Liste Ressourcen/Zuletzt verwendete Ressourcen finden.

#### Aufgabe 2: Erstellen von Azure Load Testing-Tests

In dieser Aufgabe erstellen Sie verschiedene Azure Load Testing-Tests mit unterschiedlichen Einstellungen für die Auslastungskonfiguration.

1. Navigieren Sie im Blatt der Azure Load Testing-Ressource **eShopOnWebLoadTesting** zu **Tests**. Klicken Sie auf die Menüoption **+Erstellen**, und wählen Sie **URL-basierten Test erstellen** aus.
1. Füllen Sie die folgenden Parameter und Einstellungen aus, um einen Auslastungstest zu erstellen:
   - **Testen der URL**: Geben Sie die URL aus dem Azure App Service ein, den Sie in der vorherigen Übung bereitgestellt haben (az400eshoponweb... azurewebsites.net), **einschließlich https://**
   - **Auslastung angeben**: Virtuelle Benutzer
   - **Anzahl der virtuellen Benutzer**: 50
   - **Testdauer (Minuten)**: 5
   - **Hochlaufzeit (Minuten)**:  1

1. Bestätigen Sie die Konfiguration des Tests, indem Sie auf **Überprüfen und Erstellen** klicken,  (Nehmen Sie keine Änderungen an den anderen Registerkarten vor). Klicken Sie erneut auf **Erstellen**.
1. Dadurch werden die Auslastungstests gestartet, die den Test während 5 Minuten ausführen werden.
1. Navigieren Sie während der Ausführung des Tests zurück zur der Azure Load Testing-Ressource **eShopOnWebLoadTesting**, und navigieren Sie zu **Tests**, wählen Sie **Tests** aus, und Sie sehen einen Test **Get_eshoponweb...**
1. Klicken Sie im obersten Menü auf **Erstellen**, **URL-basierten Test erstellen**, um einen zweiten Auslastungstest zu erstellen.
1. Füllen Sie die folgenden Parameter und Einstellungen aus, um einen weiteren Auslastungstest zu erstellen:
   - **Testen der URL**: Geben Sie die URL aus dem Azure App Service ein, den Sie in der vorherigen Übung bereitgestellt haben (eShopOnWeb...azurewebsites.net), **einschließlich https://**
   - **Auslastung angeben**: Anfragen pro Sekunde (RPS)
   - **Anfragen pro Sekunde (RPS)**: 100
   - **Antwortzeit (Millisekunden)**: 500
   - **Testdauer (Minuten)**: 5
   - **Hochlaufzeit (Minuten)**:  1

1. Bestätigen Sie die Konfiguration des Tests, indem Sie auf **Überprüfen und Erstellen** klicken und noch einmal auf **Erstellen**.
1. Der Test wird ungefähr 5 Minuten lang ausgeführt.

#### Aufgabe 3: Überprüfen der Ergebnisse von Azure Load Testing

In dieser Aufgabe überprüfen Sie das Ergebnis eines Azure Load Testing-Testlaufs.

Wenn beide Schnelltests abgeschlossen sind, nehmen wir einige Änderungen daran vor und überprüfen die Ergebnisse.

1. Navigieren Sie von **Azure Load Testing** zu **Tests**. Wählen Sie eine der Testdefinitionen aus, um eine detailliertere Ansicht zu öffnen, indem Sie auf einen der Tests **klicken**. Sie werden dann auf die detailliertere Testseite weitergeleitet. Von hier aus können Sie die Details der tatsächlichen Läufe überprüfen, indem Sie die **TestRun_mm/dd/yyy-hh:hh** aus der resultierenden Liste auswählen.
1. Ermitteln Sie auf der detaillierten **TestRun**-Seite das tatsächliche Ergebnis der Azure Load Testing-Simulation. Einige der Werte sind:
   - Load / Anzahl von Anforderungen
   - Dauer
   - Antwortzeit (zeigt das Ergebnis in Sekunden an, das die Antwortzeit des 90. Perzentils widerspiegelt, d. h. die Antwortzeit für 90 % der Anfragen liegt innerhalb der angegebenen Ergebnisse)
   - Durchsatz in Anforderungen pro Sekunde

1. Weiter unten werden einige dieser Werte mithilfe von Dashboard-Diagrammlinien und -Diagrammansichten dargestellt.
1. Nehmen Sie sich ein paar Minuten Zeit, um **die Ergebnisse** der beiden simulierten Tests zu vergleichen und **die Auswirkungen** einer größeren Anzahl von Benutzer*innen auf die Leistung des App Service zu ermitteln.

### Übung 2: Automatisieren eines Auslastungstests mit CI/CD in Azure Pipelines

Sie beginnen mit der Automatisierung von Auslastungstests in Azure Load Testing, indem Sie sie einer CI/CD-Pipeline hinzufügen. Nachdem Sie einen Auslastungstest im Azure-Portal durchgeführt haben, exportieren Sie die Konfigurationsdateien und konfigurieren eine CI/CD-Pipeline in Azure Pipelines (eine ähnliche Funktion gibt es für GitHub Actions).

Nach Abschluss dieser Übung verfügen Sie über einen CI/CD-Workflow, der für die Durchführung eines Auslastungstests mit Azure Load Testing konfiguriert ist.

#### Aufgabe 1: Die Details der ADO-Dienstverbindung identifizieren

In dieser Aufgabe erteilen Sie dem Dienstprinzipal der Azure DevOps-Dienstverbindung die erforderlichen Berechtigungen.

1. Navigieren Sie im **Azure DevOps-Portal**(<https://dev.azure.com>) zum Projekt **eShopOnWeb**.
1. Wählen Sie links unten **Projekteinstellungen**.
1. Wählen Sie im Abschnitt **Pipelines** die Option **Dienstverbindungen**.
1. Achten Sie auf die Dienstverbindung, die den Namen Ihres Azure-Abonnements trägt, das Sie zu Beginn der Übung für die Bereitstellung von Azure-Ressourcen verwendet haben.
1. **Wählen Sie die Dienstverbindung aus**. Navigieren Sie auf der Registerkarte **Übersicht** zu **Details** und wählen Sie **Dienstprinzipal verwalten**.
1. Dies leitet Sie zum Azure-Portal weiter, von wo aus Sie die Details des **Dienstprinzipals** für das Identitätsobjekt öffnen.
1. Kopieren Sie den Wert des **Anzeigenamens** (formatiert wie Name_of_ADO_Organization_eShopOnWeb_-b86d9ae1-7552-4b75-a1e0-27fb2ea7f9f4) zur Seite, da Sie diesen in den nächsten Schritten benötigen werden.

#### Aufgabe 2: Erteilen von Berechtigungen für den Dienstprinzipal

Azure Load Testing verwendet Azure RBAC, um Berechtigungen für die Ausführung bestimmter Aktivitäten für Ihre Auslastungstestressource zu erteilen. Um einen Auslastungstest von Ihrer CI/CD-Pipeline aus durchzuführen, gewähren Sie dem Dienstprinzipal die Rolle **Auslastungstestmitwirkender**.

1. Navigieren Sie im **Azure-Portal** zu Ihrer **Azure Load Testing**-Ressource.
1. Wählen Sie **Zugriffssteuerung (IAM)** > Hinzufügen > Rollenzuweisung hinzufügen.
1. Wählen Sie auf der **Registerkarte "Rolle"** in der Liste der Auftragsfunktionsrollen die Option **Auslastungstestmitwirkender** aus.
1. Wählen Sie auf der Registerkarte **Mitglieder** die Option **Mitglieder auswählen** und verwenden Sie dann den **Anzeigenamen**, den Sie zuvor kopiert haben, um den Dienstprinzipal zu suchen.
1. Wählen Sie den **Dienstprinzipal** und anschließend die Option **Auswählen**.
1. Wählen Sie auf der Registerkarte **Überprüfen + zuweisen** die Option **Überprüfen + zuweisen** aus, um die Rollenzuweisung hinzuzufügen.

Sie können jetzt die Dienstverbindung in Ihrer Azure Pipelines-Workflowdefinition verwenden, um auf Ihre Azure-Auslastungstestressource zuzugreifen.

#### Aufgabe 3: Exportieren von Ladetesteingabedateien und Importieren in Azure Repos

Zum Ausführen eines Auslastungstests mit Azure Load Testing in einem CI/CD-Workflow müssen Sie die Konfigurationseinstellungen für den Auslastungstest und alle Eingabedateien in Ihrem Quellcodeverwaltungsrepository hinzufügen. Wenn Sie über einen vorhandenen Auslastungstest verfügen, können Sie die Konfigurationseinstellungen und alle Eingabedateien aus dem Azure-Portal herunterladen.

Führen Sie die folgenden Schritte aus, um die Eingabedateien für einen vorhandenen Auslastungstest im Azure-Portal herunterzuladen:

1. Navigieren Sie im **Azure-Portal** zu Ihrer **Azure Load Testing**-Ressource.
1. Wählen Sie im linken Bereich **Tests**aus, um die Liste der Auslastungstests anzuzeigen, und wählen Sie dann **Ihren Test**.
1. Wählen Sie die Auslassungspunkte (**...**) neben dem Testlauf aus, mit dem Sie arbeiten, und wählen Sie dann **Eingabedatei herunterladen** aus.
1. Der Browser lädt einen gezippten Ordner herunter, der die Eingabedateien für den Auslastungstest enthält.
1. Verwenden Sie ein beliebiges ZIP-Tool, um die Eingabedateien zu extrahieren. Der Ordner enthält die folgenden Dateien:

   - *config.yaml*: die YAML-Konfigurationsdatei für den Auslastungstest. Sie verweisen auf diese Datei in der CI/CD-Workflowdefinition.
   - *quick_test.jmx*: das JMeter-Testskript

1. Committen Sie alle extrahierten Eingabedateien in Ihr Quellcodeverwaltungsrepository. Navigieren Sie hierzu zum **Azure DevOps-Portal**(<https://dev.azure.com>), und navigieren Sie dann zum DevOps-Projekt **eShopOnWeb**.
1. Wählen Sie **Repositorys** aus. Beachten Sie in der Struktur des Quellcodeordners den Unterordner **Tests**. Beachten Sie die Auslassungspunkte (...), und wählen Sie **Neu > Ordner** aus.
1. Geben Sie **jmeter** als Ordnernamen und **placeholder.txt** als Dateinamen an (Hinweis: Ein Ordner kann nicht als leer erstellt werden)
1. Klicken Sie auf **Commit**, um die Erstellung der Platzhalterdatei und des jmeter-Ordners zu bestätigen.
1. Navigieren Sie aus der **Ordnerstruktur** zum neuen erstellten **jmeter**-Unterordner. Klicken Sie auf die **Auslassungspunkte (...)** und wählen Sie **Datei hochladen** aus.
1. Navigieren Sie mit der Option **Durchsuchen** zum Speicherort der extrahierten ZIP-Datei, und wählen Sie sowohl **config.yaml** als auch **quick_test.jmx** aus.
1. Klicken Sie auf **Commit**, um den Dateiupload in die Quellcodeverwaltung zu bestätigen.

#### Aufgabe 4: Aktualisieren der YAML-Definitionsdatei des CI/CD-Workflows

In dieser Aufgabe importieren Sie die Erweiterung „Azure Load Testing - Azure DevOps Marketplace“ sowie die vorhandene CI/CD-Pipeline mit der AzureLoadTest-Aufgabe.

1. Zum Erstellen und Ausführen eines Auslastungstests verwendet die Azure Pipelines-Workflowdefinition die Erweiterung **Azure Load Testing-Aufgabe** aus dem Azure DevOps-Marketplace. Öffnen Sie die [Azure Load Testing-Aufgabenerweiterung](https://marketplace.visualstudio.com/items?itemName=AzloadTest.AzloadTesting) im Azure DevOps-Marketplace und wählen Sie **Kostenlos abrufen** aus.
1. Wählen Sie Ihre Azure DevOps-Organisation aus und wählen Sie dann **Installieren** aus, um die Erweiterung zu installieren.
1. Navigieren Sie im Azure DevOps-Portal und im Projekt zu **Pipelines** und wählen Sie die am Anfang dieser Übung erstellte Pipeline aus. Klicken Sie auf **Bearbeiten**.
1. Navigieren Sie im YAML-Skript zu **Zeile 56** und drücken Sie ENTER/RETURN, um eine neue leere Zeile hinzuzufügen. (Dies ist direkt vor der Bereitstellungsphase der YAML-Datei).
1. Wählen Sie in Zeile 57 den Aufgaben-Assistenten rechts aus, und suchen Sie nach **Azure Load Testing**.
1. Füllen Sie den grafischen Bereich mit den richtigen Einstellungen für Ihr Szenario aus:
   - Azure-Abonnement: Wählen Sie das Abonnement, auf dem Ihre Azure-Ressourcen laufen.
   - Auslastungstestdatei: '$(Build.SourcesDirectory)/tests/jmeter/config.yaml'
   - Auslastungstest-Ressourcengruppe: Die Ressourcengruppe, die Ihre Azure Load Testing-Ressourcen enthält
   - Auslastungstest-Ressourcenname: ESHopOnWebLoadTesting
   - Name des Auslastungstestlaufs: ado_run
   - Beschreibung des Auslastungstestlaufs: Auslastungstest aus ADO

1. Bestätigen Sie die Einfügung der Parameter als YAML-Codeausschnitt, indem Sie auf **Hinzufügen** klicken.
1. Wenn beim Einzug des YAML-Codeschnipsels Fehler auftreten (rote Wellenlinien), beheben Sie diese, indem Sie zwei Leerzeichen oder Tabstopps hinzufügen, um den Codeschnipsel richtig zu positionieren.  
1. Der folgende Beispielausschnitt zeigt, wie der YAML-Code aussehen soll.

    ```yml
         - task: AzureLoadTest@1
          inputs:
            azureSubscription: 'AZURE DEMO SUBSCRIPTION(b86d9ae1-1234-4b75-a8e7-27fb2ea7f9f4)'
            loadTestConfigFile: '$(Build.SourcesDirectory)/tests/jmeter/config.yaml'
            resourceGroup: 'az400m05l11-RG'
            loadTestResource: 'eShopOnWebLoadTesting'
            loadTestRunName: 'ado_run'
            loadTestRunDescription: 'load testing from ADO'
    ```

1. Fügen Sie unter dem eingefügten YAML-Codeausschnitt eine neue leere Zeile hinzu, indem Sie die Enter-Taste drücken.
1. Fügen Sie unter dieser leeren Zeile einen Codeausschnitt für die Aufgabe „Veröffentlichen“ hinzu, die die Ergebnisse der Azure Load Test-Aufgabe während der Pipelineausführung anzeigt:

    ```yml
        - publish: $(System.DefaultWorkingDirectory)/loadTest
          artifact: loadTestResults
    ```

1. Wenn beim Einzug des YAML-Codeschnipsels Fehler auftreten (rote Wellenlinien), beheben Sie diese, indem Sie zwei Leerzeichen oder Tabstopps hinzufügen, um den Codeschnipsel richtig zu positionieren.  
1. Wenn beide Codeausschnitte zur CI/CD-Pipeline hinzugefügt wurden, **speichern** Sie die Änderungen.
1. Klicken Sie nach dem Speichern auf **Ausführen**, um die Pipeline auszulösen.
1. Bestätigen Sie den Branch (Main) und klicken Sie auf die Schaltfläche **Ausführen**, um die Pipelineausführung zu starten.
1. Klicken Sie auf der Seite Pipelinestatus auf den Status **Build**, um die ausführlichen Protokollierungsdetails der verschiedenen Aufgaben in der Pipeline zu öffnen.
1. Warten Sie, bis die Pipeline den Build-Status verlässt und zur **AzureLoadTest**-Aufgabe im Workflow der Pipeline gelangt.
1. Navigieren Sie während der Ausführung der Aufgabe zum **Azure Load Testing** im Azure-Portal, und sehen Sie, wie die Pipeline einen neuen RunTest namens **adoloadtest1** erstellt. Sie können ihn auswählen, um die Ergebniswerte des TestRun-Auftrags anzuzeigen.
1. Navigieren Sie zurück zur Ansicht Azure DevOps CI/CD Pipeline-Ausführung, in der die **AzureLoadTest-Aufgabe** erfolgreich abgeschlossen wurde. In der ausführlichen Protokollierungsausgabe werden auch die resultierenden Werte des Auslastungstests angezeigt:

    ```text
    Task         : Azure Load Testing
    Description  : Automate performance regression testing with Azure Load Testing
    Version      : 1.2.30
    Author       : Microsoft Corporation
    Help         : https://docs.microsoft.com/azure/load-testing/tutorial-cicd-azure-pipelines#azure-load-testing-task
    ==============================================================================
    Test '0d295119-12d0-482d-94be-a7b84787c004' already exists
    Uploaded test plan for the test
    Creating and running a testRun for the test
    View the load test run in progress at: https://portal.azure.com/#blade/Microsoft_Azure_CloudNativeTesting/NewReport//resourceId/%2fsubscriptions%4b75-a1e0-27fb2ea7f9f4%2fresourcegroups%2faz400m05l11-rg%2fproviders%2fmicrosoft.loadtestservice%2floadtests%2feshoponwebloadtesting/testId/0d295119-12d0-787c004/testRunId/161046f1-d2d3-46f7-9d2b-c8a09478ce4c
    TestRun completed
    
    -------------------Summary ---------------
    TestRun start time: Mon Jul 24 2023 21:46:26 GMT+0000 (Coordinated Universal Time)
    TestRun end time: Mon Jul 24 2023 21:51:50 GMT+0000 (Coordinated Universal Time)
    Virtual Users: 50
    TestStatus: DONE
    
    ------------------Client-side metrics------------
    
    Homepage
    response time        : avg=1359ms min=59ms med=539ms max=16629ms p(90)=3127ms p(95)=5478ms p(99)=13878ms
    requests per sec     : avg=37
    total requests       : 4500
    total errors         : 0
    total error rate     : 0
    Finishing: AzureLoadTest
    
    ```

1. Sie haben nun einen automatisierten Auslastungstest als Teil einer Pipelineausführung ausgeführt. In der letzten Aufgabe geben Sie Bedingungen für Fehler an, was bedeutet, dass unsere Bereitstellungsphase nicht gestartet werden kann, wenn die Leistung der Webanwendung unter einem bestimmten Schwellenwert liegt.

#### Aufgabe 5: Hinzufügen von Fehler-/Erfolgskriterien zur Auslastungstestpipleline

In dieser Aufgabe werden Sie die Kriterien für das Scheitern von Lasttests verwenden, um eine Warnung zu erhalten (eine fehlgeschlagene Pipeline als Ergebnis), wenn die Anwendung nicht Ihren Qualitätsanforderungen entspricht.

1. Navigieren Sie in Azure DevOps zum Projekt „eShopOnWeb“, und öffnen Sie **Repositorys**.
1. Navigieren Sie innerhalb von Repos zum Unterordner **/tests/jmeter**, der zuvor erstellt und verwendet wurde.
1. Öffnen Sie die Datei Load Testing *config.yaml**. Klicken Sie auf **Bearbeiten**, um die Bearbeitung der Datei zu ermöglichen.
1. Ersetzen Sie `failureCriteria: []` durch den folgenden Codeschnipsel:

    ```text
    failureCriteria:
      - avg(response_time_ms) > 300
      - percentage(error) > 50
    ```

1. Speichern Sie die Änderungen an der config.yaml, indem Sie auf **Commit** und ein weiteres Mal auf „Commit“ klicken.
1. Navigieren Sie zurück zu **Pipelines**, und führen Sie die Pipeline **eShopOnWeb** erneut aus. Nach ein paar Minuten wird der Lauf mit dem Status **Fehlgeschlagen** für die Aufgabe **AzureLoadTest** abgeschlossen.
1. Öffnen Sie die ausführliche Protokollierungsansicht für die Pipeline, und überprüfen Sie die Details des **AzureLoadtest**. Nachfolgend sehen Sie eine Beispielausgabe:

    ```text
    Creating and running a testRun for the test
    View the load test run in progress at: https://portal.azure.com/#blade/Microsoft_Azure_CloudNativeTesting/NewReport//resourceId/%2fsubscriptions%2fb86d9ae1-7552-47fb2ea7f9f4%2fresourcegroups%2faz400m05l11-rg%2fproviders%2fmicrosoft.loadtestservice%2floadtests%2feshoponwebloadtesting/testId/0d295119-12d0-a7b84787c004/testRunId/f4bec76a-8b49-44ee-a388-12af34f0d4ec
    TestRun completed
    
    -------------------Summary ---------------
    TestRun start time: Mon Jul 24 2023 23:00:31 GMT+0000 (Coordinated Universal Time)
    TestRun end time: Mon Jul 24 2023 23:06:02 GMT+0000 (Coordinated Universal Time)
    Virtual Users: 50
    TestStatus: DONE
    
    -------------------Test Criteria ---------------
    Results          :1 Pass 1 Fail
    
    Criteria                     :Actual Value        Result
    avg(response_time_ms) > 300                       1355.29               FAILED
    percentage(error) > 50                                                  PASSED
    
    
    ------------------Client-side metrics------------
    
    Homepage
    response time        : avg=1355ms min=58ms med=666ms max=16524ms p(90)=2472ms p(95)=5819ms p(99)=13657ms
    requests per sec     : avg=37
    total requests       : 4531
    total errors         : 0
    total error rate     : 0
    ##[error]TestResult: FAILED
    Finishing: AzureLoadTest

    ```

1. Beachten Sie, dass die letzte Zeile der Lasttest-Ausgabe **##[error]TestResult: FAILED** ist; da wir ein **FailCriteria** mit einer durchschnittlichen Antwortzeit von > 300 oder einem Fehlerprozentsatz von > 20 definiert haben und nun eine durchschnittliche Antwortzeit von mehr als 300 sehen, wird die Aufgabe als fehlgeschlagen gekennzeichnet.

    > Hinweis: Stellen Sie sich vor, Sie würden in einem realen Szenario die Leistung Ihres App Service überprüfen, und wenn die Leistung unter einem bestimmten Schwellenwert liegt (was in der Regel bedeutet, dass die Web App stärker auslastet wird), könnten Sie eine neue Bereitstellung für einen zusätzlichen Azure App Service auslösen. Da wir die Reaktionszeit für Azure-Laborumgebungen nicht kontrollieren können, haben wir beschlossen, die Logik umzukehren, um den Ausfall zu garantieren.

1. Der FAILED-Status des Pipelinevorgangs spiegelt tatsächlich einen ERFOLG der Überprüfung der Anforderungskriterien für Azure Load Testing wider.

### Übung 3: Entfernen der Azure-Laborressourcen

In dieser Übung entfernen Sie die in diesem Lab bereitgestellten Azure-Ressourcen, um unerwartete Gebühren zu vermeiden.

> **Hinweis**: Denken Sie daran, alle neu erstellten Azure-Ressourcen zu entfernen, die Sie nicht mehr verwenden. Durch das Entfernen nicht verwendeter Ressourcen wird sichergestellt, dass keine unerwarteten Gebühren anfallen.

#### Aufgabe 1: Entfernen der Azure Lab-Ressourcen

In dieser Aufgabe verwenden Sie Azure Cloud Shell, um die in diesem Lab bereitgestellten Azure-Ressourcen zu entfernen, um unnötige Gebühren zu vermeiden.

1. Öffnen Sie im Azure-Portal die **Bash**-Shell-Sitzung im Bereich **Cloud Shell**.
1. Listen Sie alle Ressourcengruppen auf, die während der Labs in diesem Modul erstellt wurden, indem Sie den folgenden Befehl ausführen:

    ```sh
    az group list --query "[?starts_with(name,'az400m09l16')].name" --output tsv
    ```

1. Löschen Sie alle Ressourcengruppen, die Sie während der praktischen Übungen in diesem Modul erstellt haben, indem Sie den folgenden Befehl ausführen:

    ```sh
    az group list --query "[?starts_with(name,'az400m09l16')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**Hinweis**: Der Befehl wird (dem --nowait-Parameter entsprechend) asynchron ausgeführt. Dies bedeutet, dass Sie zwar einen weiteren Azure CLI-Befehl in derselben Bash-Sitzung direkt im Anschluss ausführen können, es jedoch einige Minuten dauert, bis die Ressourcengruppen tatsächlich entfernt werden.

## Überprüfung

In dieser Übung haben Sie eine Web-App für Azure-App Dienst mithilfe von Azure-Pipelines bereitgestellt sowie eine Azure Load Testing-Ressource mit TestRuns bereitgestellt. Als Nächstes haben Sie die Datei JMeter Load Testing config.yaml in die Azure Repos-Quellcodeverwaltung integriert und Ihre CI/CD-Pipeline mit Azure Load Testing erweitert. In der letzten Übung haben Sie gelernt, wie Sie die Erfolgskriterien von LoadTest definieren.
