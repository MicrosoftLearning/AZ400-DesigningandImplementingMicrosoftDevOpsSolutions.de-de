---
lab:
  title: Bereitstellen von Docker-Containern für Azure App Service-Web-Apps
  module: 'Module 02: Implement CI with Azure Pipelines and GitHub Actions'
---

# Bereitstellen von Docker-Containern für Azure App Service-Web-Apps

## Lab-Handbuch für Kursteilnehmer

## Labanforderungen

- Für dieses Lab ist **Microsoft Edge** oder ein von [Azure DevOps unterstützter Browser](https://learn.microsoft.com/azure/devops/server/compatibility) erforderlich.

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

### Übung 0: Konfigurieren der Voraussetzungen für das Lab

In dieser Übung richten Sie die Voraussetzungen für das Lab ein, das aus einem neuen Azure DevOps-Projekt mit einem Repository basierend auf dem [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb) besteht.

#### Aufgabe 1: (überspringen, wenn fertig) Erstellen und Konfigurieren des Teamprojekts

In dieser Aufgabe erstellen Sie ein **eShopOnWeb** Azure DevOps-Projekt, das von mehreren Labs verwendet werden soll.

1. Öffnen Sie auf Ihrem Lab-Computer in einem Browserfenster Ihre Azure DevOps-Organisation. Klicken Sie auf **Neues Projekt**. Geben Sie Ihrem Projekt den Namen **eShopOnWeb**, und wählen Sie **Scrum** in der Dropdownliste **Arbeitselementprozess** aus. Klicken Sie auf **Erstellen**.

#### Aufgabe 2: (überspringen, wenn erledigt) Importieren von eShopOnWeb Git Repository

Bei dieser Aufgabe importieren Sie das eShopOnWeb Git-Repository, das von mehreren Labs verwendet wird.

1. Öffnen Sie auf Ihrem Lab-Computer in einem Browserfenster Ihre Azure DevOps-Organisation und das zuvor erstellte **eShopOnWeb**-Projekt. Klicken Sie auf **Repos>Dateien**, **Importieren**. Fügen Sie im Fenster **Git Repository importieren** die folgende URL <https://github.com/MicrosoftLearning/eShopOnWeb.git> ein und klicken Sie auf **Importieren**:

1. Das Repository ist wie folgt organisiert:
    - Der Ordner **.ado** enthält Azure DevOps-YAML-Pipelines.
    - Der Ordner **.devcontainer** enthält ein Containersetup für die Entwicklung mithilfe von Containern (entweder lokal in VS Code oder über GitHub Codespaces).
    - Der Ordner **infra** enthält eine Bicep&ARM-Infrastruktur als Codevorlagen, die in einigen Labszenarien verwendet werden.
    - Der Ordner **.github** enthält YAML GitHub-Workflow-Definitionen.
    - Der Ordner **src** enthält die .NET 8-Website, die in den Labszenarios verwendet wird.

#### Aufgabe 3: (überspringen, wenn erledigt) Legen Sie den Mainbranch als Standardbranch fest

1. Wechseln Sie zu **Repos>Branches**.
1. Bewegen Sie den Mauszeiger auf den **Main**-Branch und klicken Sie dann rechts neben der Spalte auf die Auslassungspunkte.
1. Klicken Sie auf **Als Mainbranch festlegen**.

### Übung 1: Verwalten der Dienstverbindung

In dieser Übung konfigurieren Sie die Dienstverbindung mit Ihrem Azure-Abonnement, importieren die CI-Pipeline und führen sie dann aus.

#### Aufgabe 1: (überspringen, falls erledigt) Verwalten der Dienstverbindung

Sie können eine Verbindung zwischen Azure Pipelines und externen Diensten und Remotediensten herstellen, um Aufgaben in einem Auftrag auszuführen.

In dieser Aufgabe erstellen Sie einen Dienstprinzipal mithilfe der Azure CLI, was Azure DevOps Folgendes erlaubt:

- Bereitstellen von Ressourcen in einem Azure-Abonnement
- Übertragen des Docker-Images an Azure Container Registry per Pushvorgang.
- Fügen Sie eine Rollenzuweisung hinzu, um Azure App Service zu erlauben, das Docker-Image aus der Azure-Containerregistrierung abzurufen.

> **Hinweis**: Wenn Sie bereits über einen Dienstprinzipal verfügen, können Sie direkt mit der nächsten Aufgabe fortfahren.

Sie benötigen einen Dienstprinzipal, um Azure-Ressourcen aus Azure-Pipelines bereitzustellen.

Ein Dienstprinzipal wird automatisch von Azure Pipeline erstellt, wenn Sie eine Verbindung mit einem Azure-Abonnement innerhalb einer Pipelinedefinition herstellen oder wenn Sie eine neue Dienstverbindung über die Seite mit den Projekteinstellungen (automatische Option) erstellen. Sie können den Dienstprinzipal auch manuell über das Portal erstellen oder Azure CLI verwenden und es für alle Projekte wiederverwenden.

1. Starten Sie auf Ihrem Labcomputer einen Webbrowser, navigieren Sie zum [**Azure-Portal**](https://portal.azure.com), und melden Sie sich an. Verwenden Sie hierzu die Anmeldeinformationen eines Benutzerkontos, das in dem Abonnement, das Sie in diesem Lab verwenden, und das in dem in dem Microsoft Entra-Mandanten, der dem Abonnement zugeordnet ist, über die Rolle „Globaler Administrator“ verfügt.
1. Klicken Sie im Azure-Portal auf das Symbol **Cloud Shell**, das sich direkt rechts neben dem Textfeld für die Suche im oberen Bereich der Seite befindet.
1. Wählen Sie bei Aufforderung zur Auswahl von **Bash** oder **PowerShell** die Option **Bash** aus.

   >**Hinweis**: Wenn Sie **Cloud Shell** zum ersten Mal starten und die Meldung **Für Sie wurde kein Speicher bereitgestellt** angezeigt wird, wählen Sie das in diesem Lab verwendete Abonnement aus, und klicken Sie dann auf **Speicher erstellen**.

1. Führen Sie in der **Bash**-Eingabeaufforderung im **Cloud Shell**-Bereich die folgenden Befehle aus, um die Werte des Azure-Abonnement-ID-Attributs abzurufen:

    ```bash
    subscriptionName=$(az account show --query name --output tsv)
    subscriptionId=$(az account show --query id --output tsv)
    echo $subscriptionName
    echo $subscriptionId
    ```

    > **Hinweis**: Kopieren Sie beide Werte in eine Textdatei. Sie werden sie später in diesem Lab benötigen.

1. Führen Sie in der **Bash**-Eingabeaufforderung im Bereich **Cloud Shell** den folgenden Befehl aus, um einen Dienstprinzipal zu erstellen:

    ```bash
    az ad sp create-for-rbac --name sp-az400-azdo --role contributor --scopes /subscriptions/$subscriptionId
    ```

    > **Hinweis**: Der Befehl generiert eine JSON-Ausgabe. Kopieren Sie die Ausgabe in eine Textdatei. Sie benötigen diese später in diesem Lab.

1. Starten Sie als Nächstes auf dem Laborcomputer einen Webbrowser, navigieren Sie zum Azure DevOps **eShopOnWeb**-Projekt. Klicken Sie auf **Project Einstellungen>Dienstverbindungen (unter Pipelines)** und **Neue Dienstverbindung**.
1. Wählen Sie im Bildschirm **Neue Dienstverbindung** die Option **Azure Resource Manager** und anschließend **Weiter** aus (Sie müssen möglicherweise scrollen).
1. Wählen Sie **Dienstprinzipal (manuell)** aus, und klicken Sie auf **Weiter**.
1. Füllen Sie die leeren Felder mit den Informationen aus, die während der vorherigen Schritte gesammelt wurden:
    - Abonnement-ID und -Name.
    - Dienstprinzipal-ID (appId), Dienstprinzipalschlüssel (Kennwort) und Mandanten-ID (Mandant).
    - Geben Sie unter **Name des Dienstverbindungstyps** den Text **azure-connection** ein. Auf diesen Namen wird in YAML-Pipelines verwiesen, wenn eine Azure DevOps-Dienstverbindung erforderlich ist, um mit Ihrem Azure-Abonnement zu kommunizieren.

1. Klicken Sie auf **Überprüfen und speichern**.

### Übung 2: Importieren und Ausführen der CI-Pipeline

In dieser Übung importieren Sie die CI-Pipeline und führen sie aus.

#### Aufgabe 1: Importieren und Ausführen der CI-Pipeline

1. Navigieren Sie zu **Pipelines>Pipelines**
1. Klicken Sie auf die Schaltfläche **Neue Pipeline** (oder **Pipeline erstellen**, wenn Sie noch keine anderen Pipelines erstellt haben)
1. Wählen Sie **Azure Repos Git (YAML)** aus.
1. Wählen Sie das **eShopOnWeb**-Repository
1. Wählen Sie die Option **Vorhandene Azure Pipelines-YAML-Datei** aus.
1. Wählen Sie die **Haupt**-Verzweigung und die Datei **/.ado/eshoponweb-ci-docker.yml** aus, und klicken Sie dann auf **Weiter**.
1. Passen Sie in der YAML-Pipelinedefinition Folgendes an:
   - Ersetzen Sie **YOUR-SUBSCRIPTION-ID** durch Ihre Azure-Abonnement-ID.
   - **rg-az400-container-NAME** mit dem Ressourcengruppennamen, der von der Pipeline erstellt wird (es kann auch eine vorhandene Ressourcengruppe sein).

1. Klicken Sie auf **Speichern und Ausführen**, und warten Sie, bis die Pipeline erfolgreich ausgeführt wird.

    > **Hinweis**: Die Bereitstellung kann einige Minuten dauern.

    Die CI-Definition umfasst die folgenden Aufgaben:
    - **Ressourcen**: Sie lädt die Repositorydateien herunter, die in den folgenden Aufgaben verwendet werden.
    - **AzureResourceManagerTemplateDeployment**: Stellt die Azure-Containerregistrierung mithilfe der Bicep-Vorlage bereit.
    - **PowerShell**: Abrufen des Werts des **ACR-Anmeldeserver** aus der Ausgabe der vorherigen Aufgabe und Erstellen eines neuen Parameters namens **acrLoginServer**
    - [**Docker**](https://learn.microsoft.com/azure/devops/pipelines/tasks/reference/docker-v0?view=azure-pipelines)**– Build**: Erstellen des Docker-Images und Erstellen von zwei Tags (Neueste und aktuelle BuildID)
    - **Docker - Push**:Übertragen des Docker-Images an Azure Container Registry per Pushvorgang.

1. Ihre Pipeline bekommt einen Namen basierend auf dem Projektnamen. Wir **benennen sie um**, damit wir die Pipeline besser identifizieren können. Wechseln Sie zu **Pipelines>Pipelines** , und klicken Sie auf die kürzlich erstellte Pipeline. Klicken Sie auf die Auslassungspunkte und die Option **Umbenennen/Verschieben**. Nennen Sie sie **eshoponweb-ci-docker** und klicken Sie auf **Speichern**.

1. Navigieren Sie zum [**Azure-Portal**](https://portal.azure.com), suchen Sie in der kürzlich erstellten Ressourcengruppe nach der Azure-Containerregistrierung (sie sollte **rg-az400-container-NAME** heißen). Klicken Sie auf der linken Seite unter **Dienste** auf **Repositorys** und stellen Sie sicher, dass das Repository **eshoponweb/web** erstellt wurde. Wenn Sie auf den Repositorylink klicken, sollten zwei Tags angezeigt werden (einer davon ist **neueste**), dies sind die gepushten Images. Wenn diese nicht angezeigt wird, überprüfen Sie den Status Ihrer Pipeline.

### Übung 3: Importieren und Ausführen der CD-Pipeline

In dieser Übung konfigurieren Sie die Dienstverbindung mit Ihrem Azure-Abonnement und führen dann die CD-Pipeline aus.

#### Aufgabe 1: Hinzufügen einer neuen Rollenzuweisung

In dieser Aufgabe fügen Sie eine neue Rollenzuweisung hinzu, um Azure App Service zu erlaubendas Docker-Image aus der Azure-Containerregistrierung abzurufen.

1. Navigieren Sie zum [**Azure-Portal**](https://portal.azure.com).
1. Klicken Sie im Azure-Portal auf das Symbol **Cloud Shell**, das sich direkt rechts neben dem Textfeld für die Suche im oberen Bereich der Seite befindet.
1. Wählen Sie bei Aufforderung zur Auswahl von **Bash** oder **PowerShell** die Option **Bash** aus.
1. Führen Sie in der **Bash**-Eingabeaufforderung im **Cloud Shell**-Bereich die folgenden Befehle aus, um die Werte des Azure-Abonnement-ID-Attributs abzurufen:

    ```sh
    subscriptionId=$(az account show --query id --output tsv)
    echo $subscriptionId
    spId=$(az ad sp list --display-name sp-az400-azdo --query "[].id" --output tsv)
    echo $spId
    roleName=$(az role definition list --name "User Access Administrator" --query "[0].name" --output tsv)
    echo $roleName
    ```

1. Nach dem Abrufen der Dienstprinzipal-ID und des Rollennamens erstellen wir die Rollenzuweisung durch Ausführen dieses Befehls (ersetzen Sie **&lt;rg-az400-container-NAME&gt;** durch ihren Ressourcengruppennamen)

    ```sh
    az role assignment create --assignee $spId --role $roleName --scope /subscriptions/$subscriptionId/resourceGroups/<rg-az400-container-NAME>
    ```

Nun sollte die JSON-Ausgabe angezeigt werden, die den Erfolg der Ausführung des Befehls bestätigt.

#### Übung 2: Importieren und Ausführen der CD-Pipeline

In dieser Aufgabe importieren Sie die CD-Pipeline und führen sie aus.

1. Navigieren Sie zu **Pipelines>Pipelines**
1. Klicken Sie auf die Schaltfläche **Neue Pipeline**
1. Wählen Sie **Azure Repos Git (YAML)** aus.
1. Wählen Sie das **eShopOnWeb**-Repository
1. Wählen Sie die Option **Vorhandene Azure Pipelines-YAML-Datei** aus
1. Wählen Sie die **Haupt**-Verzweigung und die Datei **/.ado/eshoponweb-cd-webapp-docker.yml** aus, und klicken Sie dann auf **Weiter**.
1. Passen Sie in der YAML-Pipelinedefinition Folgendes an:
   - Ersetzen Sie **YOUR-SUBSCRIPTION-ID** durch Ihre Azure-Abonnement-ID.
   - **rg-az400-container-NAME** mit dem zuvor im Labor definierten Ressourcengruppennamen.

1. Klicken Sie auf **Speichern und Ausführen**, und warten Sie, bis die Pipeline erfolgreich ausgeführt wird.

    > **Hinweis**: Die Bereitstellung kann einige Minuten dauern.

    > **Wichtig**: Wenn Sie die Fehlermeldung "TF402455: Pushes in diesen Branch sind nicht zulässig. Sie müssen eine Pull-Anforderung verwenden, um diesen Branch zu aktualisieren", müssen Sie die Branch-Schutzregel "Eine Mindestanzahl von Prüfern erforderlich" deaktivieren, die in den vorherigen Labs aktiviert war.

    Die CD-Definition besteht aus den folgenden Aufgaben:
    - **Ressourcen**: Sie lädt die Repositorydateien herunter, die in den folgenden Aufgaben verwendet werden.
    - **AzureResourceManagerTemplateDeployment**: Stellt den Azure App Service mithilfe der Bicep-Vorlage bereit.
    - **AzureResourceManagerTemplateDeployment**:Hinzufügen von Azure-Rollenzuweisungen mithilfe von Bicep

1. Ihre Pipeline bekommt einen Namen basierend auf dem Projektnamen. Wir **benennen sie um**, damit wir die Pipeline besser identifizieren können. Wechseln Sie zu **Pipelines>Pipelines** und zeigen Sie auf die kürzlich erstellte Pipeline. Klicken Sie auf die Auslassungspunkte und die Option **Umbenennen/Verschieben**. Nennen Sie sie **eshoponweb-cd-webapp-docker** und klicken Sie auf **Speichern**.

    > **Hinweis 1**: Die Verwendung der Vorlage **/infra/webapp-docker.bicep** erstellt einen App Service-Plan, eine Web-App mit aktivierter systemseitig zugewiesener verwalteter Identität und verweist auf das zuvor gepushte Docker-Image **${acr.properties.loginServer}/eshoponweb/web:latest**.

    > **Hinweis 2**: Die Verwendung der Vorlage **/infra/webapp-to-acr-roleassignment.bicep** erstellt eine neue Rollenzuweisung für die Web-App mit AcrPull-Rolle, um das Docker-Image abrufen zu können. Dies kann in der ersten Vorlage erfolgen, da die Rollenzuweisung jedoch einige Zeit in Anspruch nehmen kann, ist es ratsam, beide Aufgaben separat auszuführen.

#### Aufgabe 3: Testen der Lösung

1. Navigieren Sie im Azure-Portal zur zuletzt erstellten Ressourcengruppe. Nun sollten drei Ressourcen (App Service, App Service Plan und Containerregistrierung) angezeigt werden.

1. Navigieren Sie zum App Service und klicken Sie auf **Durchsuchen**, damit gelangen Sie zur Website.

Herzlichen Glückwunsch! In dieser Übung haben Sie eine Website mit einem benutzerdefinierten Docker-Image bereitgestellt.

### Übung 4: Entfernen der Azure Lab-Ressourcen

In dieser Übung entfernen Sie die in dieser Übung bereitgestellten Azure-Ressourcen, um unerwartete Gebühren zu beseitigen.

>**Hinweis**: Denken Sie daran, alle neu erstellten Azure-Ressourcen zu entfernen, die Sie nicht mehr verwenden. Durch das Entfernen nicht verwendeter Ressourcen wird sichergestellt, dass keine unerwarteten Gebühren anfallen.

#### Aufgabe 1: Entfernen der Azure Lab-Ressourcen

In dieser Aufgabe verwenden Sie Azure Cloud Shell, um die in diesem Lab bereitgestellten Azure-Ressourcen zu entfernen, um unnötige Gebühren zu vermeiden.

1. Öffnen Sie im Azure-Portal die **Bash**-Shell-Sitzung im Bereich **Cloud Shell**.
1. Listen Sie alle Ressourcengruppen auf, die während der Labs in diesem Modul erstellt wurden, indem Sie den folgenden Befehl ausführen:

    ```sh
    az group list --query "[?starts_with(name,'rg-az400-container-')].name" --output tsv
    ```

1. Löschen Sie alle Ressourcengruppen, die Sie während der praktischen Übungen in diesem Modul erstellt haben, indem Sie den folgenden Befehl ausführen:

    ```sh
    az group list --query "[?starts_with(name,'rg-az400-container-')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**Hinweis**: Der Befehl wird (dem --nowait-Parameter entsprechend) asynchron ausgeführt. Dies bedeutet, dass Sie zwar einen weiteren Azure CLI-Befehl in derselben Bash-Sitzung direkt im Anschluss ausführen können, es jedoch einige Minuten dauert, bis die Ressourcengruppen tatsächlich entfernt werden.

## Überprüfung

In diesem Lab haben Sie erfahren, wie Sie mithilfe einer Azure DevOps CI/CD-Pipeline ein benutzerdefiniertes Docker-Image erstellen, es an Azure Container Registry übergeben und es als Container für Azure App Service bereitstellen.
