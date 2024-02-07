---
lab:
  title: Konfigurieren von Pipelines-as-Code mit YAML
  module: 'Module 05: Implement a secure continuous deployment using Azure Pipelines'
---

# Konfigurieren von Pipelines-as-Code mit YAML

## Lab-Handbuch für Kursteilnehmer

## Labanforderungen

- Für dieses Lab ist **Microsoft Edge** oder ein von [Azure DevOps unterstützter Browser](https://docs.microsoft.com/azure/devops/server/compatibility) erforderlich.

- **Einrichten einer Azure DevOps-Organisation**: Wenn Sie nicht bereits eine Azure DevOps-Organisation haben, die Sie für dieses Lab verwenden können, müssen Sie diese erstellen, indem Sie die unter [Erstellen einer Organisation oder Projektsammlung](https://docs.microsoft.com/azure/devops/organizations/accounts/create-organization) beschriebenen Anweisungen befolgen.

- Identifizieren Sie ein vorhandenes Azure-Abonnement, oder erstellen Sie ein neues Abonnement.

- Stellen Sie sicher, dass Sie über ein Microsoft- oder Microsoft Entra-Konto mit der Rolle „Besitzer“ im Azure-Abonnement und der Rolle „Globaler Administrator“ im Microsoft Entra-Mandanten verfügen, der dem Azure-Abonnement zugeordnet ist. Ausführliche Informationen finden Sie in den Artikeln zum [Auflisten von Azure-Rollenzuweisungen mithilfe des Azure-Portals](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-list-portal) und [Anzeigen und Zuweisen von Administratorrollen in Azure Active Directory](https://docs.microsoft.com/azure/active-directory/roles/manage-roles-portal).

## Übersicht über das Labor

Viele Teams ziehen es vor, ihre Build- und Releasepipelines mit YAML zu definieren. Dies ermöglicht ihnen den Zugriff auf dieselben Pipeline-Funktionen wie denjenigen, die den visuellen Designer verwenden, aber mit einer Markup-Datei, die wie jede andere Quelldatei verwaltet werden kann. YAML-Builddefinitionen können zu einem Projekt hinzugefügt werden, indem die entsprechenden Dateien einfach zum Repositorystamm hinzugefügt werden. Azure DevOps bietet außerdem Standardvorlagen für gängige Projekttypen und einen YAML-Designer, um den Prozess der Definition von Build- und Release-Aufgaben zu vereinfachen.

## Ziele

In diesem Lab lernen Sie Folgendes:

- Konfigurieren Sie CI/CD-Pipelines als Code mit YAML in Azure DevOps.

## Geschätzte Zeit: 60 Minuten

## Anweisungen

### Übung 0: Konfigurieren der Voraussetzungen für das Lab

In dieser Übung richten Sie die Voraussetzungen für das Lab ein, das aus einem neuen Azure DevOps-Projekt mit einem Repository basierend auf dem [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb) besteht.

#### Aufgabe 1: (überspringen, wenn fertig) Erstellen und Konfigurieren des Teamprojekts

In dieser Aufgabe erstellen Sie ein **eShopOnWeb_MultiStageYAML** Azure DevOps-Projekt, das in mehreren Labss verwendet werden soll.

1. Öffnen Sie auf Ihrem Lab-Computer in einem Browserfenster Ihre Azure DevOps-Organisation. Klicken Sie auf **Neues Projekt**. Geben Sie Ihrem Projekt den Namen **eShopOnWeb_MultiStageYAML** und lassen Sie die anderen Felder auf Standardwerten. Klicken Sie auf **Erstellen**.

    ![Erstellen eines Projekts](images/create-project.png)

#### Aufgabe 2: (überspringen, wenn erledigt) Importieren von eShopOnWeb Git Repository

Bei dieser Aufgabe importieren Sie das eShopOnWeb Git-Repository, das von mehreren Labs verwendet wird.

1. Öffnen Sie auf Ihrem Lab-Computer in einem Browserfenster Ihre Azure DevOps-Organisation und das zuvor erstellte **eShopOnWeb_MultiStageYAML**-Projekt. Klicken Sie auf **Repos>Dateien**, **Repository importieren**. Klicken Sie auf **Importieren**. Fügen Sie im Fenster **Git Repository importieren** die folgende URL https://github.com/MicrosoftLearning/eShopOnWeb.git ein, und klicken Sie auf **Importieren**:

    ![Importieren eines Repositorys](images/import-repo.png)

2. Das Repository ist wie folgt organisiert:
    - Der Ordner **.ado** enthält Azure DevOps-YAML-Pipelines.
    - Der Ordner **.devcontainer** enthält ein Containersetup für die Entwicklung mithilfe von Containern (entweder lokal in VS Code oder über GitHub Codespaces).
    - Der Ordner **.azure** enthält eine Bicep- und ARM-Infrastruktur als Codevorlagen, die in einigen Labs verwendet werden.
    - Der Ordner **.github** enthält YAML GitHub-Workflow-Definitionen.
    - Der Ordner **src** enthält die .NET 7-Website, die für die Labszenarien verwendet wird.

#### Schritt 2: Erstellen von Azure-Ressourcen

In dieser Aufgabe verwenden Sie das Azure-Portal, um eine Azure-Web-App zu erstellen.

1. Starten Sie auf Ihrem Labcomputer einen Webbrowser, navigieren Sie zum [**Azure-Portal**](https://portal.azure.com), und melden Sie sich an. Verwenden Sie hierzu die Anmeldeinformationen eines Benutzerkontos, das in dem Abonnement, das Sie in diesem Lab verwenden, und das in dem in dem Microsoft Entra-Mandanten, der dem Abonnement zugeordnet ist, über die Rolle „Globaler Administrator“ verfügt.
2. Klicken Sie im Azure-Portal in der Symbolleiste auf das Symbol **Cloud Shell**, das sich direkt rechts neben dem Suchtextfeld befindet.
3. Wählen Sie bei Aufforderung zur Auswahl von **Bash** oder **PowerShell** die Option **Bash** aus.

    >**Hinweis**: Wenn Sie **Cloud Shell** zum ersten Mal starten und die Meldung **Für Sie wurde kein Speicher bereitgestellt** angezeigt wird, wählen Sie das in diesem Lab verwendete Abonnement aus, und klicken Sie dann auf **Speicher erstellen**.

    > **Hinweis:** Um eine Liste der Regionen und deren Alias zu erhalten, führen Sie den folgenden Befehl in der Azure Cloud Shell - Bash aus:

    ```bash
    az account list-locations -o table
    ```

4. Führen Sie in der **Bash-Eingabeaufforderung** im **Cloud Shell**-Bereich den folgenden Befehl aus, um eine Ressourcengruppe zu erstellen (ersetzen Sie den `<region>`-Platzhalter durch den Namen der Azure-Region, die Ihnen am nächsten ist, z. B. „centralus“, „westeurope“ oder eine andere Region Ihrer Wahl).

    ```bash
    LOCATION='<region>'
    ```

    ```bash
    RESOURCEGROUPNAME='az400m05l11-RG'
    az group create --name $RESOURCEGROUPNAME --location $LOCATION
    ```

5. Um einen Windows-App-Service-Plan zu erstellen, führen Sie den folgenden Befehl aus:

    ```bash
    SERVICEPLANNAME='az400m05l11-sp1'
    az appservice plan create --resource-group $RESOURCEGROUPNAME --name $SERVICEPLANNAME --sku B3
    ```

6. Erstellen Sie eine Web-App mit einem eindeutigen Namen.

    ```bash
    WEBAPPNAME=eshoponWebYAML$RANDOM$RANDOM
    az webapp create --resource-group $RESOURCEGROUPNAME --plan $SERVICEPLANNAME --name $WEBAPPNAME
    ```

    > **Anmerkung**: Tragen Sie den Namen der Web-App ein. Sie benötigen diese später in diesem Lab.

7. Schließen Sie die Azure Cloud Shell, lassen Sie aber das Azure-Portal im Browser geöffnet.

### Übung 1: Konfigurieren Sie CI/CD-Pipelines-as-Code mit YAML in Azure DevOps.

In dieser Übung werden Sie CI/CD-Pipelines-as-Code mit YAML in Azure DevOps konfigurieren.

#### Aufgabe 1: Hinzufügen einer YAML-Build-Definition

In dieser Aufgabe fügen Sie dem vorhandenen Projekt eine YAML-Builddefinition hinzu.

1. Navigieren Sie zurück zum Bereich **Pipelines** im **Pipelinehub**.
2. Klicken Sie im Fenster **Erste Pipeline erstellen** auf **Pipeline erstellen**.

    > **Hinweis**: Wir verwenden den Assistenten, um eine neue YAML-Pipelinedefinition basierend auf unserem Projekt zu erstellen.

3. Klicken Sie im Bereich **Wo befindet sich Ihr Code?** auf die Option **Azure Repos Git (YAML)**.
4. Klicken Sie im Bereich **Repository auswählen** auf **eShopOnWeb_MultiStageYAML**.
5. Scrollen Sie auf dem Bildschirm **Pipeline konfigurieren** nach unten und wählen Sie die Option **Vorhandene Azure Pipelines-YAML-Datei** aus.
6. Geben Sie im Blatt **Auswählen einer vorhandenen YAML-Datei** die folgenden Parameter an:
   - Branch: **main**
   - Pfad: **.ado/eshoponweb-ci.yml**
7. Klicken Sie auf **Weiter**, um die Einstellungen zu speichern.
8. Klicken Sie auf dem Bildschirm **Pipeline-YML überprüfen** auf **Ausführen**, um den Buildpipelineprozess zu starten.
9. Warten Sie auf die erfolgreiche Fertigstellung der Buildpipeline. Ignorieren Sie alle Warnungen bezüglich des Quellcodes selbst, da sie für diese Übung nicht relevant sind.

    > **Hinweis**: Jede Aufgabe aus der YAML-Datei steht zur Überprüfung zur Verfügung, einschließlich aller Warnungen und Fehler.

#### Aufgabe 2: Hinzufügen von Continuous Delivery zur YAML-Definition

In dieser Aufgabe fügen Sie der YAML-basierten Definition der Pipeline, die Sie in der vorherigen Aufgabe erstellt haben, Continuous Delivery hinzu.

> **Hinweis**: Da die Build- und Testprozesse erfolgreich sind, können wir nun die Bereitstellung zur YAML-Definition hinzufügen.

1. Klicken Sie im Pipelineausführungsbereich auf das Auslassungszeichen in der oberen rechten Ecke, und klicken Sie im Dropdownmenü auf **Pipeline bearbeiten**.
2. Navigieren Sie im Bereich mit dem Inhalt der Datei ** eShopOnWeb_MultiStageYAML/.ado/eshoponweb-ci.yml** zum Ende der Datei (Zeile 56), und drücken Sie die **Eingabetaste**, um eine neue, leere Zeile hinzuzufügen.
3. Fügen Sie in Zeile **57** den folgenden Inhalt hinzu, um die **Releasephase** in der YAML-Pipeline zu definieren.

    > **Hinweis**: Sie können welche Phasen auch immer Sie benötigen definieren, um den Pipelinefortschritt besser zu organisieren und nachzuverfolgen.

    ```yaml
    - stage: Deploy
      displayName: Deploy to an Azure Web App
      jobs:
      - job: Deploy
        pool:
          vmImage: 'windows-2019'
        steps:
    ```

4. Setzen Sie den Cursor in eine neue Zeile am Ende der YAML-Definition.

    > **Hinweis**: Dies ist der Ort, an dem neue Aufgaben hinzugefügt werden.

5. Suchen Sie in der Liste der Aufgaben auf der rechten Seite des Codebereichs nach der Aufgabe **Azure-App Service-Bereitstellung** und wählen Sie sie aus.
6. Geben Sie im Bereich **Azure-App Sevice-Bereitstellung** die folgenden Einstellungen an, und klicken Sie auf **Hinzufügen**:

    - Wählen Sie in der Dropdownliste für **Azure-Abonnements** das Azure-Abonnement aus, in dem Sie die Azure-Ressourcen zuvor im Lab bereitgestellt haben, klicken Sie auf **Autorisieren** und authentifizieren Sie sich bei Aufforderung mit demselben Benutzerkonto, das Sie während der Azure-Ressourcenbereitstellung verwendet haben.
    - Wählen Sie in der Dropdownliste **App Service-Name** den Namen der Web-App aus, die Sie zuvor im Lab bereitgestellt haben.
    - **Aktualisieren** Sie im Textfeld **Paket oder Ordner** den Standardwert auf `$(Build.ArtifactStagingDirectory)/**/Web.zip`.
7. Bestätigen Sie die Einstellungen im Bereich Assistent, indem Sie auf die Schaltfläche **Hinzufügen** klicken.

    > **Hinweis**: Dadurch wird der YAML-Pipelinedefinition automatisch die Bereitstellungsaufgabe hinzugefügt.

8. Der Codeausschnitt, der dem Editor hinzugefügt wurde, sollte ähnlich wie unten aussehen und für die Parameter azureSubscription und WebappName Ihren Namen aufweisen:

    ```yaml
        - task: AzureRmWebAppDeployment@4
          inputs:
            ConnectionType: 'AzureRM'
            azureSubscription: 'AZURE SUBSCRIPTION HERE (b999999abc-1234-987a-a1e0-27fb2ea7f9f4)'
            appType: 'webApp'
            WebAppName: 'eshoponWebYAML369825031'
            packageForLinux: '$(Build.ArtifactStagingDirectory)/**/Web.zip'
    ```

9. Überprüfen Sie, ob die Aufgabe als untergeordnetes Element der Aufgabe **Schritte** aufgeführt ist. Wenn nicht, wählen Sie alle Zeilen aus der hinzugefügten Aufgabe aus und drücken Sie zweimal die **Tab**-Taste, um sie vier Leerzeichen einzurücken, sodass sie als untergeordnetes Element der Aufgabe **Schritte** aufgeführt wird.

    > **Hinweis**: Der **parameter packageForLinux** ist im Kontext dieses Labs irreführend, ist aber für Windows oder Linux gültig.

    > **Hinweis**: Diese beiden Phasen werden standardmäßig unabhängig ausgeführt. Daher ist die Buildausgabe aus der ersten Phase möglicherweise nicht ohne zusätzliche Änderungen für die zweite Phase verfügbar. Um diese Änderungen zu implementieren, fügen wir eine neue Aufgabe hinzu, um das Bereitstellungsartefakt am Anfang der Bereitstellungsphase herunterzuladen.

10. Platzieren Sie den Cursor in der ersten Zeile unter dem **Schrittknoten** der **Bereitstellungsphase** und drücken Sie Enter, um eine neue leere Zeile hinzuzufügen (Zeile 64).
11. Suchen Sie im Bereich ** Aufgaben** nach der Aufgabe **Buildartefakte herunterladen**, und wählen Sie sie aus.
12. Geben Sie die folgenden Parameter für diese Aufgabe an:
    - Herunterladen von Artefakten erstellt von: **aktueller Build**
    - Download-Typ: **Spezifisches Artefakt**
    - Artefaktname: **Wählen Sie „Website“ aus der Liste** (oder **tippen Sie „Website“** direkt ein, wenn sie nicht automatisch in der Liste erscheint)
    - Zielverzeichnis: **$(Build.ArtifactStagingDirectory)**
13. Klicken Sie auf **Hinzufügen**.
14. Der hinzugefügte Codeschnipsel sollte etwa so aussehen wie unten:

    ```yaml
        - task: DownloadBuildArtifacts@0
          inputs:
            buildType: 'current'
            downloadType: 'single'
            artifactName: 'Website'
            downloadPath: '$(Build.ArtifactStagingDirectory)'
    ```

15. Wenn die YAML-Einrückung ausgeschaltet ist, drücken Sie zweimal die **Tab**-Taste, während die hinzugefügte Aufgabe noch im Editor ausgewählt ist, um sie um vier Leerzeichen einzurücken.

    > **Hinweis**: Auch hier können Sie zur besseren Lesbarkeit eine Leerzeile davor und danach einfügen.

16. Klicken Sie auf **Speichern**, im Bereich **Speichern**, klicken Sie erneut auf **Speichern**, um die Änderung direkt in die Hauptverzweigung zu übernehmen.

    > **Hinweis**: Da unser ursprüngliches CI-YAML nicht so konfiguriert wurde, dass automatisch ein neuer Build ausgelöst wird, müssen wir diesen manuell initiieren.

17. Navigieren Sie im linken Menü von Azure DevOps zu **Pipelines** und wählen Sie erneut **Pipelines**.
18. Öffnen Sie die Pipeline **EShopOnWeb_MultiStageYAML** und klicken Sie auf **Pipeline ausführen**.
19. Bestätigen Sie das **Ausführen** im sich öffnenden Fenster.
20. Beachten Sie die zwei dargestellten Phasen **Build .Net Core Solution** und **Deploy to Azure Web App**.
21. Warten Sie auf den Start der Pipeline und dann auf den erfolgreichen Abschluss der Build-Phase.
22. Sobald die Bereitstellungsphase gestartet werden soll, werden die **erforderlichen Berechtigungen** sowie eine orangefarbene Statusleiste angezeigt:

    ```text
    This pipeline needs permission to access a resource before this run can continue to Deploy to an Azure Web App
    ```

23. Klicken Sie auf **Ansicht**
24. Klicken Sie im Bereich **Warten auf Überprüfung** auf **Zulassen**.
25. Überprüfen Sie die Meldung im Fenster **Popup zulassen** und bestätigen Sie mit **Zulassen**.
26. Damit wird die Bereitstellungsphase eingeleitet. Warten Sie, bis die Bereitstellung erfolgreich abgeschlossen wurden.

     > **Hinweis**: Sollte die Bereitstellung aufgrund eines Problems mit der YAML-Pipeline-Syntax fehlschlagen, verwenden Sie dies als Referenz:

     ```yaml
    #NAME THE PIPELINE SAME AS FILE (WITHOUT ".yml")
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
          displayName: Test
          inputs:
            command: 'test'
            projects: 'tests/UnitTests/*.csproj'
        
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
        
        - task: PublishBuildArtifacts@1
          displayName: Publish Artifacts ADO - Bicep
          inputs:
            PathtoPublish: '$(Build.SourcesDirectory)/.azure/bicep/webapp.bicep'
            ArtifactName: 'Bicep'
            publishLocation: 'Container'
    
    - stage: Deploy
      displayName: Deploy to an Azure Web App
      jobs:
      - job: Deploy
        pool:
          vmImage: 'windows-2019'
        steps:
        - task: DownloadBuildArtifacts@0
          inputs:
            buildType: 'current'
            downloadType: 'single'
            artifactName: 'Website'
            downloadPath: '$(Build.ArtifactStagingDirectory)'
        - task: AzureRmWebAppDeployment@4
          inputs:
            ConnectionType: 'AzureRM'
            azureSubscription: 'AZURE SUBSCRIPTION HERE (b999999abc-1234-987a-a1e0-27fb2ea7f9f4)'
            appType: 'webApp'
            WebAppName: 'eshoponWebYAML369825031'
            packageForLinux: '$(Build.ArtifactStagingDirectory)/**/Web.zip'
    
    ```

#### Aufgabe 4: Die bereitgestellte Site überprüfen

1. Wechseln Sie zurück zum Webbrowser-Fenster, in dem das Azure-Portal angezeigt wird, und navigieren Sie zu dem Blatt, auf dem die Eigenschaften der Azure-Webanwendung angezeigt werden.
2. Klicken Sie auf dem Azure-Web-App-Blatt auf **Übersicht** und im Übersichts-Blatt auf **Durchsuchen**, um Ihre Site in einer neuen Webbrowser-Registerkarte zu öffnen.
3. Überprüfen Sie, ob die bereitgestellte Site wie erwartet in der neuen Browser-Registerkarte geladen wird und die EShopOnWeb E-Commerce-Website anzeigt.

### Übung 2: Umgebungseinstellungen für CI/CD-Pipelines-as-Code mit YAML in Azure DevOps konfigurieren

In dieser Übung fügen Sie Genehmigungen zu einer YAML-basierten Pipeline in Azure DevOps hinzu.

#### Aufgabe 1: Pipelineumgebungen einrichten

YAML-Pipelines-as-Code haben keine Release/Quality Gates, wie es bei Azure DevOps Classic Release Pipelines der Fall ist. Einige Ähnlichkeiten können jedoch für YAML-Pipelines-as-Code mithilfe von **Umgebungen** konfiguriert werden. In dieser Aufgabe verwenden Sie diesen Mechanismus, um Genehmigungen für die Build-Stufe zu konfigurieren.

1. Navigieren Sie im Azure DevOps-Projekt **EShopOnWeb_MultiStageYAML** zu **Pipelines**.
2. Wählen Sie im Menü "Pipelines" auf der linken Seite **Umgebungen**.
3. Klicken Sie auf **Umgebung erstellen**.
4. Fügen Sie im Bereich **Neue Umgebung** einen Namen für die Umgebung hinzu, der **Genehmigungen** lautet.
5. Wählen Sie unter **Ressourcen** die Option **Keine**.
6. Bestätigen Sie die Einstellungen durch Drücken der Schaltfläche **Erstellen**.
7. Sobald die Umgebung erstellt wurde, klicken Sie auf die Auslassungspunkte (...) neben der Schaltfläche "Ressource hinzufügen".
8. Wählen Sie **Genehmigungen und Überprüfungen** aus.
9. Wählen Sie auf der Seite **Erste Prüfung hinzufügen** die Option **Genehmigungen**.
10. Fügen Sie den Namen Ihres Azure DevOps-Benutzerkontos in das Feld **Genehmigende Person** ein.

    > **Hinweis:** In einem realen Szenario würde dies den Namen Ihres DevOps-Teams widerspiegeln, das an diesem Projekt arbeitet.

11. Bestätigen Sie die festgelegten Genehmigungseinstellungen, indem Sie die auf Schaltfläche **Erstellen** drücken.
12. Zuletzt müssen noch die notwendigen "Umgebung: Genehmigungen"-Einstellungen in den YAML-Pipeline-Code für die Bereitstellungsphase eingefügt werden. Navigieren Sie dazu zu **Repos**, suchen Sie den Ordner **.ado** und wählen Sie die Pipeline-as-Code-Datei **eshoponweb-ci.yml**.
13. Klicken Sie in der Inhaltsansicht auf die Schaltfläche **Bearbeiten**, um in den Bearbeitungsmodus zu wechseln.
14. Navigieren Sie zum Anfang des **Bereitstellungsauftrags** (-auftrag: Bereitstellen in Zeile 60)
15. Fügen Sie direkt darunter eine neue leere Zeile ein und fügen Sie den folgenden Ausschnitt hinzu:

    ```yaml
      environment: approvals
    ```

    Der resultierende Codeschnipsel sollte wie folgt aussehen:

    ```yaml
     jobs:
      - job: Deploy
        environment: approvals
        pool:
          vmImage: 'windows-2019'
    ```

16. Da die Umgebung eine bestimmte Einstellung einer Bereitstellungsphase ist, kann sie nicht von "Aufträgen" verwendet werden. Daher müssen wir einige zusätzliche Änderungen an der aktuellen Auftragsdefinition vornehmen.
17. Benennen Sie in Zeile **60** „- job: Deploy“ um in **- deployment: Deploy**
18. Fügen Sie als Nächstes unter Zeile **63** (vmImage: Windows-2019) eine neue leere Zeile hinzu.
19. Fügen Sie den folgende Yaml-Codeausschnitt ein:

    ```yaml
        strategy:
          runOnce:
            deploy:
    ```

20. Wählen Sie den Rest des Codeausschnitts (Zeile **67** bis zum Ende) aus, und verwenden Sie die **Tabulator**-Taste, um den YAML-Einzug zu berichtigen.

    Das resultierende YAML-Schnipsel sollte nun wie folgt aussehen und die **Bereitstellungsphase** widerspiegeln:

    ```yaml
    - stage: Deploy
      displayName: Deploy to an Azure Web App
      jobs:
      - deployment: Deploy
        environment: approvals
        pool:
          vmImage: 'windows-2019'
        strategy:
          runOnce:
            deploy:
              steps:
              - task: DownloadBuildArtifacts@0
                inputs:
                  buildType: 'current'
                  downloadType: 'single'
                  artifactName: 'Website'
                  downloadPath: '$(Build.ArtifactStagingDirectory)'
              - task: AzureRmWebAppDeployment@4
                inputs:
                  ConnectionType: 'AzureRM'
                  azureSubscription: 'AZURE SUBSCRIPTION HERE (b999999abc-1234-987a-a1e0-27fb2ea7f9f4)'
                  appType: 'webApp'
                  WebAppName: 'eshoponWebYAML369825031'
                  packageForLinux: '$(Build.ArtifactStagingDirectory)/**/Web.zip'
    ```

21. Bestätigen Sie die Änderungen an der YAML-Codedatei, indem Sie im auf **Commit** klicken und im angezeigten Commitbereich erneut auf **Commit** klicken.
22. Navigieren Sie links zum Azure DevOps-Projektmenü, wählen Sie **Pipelines** und danach **Pipelines** aus und achten Sie auf die zuvor verwendete **EshopOnWeb_MultiStageYAML**-Pipeline.
23. Öffnen Sie die Pipeline.
24. Klicken Sie auf **Pipeline ausführen**, um eine neue Pipelineausführung auszulösen. Bestätigen Sie, indem Sie auf **Ausführen** klicken.
25. Genau wie zuvor beginnt die Buildphase wie erwartet. Warten Sie, bis der Vorgang erfolgreich abgeschlossen ist.
26. Da die *environment:approvals* für die Bereitstellungsphase konfiguriert sind, wird als Nächstes eine Genehmigungsbestätigung angefordert, bevor sie gestartet wird.
27. Dies ist in der Pipelineansicht sichtbar, in der **Warten (0/1 Prüfungen bestanden)** angezeigt wird. Außerdem wird folgende Benachrichtigung angezeigt: **Genehmigung muss überprüft werden, bevor diese Ausführung weiterhin in einer Azure-Web-App bereitstellen kann**.
28. Klicken Sie auf die Schaltfläche **Anzeigen** neben der Nachricht.
29. Klicken Sie im angezeigten Bereich **Überprüfungen und manuelle Validierung für die Bereitstellung in Azure-Web-App** auf die Meldung **Genehmigung ausstehend**.
30. Klicken Sie auf **Approve**.
31. Auf diese Weise kann die Bereitstellungsphase gestartet und der Azure Web App-Quellcode erfolgreich bereitgestellt werden.

    > **Hinweis:** Obwohl in diesem Beispiel nur die Genehmigungen verwendet wurden, werden die anderen Prüfungen wie Azure Monitor, REST-API usw. auf ähnliche Weise verwendet.

### Übung 3: Entfernen der Azure-Laborressourcen

In dieser Übung entfernen Sie die in diesem Lab bereitgestellten Azure-Ressourcen, um unerwartete Gebühren zu vermeiden.

>**Hinweis**: Denken Sie daran, alle neu erstellten Azure-Ressourcen zu entfernen, die Sie nicht mehr verwenden. Durch das Entfernen nicht verwendeter Ressourcen wird sichergestellt, dass keine unerwarteten Gebühren anfallen.

#### Aufgabe 1: Entfernen der Azure Lab-Ressourcen

In dieser Aufgabe verwenden Sie Azure Cloud Shell, um die in diesem Lab bereitgestellten Azure-Ressourcen zu entfernen, um unnötige Gebühren zu vermeiden.

1. Öffnen Sie im Azure-Portal die **Bash**-Shell-Sitzung im Bereich **Cloud Shell**.
2. Listen Sie alle Ressourcengruppen auf, die während der Labs in diesem Modul erstellt wurden, indem Sie den folgenden Befehl ausführen:

    ```sh
    az group list --query "[?starts_with(name,'az400m05l11-RG')].name" --output tsv
    ```

3. Löschen Sie alle Ressourcengruppen, die Sie während der praktischen Übungen in diesem Modul erstellt haben, indem Sie den folgenden Befehl ausführen:

    ```sh
    az group list --query "[?starts_with(name,'az400m05l11-RG')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**Hinweis**: Der Befehl wird (dem --nowait-Parameter entsprechend) asynchron ausgeführt. Dies bedeutet, dass Sie zwar einen weiteren Azure CLI-Befehl in derselben Bash-Sitzung direkt im Anschluss ausführen können, es jedoch einige Minuten dauert, bis die Ressourcengruppen tatsächlich entfernt werden.

## Überprüfung

In diesem Lab haben Sie CI/CD-Pipelines als Code mit YAML in Azure DevOps konfiguriert.
