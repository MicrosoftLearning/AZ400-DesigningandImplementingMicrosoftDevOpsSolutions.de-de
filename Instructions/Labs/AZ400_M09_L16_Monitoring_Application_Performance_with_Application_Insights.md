---
lab:
  title: Testen der Anwendungsleistung mit Azure Load Testing
  module: 'Module 09: Implement continuous feedback'
---

# Überwachung und Leistungsverwaltung für Anwendungen mit Application Insights

## Lab-Handbuch für Kursteilnehmer

## Labanforderungen

- Für dieses Lab ist **Microsoft Edge** oder ein [von Azure DevOps unterstützter Browser](https://docs.microsoft.com/azure/devops/server/compatibility) erforderlich.

- **Einrichten einer Azure DevOps-Organisation**: Wenn Sie nicht bereits eine Azure DevOps-Organisation haben, die Sie für dieses Lab verwenden können, müssen Sie diese erstellen, indem Sie die unter [Erstellen einer Organisation oder Projektsammlung](https://docs.microsoft.com/azure/devops/organizations/accounts/create-organization) beschriebenen Anweisungen befolgen.

- Identifizieren Sie ein vorhandenes Azure-Abonnement, oder erstellen Sie ein neues Abonnement.

- Stellen Sie sicher, dass Sie über ein Microsoft- oder Microsoft Entra-Konto mit der Rolle „Besitzer“ im Azure-Abonnement und der Rolle „Globaler Administrator“ im Microsoft Entra-Mandanten verfügen, der dem Azure-Abonnement zugeordnet ist. Ausführliche Informationen finden Sie in den Artikeln zum [Auflisten von Azure-Rollenzuweisungen mithilfe des Azure-Portals](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-list-portal) und [Anzeigen und Zuweisen von Administratorrollen in Azure Active Directory](https://docs.microsoft.com/azure/active-directory/roles/manage-roles-portal#view-my-roles).

## Übersicht über das Labor

**Azure Load Testing** ist ein vollständig verwalteter Auslastungstestdienst, mit dem Sie eine hohe Auslastung generieren können. Der Dienst simuliert Datenverkehr für Ihre Anwendungen, unabhängig davon, wo sie gehostet werden. Fachkräfte in der Entwicklung und Qualitätssicherung sowie Tester*innen können damit die Leistung, Skalierbarkeit oder Kapazität einer Anwendung optimieren.
Erstellen Sie schnell einen Auslastungstest für Ihre Webanwendung mithilfe einer URL und ohne vorherige Kenntnisse von Testtools. Azure Load Testing abstrahiert Komplexität und Infrastruktur, um Ihre Auslastungstests nach Maß durchzuführen.
Für komplexere Auslastungstestszenarien können Sie einen Auslastungstest erstellen, indem Sie ein vorhandenes Apache JMeter-Testskript, ein beliebtes Open-Source-Tool für Auslastung und Leistung, wiederverwenden. Ihr Testplan kann beispielsweise aus mehreren Anwendungsanforderungen bestehen, Sie möchten Nicht-HTTP-Endpunkte aufrufen, oder Sie verwenden Eingabedaten und Parameter, um den Test dynamischer zu gestalten.

In dieser Übung erfahren Sie, wie Sie Azure Load Testing verwenden können, um Leistungstests mit einer live ausgeführten Webanwendung mit verschiedenen Ladeszenarien zu simulieren. Schließlich erfahren Sie, wie Sie Azure Load Testing in Ihre CI/CD-Pipelines integrieren. 

## Ziele

In diesem Lab lernen Sie Folgendes:

- Stellen Sie Azure App Service-Webanwendungen bereit.
- Verfassen und Ausführen einer YAML-basierten CI/CD-Pipeline.
- Wählen Sie Azure Load Testing aus.
- Untersuchen Sie die Leistung von Azure Web App mithilfe von Azure Load Testing.
- Integrieren von Azure Load Testing und Azure Chaos Studio in Ihren CI/CD-Pipelines

## Geschätzte Zeit: 60 Minuten

## Anweisungen

### Übung 0: Konfigurieren der Voraussetzungen für das Lab.

In dieser Übung richten Sie die Voraussetzungen für das Labor ein, das aus einem neuen Azure DevOps-Projekt mit einem Repository basierend auf dem [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb) besteht.

#### Aufgabe 1: (überspringen, wenn fertig) Erstellen und Konfigurieren des Teamprojekts

In dieser Aufgabe erstellen Sie ein **eShopOnWeb** Azure DevOps-Projekt, das von mehreren Laboren verwendet werden soll.

1. Öffnen Sie auf Ihrem Laborcomputer in einem Browserfenster Ihre Azure DevOps-Organisation. Klicken auf „Neues Projekt“ Geben Sie Ihrem Projekt den Namen **"eShopOnWeb** ", und wählen Sie **"Scrum** " in der **Dropdownliste "Arbeitsaufgabe"** aus. Klicken Sie auf **Erstellen**.

    ![Erstellen eines Projekts](images/create-project.png)

#### Aufgabe 2: (überspringen, wenn erledigt) Importieren von eShopOnWeb Git Repository

Bei dieser Aufgabe importieren Sie das eShopOnWeb Git-Repository, das von mehreren Labs verwendet wird.

1. Öffnen Sie auf Ihrem Laborcomputer in einem Browserfenster Ihre Azure DevOps-Organisation und das zuvor erstellte **eShopOnWeb-Projekt** . Klicken Sie auf **Repos>Files** , **Import**. Fügen Sie im **Fenster "Git Repository** importieren" die folgende URL https://github.com/MicrosoftLearning/eShopOnWeb.git  ein, und klicken Sie auf " **Importieren**":

    ![Importieren eines Repositorys](images/import-repo.png)

2. Das Repository ist wie folgt organisiert:
    - Der Ordner „.ado“ enthält Azure DevOps-YAML-Pipelines.
    - Der Ordner „.devcontainer“ enthält ein Containersetup für die Entwicklung mithilfe von Containern (entweder lokal in VS Code oder über GitHub Codespaces).
    - **Azure-Ordner** enthält Bicep&ARM-Infrastruktur als Codevorlagen, die in einigen Lab-Szenarien verwendet werden.
    - **GITHUB-Ordnercontainer-YAML-GitHub-Workflowdefinitionen** .
    - Der Ordner „src“ enthält die .NET 6-Website, die in den Labszenarien verwendet wird.

#### Schritt 2: Erstellen von Azure-Ressourcen

In dieser Aufgabe erstellen Sie eine Azure Web App mithilfe der Cloudshell in Azure-Portal.

1. Starten Sie auf Ihrem Labcomputer einen Webbrowser, navigieren Sie zum [**Azure-Portal**](https://portal.azure.com), und melden Sie sich an. Verwenden Sie hierzu die Anmeldeinformationen eines Benutzerkontos, das in dem Abonnement, das Sie in diesem Lab verwenden, über die Rolle „Besitzer“ und in dem Azure AD-Mandanten, der dem Azure-Abonnement zugeordnet ist, über die Rolle „Globaler Administrator“ verfügt.
2. Öffnen Sie im Azure-Portal den Bereich **Cloud Shell**, indem Sie auf das Symbolleistensymbol rechts neben dem Textfeld für die Suche klicken.
3. Wählen Sie bei Aufforderung zur Auswahl von **Bash** oder **PowerShell** die Option **Bash** aus.
    >**Hinweis**: Wenn Sie **Cloud Shell** zum ersten Mal starten und die Meldung **Für Sie wurde kein Speicher bereitgestellt** angezeigt wird, wählen Sie das in diesem Lab verwendete Abonnement aus, und klicken Sie dann auf **Speicher erstellen**.

4. Führen Sie in der **Bash-Eingabeaufforderung** im **Cloud Shell-Bereich** den folgenden Befehl aus, um eine Ressourcengruppe zu erstellen (ersetzen Sie den `<region>` Platzhalter durch den Namen der Azure-Region, die Ihnen am nächsten kommt, z. B. "eastus").

    ```bash
    RESOURCEGROUPNAME='az400m09l16-RG'
    LOCATION='<region>'
    az group create --name $RESOURCEGROUPNAME --location $LOCATION
    ```

5. Führen Sie den folgenden Befehl  aus, um einen App Service-Plan zu erstellen.

    ```bash
    SERVICEPLANNAME='az400l16-sp'
    az appservice plan create --resource-group $RESOURCEGROUPNAME \
        --name $SERVICEPLANNAME --sku B3 
    ```

6. Erstellen Sie in  eine Web-App mit einem eindeutigen Namen.

    ```bash
    WEBAPPNAME=partsunlimited$RANDOM$RANDOM
    az webapp create --resource-group $RESOURCEGROUPNAME --plan $SERVICEPLANNAME --name $WEBAPPNAME 
    ```

    > **Hinweis**: Notieren Sie den Namen der ACR-Instanz. Sie benötigen ihn später in diesem Lab.

### Übung 1: Konfigurieren Sie CI/CD-Pipelines als Code mit YAML in Azure DevOps.

Übung 1: Konfigurieren Sie CI/CD-Pipelines als Code mit YAML in Azure DevOps.

#### Aufgabe 1: Hinzufügen eines YAML-Builds und Bereitstellen einer Definition

In diesem Vorgang fügen Sie dem vorhandenen Projekt eine YAML-Builddefinition hinzu.

1. Navigieren Sie zurück zum **Bereich "Pipelines** " im **Pipelinehub** .
2. Klicken Sie im **Fenster "Erste Pipeline** erstellen" auf **"Pipeline erstellen"**.

    > **Hinweis**: Wir verwenden den Assistenten, um eine neue YAML-Pipelinedefinition basierend auf unserem Projekt zu erstellen.

3. Klicken Sie im **Bereich "Wo befindet sich Ihr Code?** " auf die **Option "Azure Repos Git (YAML)** ".
4. Klicken Sie im **Bereich "Repository** auswählen" auf **"eShopOnWeb**".
5. Wählen Sie im Bildschirm **Pipeline konfigurieren** die Option **Starterpipeline** aus.
6. **Wählen Sie alle Zeilen aus der Starterpipeline aus, und löschen Sie** sie.
7. **Kopieren Sie** die vollständige Vorlagenpipeline von unten, da Sie wissen, dass Sie Parameteränderungen **vornehmen müssen, bevor Sie die Änderungen speichern** :

```
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
4. Setzen Sie den Cursor auf eine neue Zeile am Ende der YAML-Definition (Zeile 69).

    > **Hinweis**: Dies ist der Ort, an dem neue Aufgaben hinzugefügt werden.

5. Suchen Sie in der Liste der Aufgaben auf der rechten Seite des Codebereichs nach der **aufgabe "Azure-App Dienstbereitstellung"**, und wählen Sie sie aus.
6. Geben Sie im **Bereich Azure-App Dienstbereitstellung** die folgenden Einstellungen an, und klicken Sie auf **"Hinzufügen**":

    - wählen Sie in der **Dropdownliste für Azure-Abonnements** das Azure-Abonnement aus, in dem Sie die Azure-Ressourcen weiter oben in der Übung bereitgestellt haben, klicken Sie auf **Autorisieren** und authentifizieren Sie sich bei Aufforderung mit demselben Benutzerkonto, das Sie während der Azure-Ressourcenbereitstellung verwendet haben.
    - wählen Sie in der **Dropdownliste "App Service-Name** " den Namen der Web-App aus, die Sie zuvor in der Übung bereitgestellt haben.
    - aktualisieren Sie im **Textfeld **"Paket" oder "Ordner**" den Standardwert auf `$(Build.ArtifactStagingDirectory)/**/Web.zip`.**
7. Bestätigen Sie die Einstellungen im Bereich "Assistent", indem Sie auf die **Schaltfläche "Hinzufügen** " klicken.

    > **Hinweis**: Dadurch wird der YAML-Pipelinedefinition automatisch die Bereitstellungsaufgabe hinzugefügt.

8. Der Codeausschnitt, der dem Editor hinzugefügt wurde, sollte ähnlich wie unten aussehen und ihren Namen für die Parameter azureSubscription und WebappName widerspiegeln:

> **Hinweis**: Der **parameter packageForLinux** ist im Kontext dieses Labors irreführend, ist aber für Windows oder Linux gültig.

    ```yaml
        - task: AzureRmWebAppDeployment@4
          inputs:
            ConnectionType: 'AzureRM'
            azureSubscription: 'AZURE SUBSCRIPTION HERE (b999999abc-1234-987a-a1e0-27fb2ea7f9f4)'
            appType: 'webApp'
            WebAppName: 'eshoponWebYAML369825031'
            packageForLinux: '$(Build.ArtifactStagingDirectory)/**/Web.zip'
    ```
9. Klicken Sie **im **Bereich "Speichern**" auf "Speichern****", um **die Änderung direkt in die Master-Verzweigung zu übernehmen.

    > **Hinweis**: Da unser ursprüngliches CI-YAML nicht so konfiguriert wurde, dass automatisch ein neuer Build ausgelöst wird, müssen wir diese manuell initiieren.

10. Navigieren Sie in Azure DevOps zur Registerkarte **Pipelines**, und wählen Sie **Pipelines** aus.
11. Öffnen Sie die **EShopOnWeb_MultiStageYAML** Pipeline, und klicken Sie auf **"Pipeline ausführen"**.
12. Bestätigen Sie die **Ausführung** aus dem angezeigten Bereich.
13. Beachten Sie die zwei verschiedenen Phasen, **Build .Net Core Solution** and **Deploy to Azure Web App** wird angezeigt.
14. Warten Sie, bis die Pipeline gestartet wird, und warten Sie, bis sie die Buildphase erfolgreich abgeschlossen hat.
15. Sobald die Bereitstellungsphase gestartet werden soll, werden Sie mit **den erforderlichen** Berechtigungen sowie einer orangefarbenen Leiste gefragt:

    ```text
    This pipeline needs permission to access a resource before this run can continue to Deploy to an Azure Web App
    ```

16. Klicken Sie auf **JSON-Ansicht**.
17. Klicken Sie im **Bereich "Auf Überprüfung** warten" auf **"Zulassen"**.
18. Überprüfen Sie die Nachricht im **Popupfenster** "Genehmigung", und bestätigen Sie, indem Sie auf "Zulassen"** klicken**.
19. Dadurch wird die Bereitstellungsphase deaktiviert. Warten Sie, bis die Bereitstellung erfolgreich abgeschlossen wurden.

#### Aufgabe 2: Überprüfen der bereitgestellten Website

1. Wechseln Sie zurück zum Webbrowserfenster, in dem die Azure-Portal angezeigt wird, und navigieren Sie zum Blatt, in dem die Eigenschaften der Azure Web App angezeigt werden.
2. Klicken Sie auf dem Blatt "Azure Web App" auf **"Übersicht** ", und klicken Sie auf dem Blatt "Übersicht" auf " **Durchsuchen** ", um Ihre Website auf einer neuen Webbrowserregisterkarte zu öffnen.
3. Stellen Sie sicher, dass die bereitgestellte Website auf der neuen Browserregisterkarte mit der EShopOnWeb E-Commerce-Website wie erwartet geladen wird.

### Übung 2: Bereitstellen und Einrichten von Azure Load Testing

In dieser Übung stellen Sie eine Azure Load Testing-Ressource in Azure bereit und konfigurieren unterschiedliche Auslastungstests für Ihren live ausgeführten Azure-App-Dienst.

#### Aufgabe 1: Bereitstellen von Azure Load Testing

Die Aufgabe  stellt eine neue Azure Load Testing-Ressource in Ihrem Azure-Abonnement bereit.

1. Navigieren Sie zu https://portal.azure.com)Erstellen einer Ressource im Azure-Portal.
2. Geben Sie **im Suchfeld "Suchdienste und Marketplace" azure Load Testing** ein.
3. Wählen Sie in den Suchergebnissen den (von Microsoft veröffentlichten) Load Balancer aus.
4. Klicken Sie auf der Seite "Azure Load Testing" auf " **Erstellen** ", um den Bereitstellungsprozess zu starten.
5. Geben Sie auf der Seite "Load Testing Resource" die erforderlichen Details für die Ressourcenbereitstellung an:
- **Abonnement**: Wählen Sie Ihr Azure-Abonnement aus.
- **Ressourcengruppe**: Wählen Sie die Ressourcengruppe aus, die Sie für die Bereitstellung des Web App-Diensts in der vorherigen Übung verwendet haben.
- **Name**: EShopOnWebLoadTesting
- Wählen Sie unter Region eine Region in Ihrer Nähe aus.

    > **Hinweis**: Der Azure Load Testing-Dienst ist in allen Azure-Regionen nicht verfügbar.

6. Überprüfen Sie Ihre Einstellungen, und klicken Sie auf **Erstellen**.
7. Klicken Sie auf " **Erstellen** ", um dies zu bestätigen, und rufen Sie die Azure Load Testing-Ressource auf.
8. Sie werden zur Seite "Bereitstellung ist in Bearbeitung" gewechselt. Es dauert einige Minuten, bis die Bereitstellung erfolgreich abgeschlossen ist.
9. Klicken Sie **auf der Seite "Bereitstellungsfortschritt" auf "Zur Ressource** wechseln", um zur Ressource "EShopOnWebLoadTesting** Azure Load Testing" zu **navigieren. 

    > **Hinweis**: Wenn Sie das Blatt geschlossen oder das Azure-Portal während der Bereitstellung der Azure Load Testing Resource geschlossen haben, können Sie es erneut aus dem Azure Portal Search-Feld oder aus der Liste "Ressourcen/Zuletzt verwendete Ressourcen" finden. 

#### Aufgabe 2: Erstellen von Azure Load Testing-Tests

In dieser Aufgabe erstellen Sie verschiedene Azure Load Testing-Tests mit unterschiedlichen Einstellungen für die Auslastungskonfiguration. 

10. Beachten Sie im **Blatt "EShopOnWebLoadTesting** Azure Load Testing Resource" das **Blatt "Erste Schritte mit einem Schnelltest**", und klicken Sie auf die **Schaltfläche "Schnelltest** ".
11. Führen Sie die folgenden Parameter und Einstellungen aus, um einen Auslastungstest zu erstellen:
- **Test-URL**: Geben Sie die URL aus dem Azure-App Dienst ein, den Sie in der vorherigen Übung bereitgestellt haben (EShopOnWeb... azurewebsites.net), **einschließlich https://**
- **Load** angeben: Virtuelle Benutzer
- **Anzahl der virtuellen Benutzer**
- Testdauer in Sekunden
- **Ramp-up-Zeit (Sekunden)**: 0
12. Bestätigen Sie die Konfiguration des Tests, indem Sie auf "Test** ausführen" klicken**.
13. Der Test wird alle 30 Minuten ausgeführt. 
14. Navigieren Sie mit der Ausführung des Tests zurück zur **Seite "EShopOnWebLoadTesting** Azure Load Testing Resource", und navigieren Sie zu **"Tests", wählen Sie "Tests****" aus, und sehen Sie **einen Test **Get_eshoponweb...**
15. Klicken Sie im oberen Menü auf "Erstellen **", **"** Schnelltest** erstellen", um einen zweiten Ladetest zu erstellen.
16. Führen Sie die folgenden Parameter und Einstellungen aus, um einen weiteren Auslastungstest zu erstellen:
- **Test-URL**: Geben Sie die URL aus dem Azure-App Dienst ein, den Sie in der vorherigen Übung bereitgestellt haben (EShopOnWeb... azurewebsites.net), **einschließlich https://**
- **Last** angeben: Anforderungen pro Sekunde (RPS)
- **Anforderungen pro Sekunde (RPS), SLL**
- **Antwortzeit (Millisekunden)**: 500
- Testdauer in Sekunden
- **Ramp-up-Zeit (Sekunden)**: 0
17. Bestätigen Sie die Konfiguration des Tests, indem Sie auf "Test** ausführen" klicken**.
18. Der Test wird alle 30 Minuten ausgeführt.

#### Aufgabe 3: Überprüfen der Ergebnisse von Azure Load Testing

In dieser Aufgabe überprüfen Sie das Ergebnis eines Azure Load Testing TestRun. 

Wenn beide Schnelltests abgeschlossen sind, nehmen wir einige Änderungen daran vor und überprüfen die Ergebnisse.

19. Navigieren Sie auf dem **Blatt "EShopOnWebLoadTesting** Resource" zu **"Tests**", und wählen Sie die erste Get_eshoponwebyaml aus... Test. Klicken Sie im Menü am oberen Rand auf **Beenden**.
20. Hier können Sie den **Testnamen** aus dem generierten Standardnamen in einen aussagekräftigeren ändern. Außerdem können Sie weiterhin Änderungen an den zuvor definierten Parametern vornehmen.
21. Navigieren Sie auf dem **Blatt "Test** bearbeiten" zur **Registerkarte "Testplan** ". 
22. Hier können Sie die **Apache JMeter** Load Testing Skriptdatei verwalten, was Azure Load Testing als Framework verwendet. Beachten Sie die Datei **quick_test.jmx**. Wählen Sie diese Datei aus, um sie auf dem virtuellen Laborcomputer zu **öffnen** . Wählen Sie **im Popupfenster Visual Studio Code** als Editor aus, um die Datei zu öffnen.
23. Beachten Sie die XML-Sprachstruktur der Datei.

    > Hinweis: Weitere Informationen und informationen zur erweiterten Syntax von Apache JMeter finden Sie unter dem folgenden [Link "Azure Load Testing – Jmeter](https://learn.microsoft.com/en-us/azure/load-testing/how-to-create-and-run-load-test-with-jmeter-script) ".

24. Zurück in der Testansicht **, in der **beide Tests angezeigt werden, wählen Sie eine der Tests aus, um eine detailliertere Ansicht zu öffnen, indem **Sie auf einen der Tests klicken**. Dadurch gelangen Sie zur detaillierteren Testseite. Von hier aus können Sie die Details der tatsächlichen Ausführung überprüfen, indem Sie die **TestRun_mm/dd/yyy-hh:hh** aus der resultierenden Liste auswählen.
25. Identifizieren Sie auf der detaillierten **TestRun-Seite** das tatsächliche Ergebnis der Azure Load Testing-Simulation. Einige der Werte sind:
- WAF Total Requests
- Duration
- Antwortzeit (zeigt das Ergebnis in Sekunden an und spiegelt die 90. Quantilantwortzeit wider . Dies bedeutet, dass für 90 % der Anforderungen die Antwortzeit innerhalb der angegebenen Ergebnisse liegt)
- Durchsatz in Anforderungen pro Sekunde
26. Weiter unten werden mehrere dieser Werte mithilfe von Dashboarddiagrammlinien und Diagrammansichten dargestellt.
27. Nehmen Sie sich ein paar Minuten Zeit, um die Ergebnisse** beider simulierter Tests miteinander zu vergleichen und **die Auswirkungen** weiterer Benutzer auf die App Service-Leistung zu **identifizieren.

### Übung 2: Automatisieren eines Auslastungstests mit CI/CD in Azure DevOps-Pipelines

Sie beginnen mit der Automatisierung von Auslastungstests in Azure Load Testing, indem Sie sie einer CI/CD-Pipeline hinzufügen. Nachdem Sie einen Auslastungstest im Azure-Portal ausgeführt haben, exportieren Sie die Konfigurationsdateien und konfigurieren eine CI/CD-Pipeline in GitHub Actions oder Azure Pipelines.

Nach Abschluss dieser Schnellstartanleitung verfügen Sie über einen CI/CD-Workflow, der für die Ausführung eines Auslastungstests mit Azure Load Testing konfiguriert ist.

#### Aufgabe 1: Identifizieren von ADO-Dienst-Verbinden iondetails

In dieser Aufgabe erteilen Sie den erforderlichen Berechtigungen für den Dienstprinzipal des Azure DevOps-Diensts Verbinden ion.

1. Navigieren Sie im **Azure-Portal** zum https://dev.azure.com)Azure DevOps-Projekt.
2. Wählen Sie links unten **Projekteinstellungen** aus.
3. Wählen Sie im Abschnitt „Pipelines“ die Option „Dienstverbindungen“ aus.
4. Beachten Sie die Dienst-Verbinden ion mit dem Namen Ihres Azure-Abonnements, das Sie zum Bereitstellen von Azure-Ressourcen am Anfang der Übung verwendet haben.
5. Wählen Sie Dienstverbindungen aus. Navigieren Sie auf der **Registerkarte "Übersicht**" zu **"Details**", und wählen Sie "Dienstprinzipal** verwalten" aus**.
6. Dadurch werden Sie zum Azure-Portal weitergeleitet, von wo aus die **Dienstprinzipaldetails** für das Identitätsobjekt geöffnet werden.
7. Kopieren Sie den **Anzeigenamenwert** (formatiert wie Name_of_ADO_Organization_EShopOnWeb_-b86d9ae1-7552-4b75-a1e0-27fb2ea7f9f4), abgesehen davon, wie Sie dies in den nächsten Schritten benötigen.

#### Gewähren von Berechtigungen für Ihren Dienstprinzipal

Azure Load Testing verwendet Azure RBAC, um Berechtigungen für die Ausführung bestimmter Aktivitäten für Ihre Auslastungstestressource zu erteilen. Um einen Auslastungstest über Ihre CI/CD-Pipeline auszuführen, weisen Sie dem Dienstprinzipal die Rolle „Mitwirkender für Auslastungstest“ zu.

1. Navigieren Sie im Azure-Portal zu Ihrer Azure Load Testing-Ressource.
2. Klicken Sie auf Zugriffssteuerung (IAM)+ HinzufügenRollenzuweisung hinzufügen.
3. Wählen Sie auf der Registerkarte **Rolle** in der Liste der Auftragsfunktionsrollen die Option **Mitwirkender für Auslastungstest** aus.
4. Wählen Sie auf der Registerkarte Mitglieder die Option Mitglieder auswählen aus, und verwenden Sie dann den Anzeigenamen, den Sie zuvor kopiert haben, um den Dienstprinzipal zu suchen.
5. Wählen Sie den Dienstprinzipal und dann Auswählen aus.
6. Wählen Sie auf der Registerkarte **Überprüfen + zuweisen** die Option **Überprüfen + zuweisen** aus, um die Rollenzuweisung hinzuzufügen.

Sie können jetzt die Dienstverbindung in Ihrer Azure Pipelines-Workflowdefinition verwenden, um auf Ihre Azure-Auslastungstestressource zuzugreifen.

#### Aufgabe 3: Exportieren von Eingabedateien für Auslastungstests und Importieren in Azure DevOps-Quellcodeverwaltung

Zum Ausführen eines Auslastungstests mit Azure Load Testing in einem CI/CD-Workflow müssen Sie die Konfigurationseinstellungen für den Auslastungstest und alle Eingabedateien in Ihrem Quellcodeverwaltungsrepository hinzufügen. Wenn Sie über einen vorhandenen Auslastungstest verfügen, können Sie die Konfigurationseinstellungen und alle Eingabedateien aus dem Azure-Portal herunterladen.

Führen Sie die folgenden Schritte aus, um die Eingabedateien für einen vorhandenen Auslastungstest im Azure-Portal herunterzuladen:

1. Navigieren Sie im Azure-Portal zu Ihrer Azure Load Testing-Ressource.
2. Wählen Sie im linken Bereich Tests aus, um die Liste der Auslastungstests anzuzeigen, und wählen Sie dann Ihren Test aus.
3. Wählen Sie die Auslassungspunkte (**...**) neben dem Testlauf aus, mit dem Sie arbeiten, und wählen Sie dann **Eingabedatei herunterladen** aus.
4. Der Browser lädt einen gezippten Ordner herunter, der die Eingabedateien für den Auslastungstest enthält.
5. Verwenden Sie ein beliebiges ZIP-Tool, um die Eingabedateien zu extrahieren. Der Ordner enthält die folgenden Dateien:

- *config.yaml*: die YaML-Konfigurationsdatei für den Auslastungstest. Sie verweisen auf diese Datei in der CI/CD-Workflowdefinition.
- *quick_test.jmx*: das JMeter-Testskript

6. Committen Sie alle extrahierten Eingabedateien in Ihr Quellcodeverwaltungsrepository. Navigieren Sie dazu zum **Azure DevOps-Portal**(https://dev.azure.com), und navigieren Sie zum **EShopOnWeb** DevOps-Projekt. 
7. Wählen Sie **Repositorys** aus. Beachten Sie in der Struktur des Quellcodeordners den Unterordner " **Tests** ". Beachten Sie die Auslassungspunkte (...), und wählen Sie **"Neuer > Ordner"** aus.
8. geben Sie **jmeter** als Ordnernamen und **platzhalter.txt** für den Dateinamen an (Hinweis: Ein Ordner kann nicht als leer erstellt werden)
9. Klicken Sie auf **Commit** , um die Erstellung der Platzhalterdatei und des jmeter-Ordners zu bestätigen.
10. Navigieren Sie aus der **Ordnerstruktur** zum neuen erstellten **jmeter-Unterordner** . Klicken Sie auf die **Auslassungspunkte(...)** und wählen Sie **"Datei hochladen" aus**.
11. Navigieren Sie mit der **Option "Durchsuchen**" zum Speicherort der extrahierten ZIP-Datei, und wählen Sie sowohl config.yaml** als **auch **quick_test.jmx** aus.
12. Klicken Sie auf **Commit** , um den Dateiupload in die Quellcodeverwaltung zu bestätigen.

#### Aufgabe 4: Aktualisieren der YAML-Definitionsdatei des CI/CD-Workflows

In dieser Aufgabe importieren Sie die Azure Load Testing - Azure DevOps Marketplace-Erweiterung sowie die vorhandene CI/CD-Pipeline mit der AzureLoadTest-Aufgabe.

1. Zum Erstellen und Ausführen eines Auslastungstests verwendet die Azure Pipelines-Workflowdefinition die Erweiterung **Azure Load Testing-Aufgabe** aus dem Azure DevOps-Marketplace. Öffnen Sie die [Azure Load Testing-Aufgabenerweiterung](https://marketplace.visualstudio.com/items?itemName=AzloadTest.AzloadTesting) im Azure DevOps-Marketplace und wählen Sie **Kostenlos abrufen** aus.
2. Wählen Sie Ihre Azure DevOps-Organisation aus und wählen Sie dann **Installieren** aus, um die Erweiterung zu installieren.
3. Navigieren Sie im Azure DevOps-Portal und -Projekt zu **Pipelines** , und wählen Sie die am Anfang dieser Übung erstellte Pipeline aus. Klicken Sie auf **Bearbeiten**.
4. Navigieren Sie im YAML-Skript zu **Zeile 56** , und drücken Sie die EINGABETASTE/EINGABETASTE, um eine neue leere Zeile hinzuzufügen. (dies befindet sich direkt vor der Bereitstellungsphase der YAML-Datei).
5. Wählen Sie in Zeile 57 den Aufgaben-Assistenten rechts aus, und suchen Sie nach **Azure Load Testing**.
6. Schließen Sie den grafischen Bereich mit den richtigen Einstellungen Ihres Szenarios ab:
- Azure-Abonnement (1): Wählen Sie das Azure-Abonnement Ihrer Ressourcengruppe aus.
- Load Test File: '$(Build.SourcesDirectory)/tests/jmeter/config.yaml' 
- Load Test Resource Group: Die Ressourcengruppe, die Ihre Azure Load Testing-Ressourcen enthält
- Load Test Resource Name: ESHopOnWebLoadTesting
- Name der Testausführung: ado_run
- Beschreibung der Auslastungsausführung: Laden von Tests aus ADO
7. Bestätigen Sie die Einfügung der Parameter als Codeausschnitt von YAML, indem Sie auf "Hinzufügen" klicken.****
8. Wenn beim Einzug des YAML-Codeausschnitts Fehler auftreten (rote Wellenlinien), beheben Sie diese, indem Sie zwei Leerzeichen oder Tabstopps hinzufügen, um den Codeausschnitt richtig zu positionieren.  
9. Der folgende Beispielausschnitt zeigt, wie der YAML-Code aussehen soll.
```
     - task: AzureLoadTest@1
      inputs:
        azureSubscription: 'AZURE DEMO SUBSCRIPTION(b86d9ae1-1234-4b75-a8e7-27fb2ea7f9f4)'
        loadTestConfigFile: '$(Build.SourcesDirectory)/tests/jmeter/config.yaml'
        resourceGroup: 'az400m05l11-RG'
        loadTestResource: 'EShopOnWebLoadTesting'
        loadTestRunName: 'ado_run'
        loadTestRunDescription: 'load testing from ADO'
```
10. fügen Sie unter dem eingefügten YAML-Codeausschnitt eine neue leere Zeile hinzu, indem Sie die EINGABETASTE/EINGABETASTE drücken. 
11. fügen Sie unter dieser leeren Zeile einen Codeausschnitt für die Aufgabe "Veröffentlichen" hinzu, der die Ergebnisse der Azure Load-Testaufgabe während der Pipelineausführung anzeigt:

```
    - publish: $(System.DefaultWorkingDirectory)/loadTest
      artifact: loadTestResults
```
12.  Wenn beim Einzug des YAML-Codeausschnitts Fehler auftreten (rote Wellenlinien), beheben Sie diese, indem Sie zwei Leerzeichen oder Tabstopps hinzufügen, um den Codeausschnitt richtig zu positionieren.  
13. Wenn beide Codeausschnitte zur CI/CD-Pipeline hinzugefügt wurden, **speichern Sie** die Änderungen. 
14. Klicken Sie nach dem Speichern auf **"Ausführen** ", um die Pipeline auszulösen.
15. Bestätigen Sie die Verzweigung (Standard), und klicken Sie auf die **Schaltfläche "Ausführen**", um die Pipelineausführung zu starten.
16. Klicken Sie auf der Seite "Pipelinestatus" auf die **Buildphase** , um die ausführlichen Protokollierungsdetails der verschiedenen Aufgaben in der Pipeline zu öffnen.
17. Warten Sie, bis die Pipeline die Buildphase startet, und gelangen Sie zum **AzureLoadTest-Vorgang** im Fluss der Pipeline. 
18. Navigieren Sie während der Ausführung der Aufgabe zum **Azure Load Testing** im Azure-Portal, und sehen Sie, wie die Pipeline einen neuen RunTest namens **adoloadtest1** erstellt. Sie können es auswählen, um die Ergebniswerte des TestRun-Auftrags anzuzeigen.
19. Navigieren Sie zurück zur Ansicht "Azure DevOps CI/CD Pipeline Run", in der die **AzureLoadTest-Aufgabe** erfolgreich abgeschlossen wurde. Aus der ausführlichen Protokollierungsausgabe werden auch die resultierenden Werte des Auslastungstests angezeigt:

```
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
20. Sie haben nun einen automatisierten Auslastungstest als Teil einer Pipelineausführung ausgeführt. In der letzten Aufgabe geben Sie Bedingungen für fehler an, was bedeutet, dass unsere Bereitstellungsphase nicht gestartet werden kann, wenn die Leistung der Web-App unter einem bestimmten Schwellenwert liegt. 

#### Aufgabe 5: Hinzufügen von Fehler-/Erfolgskriterien zur Lasttestpipeline

In dieser Aufgabe verwenden Sie Auslastungstestfehlerkriterien, um benachrichtigt zu werden (wenn eine fehlerhafte Pipeline als Ergebnis ausgeführt wird), wenn die Anwendung Ihre Qualitätsanforderungen nicht erfüllt.

1. Navigieren Sie in Azure DevOps zum EShopOnWeb-Projekt, und öffnen Sie **"Repos"**.
2. Navigieren Sie in Repos zu dem **zuvor erstellten /tests/jmeter-Unterordner** .
3. : Die YAML-Konfigurationsdatei für den Auslastungstest. Klicken Sie auf **"Bearbeiten** ", um die Bearbeitung der Datei zuzulassen.
4. Fügen Sie am Ende der Datei den folgenden Code hinzu:

```
failureCriteria:
  - avg(response_time_ms) > 300
  - percentage(error) > 50
```
5. Speichern Sie die Änderungen an config.yaml, indem Sie erneut auf Commit** und Commit klicken**.
6. Navigieren Sie zurück zu **Pipelines** , und führen Sie die **EShopOnWeb-Pipeline** erneut aus. Nach ein paar Minuten wird die Ausführung mit einem **Fehlerstatus** für die **AzureLoadTest-Aufgabe** abgeschlossen. 
7. Öffnen Sie die ausführliche Protokollierungsansicht für die Pipeline, und überprüfen Sie die Details von **AzureLoadtest**. Nachfolgend sehen Sie eine Beispielausgabe:

```
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

8. Beachten Sie, wie die letzte Zeile der Ausgabe des Auslastungstests ##[fehler]TestResult: FAILED** lautet**. Da wir eine **FailCriteria** mit einer durchschnittliche Antwortzeit von > 300 definiert haben oder einen Fehlerprozentsatz von > 20 aufweisen, wird nun eine durchschnittliche Antwortzeit angezeigt, die mehr als 300 ist, wird der Vorgang als fehlgeschlagen gekennzeichnet. 

    > Hinweis: Stellen Sie sich in einem realen Szenario vor, dass Sie die Leistung Ihres App-Diensts überprüfen würden, und wenn die Leistung unter einem bestimmten Haltepunkt liegt . Dies bedeutet, dass es in der Regel mehr Last für die Web App gibt, könnten Sie eine neue Bereitstellung für einen zusätzlichen Azure-App Dienst auslösen. Da wir die Reaktionszeit für Azure Lab-Umgebungen nicht steuern können, haben wir beschlossen, die Logik zu rückgängig machen, um den Fehler zu gewährleisten.

9.  Der FAILED-Status des Pipelinevorgangs spiegelt tatsächlich einen ERFOLG der Überprüfung der Anforderungskriterien für Azure Load Testing wider.

### Übung 3: Entfernen der Azure-Lab-Ressourcen.

In dieser Übung entfernen Sie die in dieser Übung bereitgestellten Azure-Ressourcen, um unerwartete Gebühren zu beseitigen.

>**Hinweis**: Denken Sie daran, alle neu erstellten Azure-Ressourcen zu entfernen, die Sie nicht mehr verwenden. Durch das Entfernen nicht verwendeter Ressourcen wird sichergestellt, dass keine unerwarteten Kosten anfallen.

#### Übung 4: Entfernen der Azure Lab-Ressourcen

In dieser Aufgabe verwenden Sie Azure Cloud Shell, um die in dieser Übung bereitgestellten Azure-Ressourcen zu entfernen, um unnötige Gebühren zu vermeiden.

1. Öffnen Sie im Azure-Portal im **Cloud Shell**-Bereich die **Bash**-Sitzung.
2. Listen Sie alle Ressourcengruppen auf, die während der Labs in diesem Modul erstellt wurden, indem Sie den folgenden Befehl ausführen:

    ```sh
    az group list --query "[?starts_with(name,'az400m16l01')].name" --output tsv
    ```

3. Löschen Sie alle Ressourcengruppen, die Sie während der praktischen Übungen in diesem Modul erstellt haben, indem Sie den folgenden Befehl ausführen:

    ```sh
    az group list --query "[?starts_with(name,'az400m16l01')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**Hinweis**: Der Befehl wird (dem --nowait-Parameter entsprechend) asynchron ausgeführt. Dies bedeutet, dass Sie zwar einen weiteren Azure CLI-Befehl in derselben Bash-Sitzung direkt im Anschluss ausführen können, es jedoch einige Minuten dauert, bis die Ressourcengruppen tatsächlich entfernt werden.

## Überprüfung

In dieser Übung haben Sie eine Web-App für Azure-App Dienst mithilfe von Azure-Pipelines bereitgestellt sowie eine Azure Load Testing-Ressource mit TestRuns bereitgestellt. Als Nächstes haben Sie die Jmeter Load Testing config.yaml-Datei in die Quellcodeverwaltung von Azure DevOps Repos integriert und Ihre CI/CD-Pipeline mit dem Azure Load Testing erweitert. In der letzten Übung haben Sie gelernt, wie Sie die Erfolgskriterien von LoadTest definieren.
