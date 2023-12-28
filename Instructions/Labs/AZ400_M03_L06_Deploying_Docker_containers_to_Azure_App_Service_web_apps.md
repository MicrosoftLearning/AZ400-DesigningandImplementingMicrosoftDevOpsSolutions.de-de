---
lab:
  title: 'Lab: Bereitstellen von Docker-Containern für Azure App Service-Web-Apps'
  module: 'Module 03: Implement CI with Azure Pipelines and GitHub Actions'
---

# Lab: Bereitstellen von Docker-Containern für Azure App Service-Web-Apps

## Lab-Handbuch für Kursteilnehmer

## Labanforderungen

- Für dieses Lab ist **Microsoft Edge** oder ein [von Azure DevOps unterstützter Browser](https://learn.microsoft.com/azure/devops/server/compatibility) erforderlich.

- **Einrichten einer Azure DevOps-Organisation**: Wenn Sie nicht bereits eine Azure DevOps-Organisation haben, die Sie für dieses Lab verwenden können, müssen Sie diese erstellen, indem Sie die unter [Erstellen einer Organisation oder Projektsammlung](https://learn.microsoft.com/azure/devops/organizations/accounts/create-organization) beschriebenen Anweisungen befolgen.

- Identifizieren Sie ein vorhandenes Azure-Abonnement, oder erstellen Sie ein neues Abonnement.

- Vergewissern Sie sich, dass Sie über ein Microsoft- oder ein Microsoft Entra-Konto mit der Rolle „Mitwirkender“ oder „Besitzer“ im Azure-Abonnement verfügen. Ausführliche Informationen finden Sie in den Artikeln zum [Auflisten von Azure-Rollenzuweisungen mithilfe des Azure-Portals](https://learn.microsoft.com/azure/role-based-access-control/role-assignments-list-portal) und [Anzeigen und Zuweisen von Administratorrollen in Azure Active Directory](https://learn.microsoft.com/azure/active-directory/roles/manage-roles-portal).

## Übersicht über das Labor

In diesem Lab erfahren Sie, wie Sie mithilfe einer Azure DevOps CI/CD-Pipeline ein benutzerdefiniertes Docker-Image erstellen, es an Azure Container Registry übergeben und es als Container für Azure App Service bereitstellen.

## Ziele

In diesem Lab lernen Sie Folgendes:

- Erstellen Sie ein benutzerdefiniertes Docker-Image mithilfe eines von Microsoft gehosteten Linux-Agents.
- Pushen Sie ein Image zu Azure Container Registry.
- Stellen Sie mithilfe von Azure DevOps ein Docker-Image als Container für Azure App Service bereit.

## Geschätzte Zeit: 30 Minuten

## Anweisungen

### Übung 0: Konfigurieren der Voraussetzungen für das Lab.

In dieser Übung richten Sie die Voraussetzungen für das Labor ein, das aus einem neuen Azure DevOps-Projekt mit einem Repository basierend auf dem [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb) besteht.

#### Aufgabe 1: (überspringen, wenn fertig) Erstellen und Konfigurieren des Teamprojekts

In dieser Aufgabe erstellen Sie ein **eShopOnWeb** Azure DevOps-Projekt, das von mehreren Laboren verwendet werden soll.

1. Öffnen Sie auf Ihrem Laborcomputer in einem Browserfenster Ihre Azure DevOps-Organisation. Klicken auf „Neues Projekt“ Geben Sie Ihrem Projekt den Namen **"eShopOnWeb** ", und wählen Sie **"Scrum** " in der **Dropdownliste "Arbeitsaufgabe"** aus. Klicken Sie auf **Erstellen**.

#### Aufgabe 2: (überspringen, wenn erledigt) Importieren von eShopOnWeb Git Repository

Bei dieser Aufgabe importieren Sie das eShopOnWeb Git-Repository, das von mehreren Labs verwendet wird.

1. Öffnen Sie auf Ihrem Laborcomputer in einem Browserfenster Ihre Azure DevOps-Organisation und das zuvor erstellte **eShopOnWeb-Projekt** . Klicken Sie auf **Repos>Files** , **Import**. Fügen Sie im **Fenster "Git Repository** importieren" die folgende URL https://github.com/MicrosoftLearning/eShopOnWeb.git  ein, und klicken Sie auf " **Importieren**":

2. Das Repository ist wie folgt organisiert:
    - Der Ordner „.ado“ enthält Azure DevOps-YAML-Pipelines.
    - Der Ordner „.devcontainer“ enthält ein Containersetup für die Entwicklung mithilfe von Containern (entweder lokal in VS Code oder über GitHub Codespaces).
    - **Azure-Ordner** enthält Bicep&ARM-Infrastruktur als Codevorlagen, die in einigen Lab-Szenarien verwendet werden.
    - **GITHUB-Ordnercontainer-YAML-GitHub-Workflowdefinitionen** .
    - Der Ordner „src“ enthält die .NET 6-Website, die in den Labszenarien verwendet wird.

#### Aufgabe 3: (Wenn erledigt) Legen Sie Standard Verzweigung als Standardbranch fest.

1. Wechseln Sie zu **Repos>Branches**.
2. Zeigen Sie auf die **Standard** Verzweigung, und klicken Sie dann rechts neben der Spalte auf die Auslassungspunkte.
3. Klicken Sie auf **"Als Standardbranch** festlegen".

### Übung 1: Verwalten der Dienstverbindung

In dieser Übung konfigurieren Sie die Dienstverbindung mit Ihrem Azure-Abonnement und führen dann die CI-Pipeline aus.

#### Aufgabe 1: (überspringen, falls erledigt) Verwalten der Dienstverbindung

Sie können eine Verbindung zwischen Azure Pipelines und externen Diensten und Remotediensten herstellen, um Aufgaben in einem Auftrag auszuführen.

In dieser Aufgabe erstellen Sie einen Dienstprinzipal mithilfe der Azure CLI, der Azure DevOps zulässt:

- Bereitstellen von Ressourcen in einem Azure-Abonnement
- Übertragen des Docker-Images an Azure Container Registry per Pushvorgang.
- Fügen Sie eine Rollenzuweisung hinzu, um Azure-App Dienst das Docker-Image aus der Azure-Containerregistrierung abzurufen.

> **Hinweis**: Wenn Sie bereits über einen Dienstprinzipal verfügen, können Sie direkt mit der nächsten Aufgabe fortfahren.

Sie benötigen einen Dienstprinzipal, um Azure-Ressourcen aus Azure-Pipelines bereitzustellen.

Ein Dienstprinzipal wird automatisch von Azure Pipeline erstellt, wenn Sie eine Verbindung mit einem Azure-Abonnement innerhalb einer Pipelinedefinition herstellen oder wenn Sie eine neue Dienstverbindung über die Seite mit den Projekteinstellungen (automatische Option) erstellen. Sie können den Dienstprinzipal auch manuell über das Portal erstellen oder Azure CLI verwenden und es für alle Projekte wiederverwenden.

1. Starten Sie auf Ihrem Labcomputer einen Webbrowser, navigieren Sie zum [**Azure-Portal**](https://portal.azure.com), und melden Sie sich an. Verwenden Sie hierzu die Anmeldeinformationen eines Benutzerkontos, das in dem Abonnement, das Sie in diesem Lab verwenden, über die Rolle „Besitzer“ und in dem Azure AD-Mandanten, der dem Azure-Abonnement zugeordnet ist, über die Rolle „Globaler Administrator“ verfügt.
2. Öffnen Sie im Azure-Portal den Bereich **Cloud Shell**, indem Sie auf das Symbolleistensymbol rechts neben dem Textfeld für die Suche klicken.
3. Wählen Sie bei Aufforderung zur Auswahl von **Bash** oder **PowerShell** die Option **Bash** aus.

   >**Hinweis**: Wenn Sie **Cloud Shell** zum ersten Mal starten und die Meldung **Für Sie wurde kein Speicher bereitgestellt** angezeigt wird, wählen Sie das in diesem Lab verwendete Abonnement aus, und klicken Sie dann auf **Speicher erstellen**.

4. Führen Sie in der **Bash-Eingabeaufforderung** im **Cloud Shell-Bereich** die folgenden Befehle aus, um die Werte des Azure-Abonnement-ID-Attributs abzurufen:

    ```bash
    subscriptionName=$(az account show --query name --output tsv)
    subscriptionId=$(az account show --query id --output tsv)
    echo $subscriptionName
    echo $subscriptionId
    ```

    > **Hinweis**: Kopieren Sie beide Werte in eine Textdatei. Sie werden sie später in diesem Lab benötigen.

5. Führen Sie in der **Bash-Eingabeaufforderung** im **Cloud Shell-Bereich** den folgenden Befehl aus, um einen Dienstprinzipal zu erstellen:

    ```bash
    az ad sp create-for-rbac --name sp-az400-azdo --role contributor --scopes /subscriptions/$subscriptionId
    ```

    > **Hinweis**: Der Befehl generiert eine JSON-Ausgabe. Speichern der Ausgabe in einer Textdatei Sie benötigen ihn später in diesem Lab.

6. Starten Sie als Nächstes auf dem Laborcomputer einen Webbrowser, navigieren Sie zum Azure DevOps **eShopOnWeb-Projekt** . Klicken Sie auf **Project Einstellungen>Service-Verbinden ionen (unter Pipelines)** und **Verbinden ion** neuer Dienst.
7. Wählen Sie im Bildschirm **Neue Dienstverbindung** die Option **Azure Resource Manager** und anschließend **Weiter** aus.
8. Wählen Sie **den Dienstprinzipal (manuell)** aus, und klicken Sie auf **"Weiter"**.
9. Füllen Sie die leeren Felder mit den informationen aus, die während der vorherigen Schritte gesammelt wurden:
    - Abonnement-ID oder -Name.
    - Dienstprinzipal-ID (appId), Dienstprinzipalschlüssel (Kennwort) und Mandanten-ID (Mandant).
    - Geben **Sie den Namen des Dienstverbindungstyps** **"azure-connection**" ein. Auf diesen Namen wird in YAML-Pipelines verwiesen, wenn ein Azure DevOps Service Verbinden ion erforderlich ist, um mit Ihrem Azure-Abonnement zu kommunizieren.

10. Klicken Sie auf **Überprüfen und speichern**.

### Übung 2: Importieren und Ausführen der CI-Pipeline

Übung 2: Importieren und Ausführen der CI-Pipeline

#### Übung 2: Importieren und Ausführen der CI-Pipeline

1. Navigieren Sie in Azure Pipelines zu PipelinesPipelines.
2. Klicken Sie auf die Schaltfläche Neue Pipeline**.
3. Wählen Sie **Azure Repos Git** (YAML) aus.
4. Auswählen des **eShopOnWeb-Repositorys**
5. Wählen Sie die Option **Vorhandene Azure Pipelines-YAML-Datei** aus.
6. Wählen Sie die **Datei "/.ado/eshoponweb-ci-docker.yml** " aus, und klicken Sie dann auf " **Weiter".**
7. Passen Sie in der YAML-Pipelinedefinition Folgendes an:
   - Ersetzen Sie <yourSubscriptionId> durch Ihre Azure-Abonnement-ID.
   - **rg-az400-container-NAME** mit dem Ressourcengruppennamen, der von der Pipeline erstellt wird (es kann auch eine vorhandene Ressourcengruppe sein).

8. Klicken Sie auf " **Speichern und Ausführen** ", und warten Sie, bis die Pipeline erfolgreich ausgeführt wird.

    > Die Bereitstellung kann einige Minuten dauern.

    Die Übung umfasst die folgenden Aufgaben:
    - **Ressourcen**: Sie lädt die Repositorydateien herunter, die in den folgenden Aufgaben verwendet werden.
    - **AzureResourceManagerTemplateDeployment**: Stellt die Azure-Containerregistrierung mithilfe der Bicep-Vorlage bereit.
    - **PowerShell**: Abrufen des **ACR-Anmeldeserverwerts** aus der Ausgabe der vorherigen Aufgabe und Erstellen eines neuen Parameters **acrLoginServer**
    - [**Docker**](https://learn.microsoft.com/azure/devops/pipelines/tasks/reference/docker-v0?view=azure-pipelines) **– Build**: Erstellen des Docker-Images und Erstellen von zwei Tags (Neueste und aktuelle BuildID)
    - Übertragen des Docker-Images an Azure Container Registry per Pushvorgang.

9. Ihre Pipeline nimmt einen Namen auf der Grundlage des Projektnamens an. **Benennen** wir sie um, um die Pipeline besser zu identifizieren. Wechseln Sie zu **Pipelines>Pipelines** , und klicken Sie auf die kürzlich erstellte Pipeline. Klicken Sie auf die Auslassungspunkte und **die Option "Umbenennen/Verschieben** ". Nennen Sie es **eshoponweb-ci-docker** , und klicken Sie auf **"Speichern"**.

10. Navigieren Sie zum [**Azure-Portal**](https://portal.azure.com), suchen Sie in der kürzlich erstellten Ressourcengruppe nach der Azure-Containerregistrierung (sie sollte rg-az400-container-NAME genannt werden****). Klicken Sie auf der linken Seite auf **Repositorys** unter **"Dienste** ", und stellen Sie sicher, dass das Repository **eshoponweb/web** erstellt wurde. Wenn Sie auf den Repositorylink klicken, sollten zwei Tags angezeigt werden (eine davon ist **neu**), dies sind die Pushbilder. Wenn dies nicht angezeigt wird, überprüfen Sie den Status Ihrer Pipeline.

### Übung 3: Importieren und Ausführen der CD-Pipeline

In dieser Übung konfigurieren Sie die Dienstverbindung mit Ihrem Azure-Abonnement und führen dann die CD-Pipeline aus.

#### Hinzufügen einer neuen Rollenzuweisung

In dieser Aufgabe fügen Sie eine neue Rollenzuweisung hinzu, um Azure-App Dienst das Docker-Image aus der Azure-Containerregistrierung abzurufen.

1. Navigieren Sie zum [**Azure-Portal**](https://portal.azure.com).
2. Öffnen Sie im Azure-Portal den Bereich **Cloud Shell**, indem Sie auf das Symbolleistensymbol rechts neben dem Textfeld für die Suche klicken.
3. Wählen Sie bei Aufforderung zur Auswahl von **Bash** oder **PowerShell** die Option **Bash** aus.
4. Führen Sie in der **Bash-Eingabeaufforderung** im **Cloud Shell-Bereich** die folgenden Befehle aus, um die Werte des Azure-Abonnement-ID-Attributs abzurufen:

    ```sh
    spId=$(az ad sp list --display-name sp-az400-azdo --query "[].id" --output tsv)
    echo $spId
    roleName=$(az role definition list --name "User Access Administrator" --query "[0].name" --output tsv)
    echo $roleName
    ```

5. Nach dem Abrufen der Dienstprinzipal-ID und des Rollennamens erstellen wir die Rollenzuweisung durch Ausführen dieses Befehls (ersetzen Sie **rg-az400-container-NAME** durch ihren Ressourcengruppennamen)

    ```sh
    az role assignment create --assignee $spId --role $roleName --resource-group "rg-az400-container-NAME"
    ```

Nun sollte die JSON-Ausgabe angezeigt werden, die den Erfolg der Ausführung des Befehls bestätigt.

#### Übung 3: Importieren und Ausführen der CD-Pipeline

In dieser Aufgabe importieren und ausführen Sie die CD-Pipeline.

1. Navigieren Sie in Azure Pipelines zu PipelinesPipelines.
2. Klicken Sie auf die Schaltfläche Neue Pipeline**.
3. Wählen Sie **Azure Repos Git** (YAML) aus.
4. Auswählen des **eShopOnWeb-Repositorys**
5. Wählen Sie die Option **Vorhandene Azure Pipelines-YAML-Datei** aus.
6. Wählen Sie die **Datei "/.ado/eshoponweb-cd-webapp-docker.yml** " aus, und klicken Sie dann auf **"Weiter".**
7. Passen Sie in der YAML-Pipelinedefinition Folgendes an:
   - Ersetzen Sie <yourSubscriptionId> durch Ihre Azure-Abonnement-ID.
   - **rg-az400-container-NAME** mit dem zuvor im Labor definierten Ressourcengruppennamen.

8. Klicken Sie auf " **Speichern und Ausführen** ", und warten Sie, bis die Pipeline erfolgreich ausgeführt wird.

    > Die Bereitstellung kann einige Minuten dauern.
    
    > **Wichtig**: Wenn Sie die Fehlermeldung "TF402455: Pushes to this branch are not zulässig; Sie müssen eine Pullanforderung verwenden, um diese Verzweigung zu aktualisieren.", müssen Sie die Branch-Schutzregel "Mindestens eine Anzahl von Prüfern erforderlich" deaktivieren, die in den vorherigen Laboren aktiviert ist.

    Die CD-Definition besteht aus den folgenden Aufgaben:
    - **Ressourcen**: Sie lädt die Repositorydateien herunter, die in den folgenden Aufgaben verwendet werden.
    - **AzureResourceManagerTemplateDeployment**: Stellt den Azure-App Dienst mithilfe der Bicep-Vorlage bereit.
    - **Hinzufügen von Azure-Rollenzuweisungen mithilfe von Azure Resource Manager-Vorlagen**

9. Ihre Pipeline nimmt einen Namen auf der Grundlage des Projektnamens an. **Benennen** wir sie um, um die Pipeline besser zu identifizieren. Wechseln Sie zu **Pipelines>Pipelines** , und zeigen Sie auf die kürzlich erstellte Pipeline. Klicken Sie auf die Auslassungspunkte und **die Option "Umbenennen/Verschieben** ". Nennen Sie es **eshoponweb-cd-webapp-docker**, und klicken Sie auf "Speichern"****.

    > **Hinweis 1**: Die Verwendung der **Vorlage "/.azure/bicep/webapp-docker.bicep** " erstellt einen App-Serviceplan, eine Web-App mit aktivierter verwalteter Identität und verweist auf das zuvor übertragene Docker-Image: **${acr.properties.loginServer}/eshoponweb/web:latest**.

    > **Hinweis 2**: Die Verwendung der **Vorlage "/.azure/bicep/webapp-to-acr-roleassignment.bicep** " erstellt eine neue Rollenzuweisung für die Web-App mit AcrPull-Rolle, um das Docker-Image abrufen zu können. Dies kann in der ersten Vorlage erfolgen, da die Rollenzuweisung jedoch einige Zeit in Anspruch nehmen kann, ist es ratsam, beide Aufgaben separat auszuführen.

#### Aufgabe 3: Testen der Lösung

1. Navigieren Sie im Azure-Portal zur zuletzt erstellten Ressourcengruppe, und nun sollten drei Ressourcen (App Service, App Service Plan und Containerregistrierung) angezeigt werden.

1. Navigieren Sie zum App-Dienst, und klicken Sie dann auf **"Durchsuchen**", damit gelangen Sie zur Website.

Herzlichen Glückwunsch! In dieser Übung haben Sie eine Website mit einem benutzerdefinierten Docker-Image bereitgestellt.

### Übung 4: Entfernen der Azure Lab-Ressourcen

In dieser Übung entfernen Sie die in dieser Übung bereitgestellten Azure-Ressourcen, um unerwartete Gebühren zu beseitigen.

>**Hinweis**: Denken Sie daran, alle neu erstellten Azure-Ressourcen zu entfernen, die Sie nicht mehr verwenden. Durch das Entfernen nicht verwendeter Ressourcen wird sichergestellt, dass keine unerwarteten Kosten anfallen.

#### Übung 4: Entfernen der Azure Lab-Ressourcen

In dieser Aufgabe verwenden Sie Azure Cloud Shell, um die in dieser Übung bereitgestellten Azure-Ressourcen zu entfernen, um unnötige Gebühren zu vermeiden.

1. Öffnen Sie im Azure-Portal im **Cloud Shell**-Bereich die **Bash**-Sitzung.
2. Listen Sie alle Ressourcengruppen auf, die während der Labs in diesem Modul erstellt wurden, indem Sie den folgenden Befehl ausführen:

    ```sh
    az group list --query "[?starts_with(name,'rg-az400-container-')].name" --output tsv
    ```

3. Löschen Sie alle Ressourcengruppen, die Sie während der praktischen Übungen in diesem Modul erstellt haben, indem Sie den folgenden Befehl ausführen:

    ```sh
    az group list --query "[?starts_with(name,'rg-az400-container-')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**Hinweis**: Der Befehl wird (dem --nowait-Parameter entsprechend) asynchron ausgeführt. Dies bedeutet, dass Sie zwar einen weiteren Azure CLI-Befehl in derselben Bash-Sitzung direkt im Anschluss ausführen können, es jedoch einige Minuten dauert, bis die Ressourcengruppen tatsächlich entfernt werden.

## Überprüfung

In diesem Lab erfahren Sie, wie Sie mithilfe einer Azure DevOps CI/CD-Pipeline ein benutzerdefiniertes Docker-Image erstellen, es an Azure Container Registry übergeben und es als Container für Azure App Service bereitstellen.
