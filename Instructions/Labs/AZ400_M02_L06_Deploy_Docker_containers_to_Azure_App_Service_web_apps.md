---
lab:
  title: Bereitstellen von Docker-Containern für Azure App Service-Web-Apps
  module: 'Module 02: Implement CI with Azure Pipelines and GitHub Actions'
---

# Bereitstellen von Docker-Containern für Azure App Service-Web-Apps

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

## Geschätzte Zeit: 20 Minuten

## Anweisungen

### Übung 0: (Überspringen, wenn bereits abgeschlossen) Konfigurieren der Lab-Voraussetzungen

In dieser Übung richten Sie die Voraussetzungen für das Lab ein, das aus einem neuen Azure DevOps-Projekt mit einem Repository basierend auf dem [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb) besteht.

#### Aufgabe 1: (überspringen, wenn fertig) Erstellen und Konfigurieren des Teamprojekts

In dieser Aufgabe erstellen Sie ein **eShopOnWeb** Azure DevOps-Projekt, das von mehreren Labs verwendet werden soll.

1. Öffnen Sie auf Ihrem Lab-Computer in einem Browserfenster Ihre Azure DevOps-Organisation. Klicken Sie auf **Neues Projekt**. Geben Sie Ihrem Projekt den Namen **eShopOnWeb**, und wählen Sie **Scrum** in der Dropdownliste **Arbeitselementprozess** aus. Klicken Sie auf **Erstellen**.

#### Aufgabe 2: (überspringen, wenn erledigt) Importieren von eShopOnWeb Git Repository

Bei dieser Aufgabe importieren Sie das eShopOnWeb Git-Repository, das von mehreren Labs verwendet wird.

1. Öffnen Sie auf Ihrem Lab-Computer in einem Browserfenster Ihre Azure DevOps-Organisation und das zuvor erstellte **eShopOnWeb**-Projekt. Klicken Sie auf **Repos > Files**, **Importieren**. Fügen Sie im Fenster **Git Repository importieren** die folgende URL <https://github.com/MicrosoftLearning/eShopOnWeb.git> ein und klicken Sie auf **Importieren**:

1. Das Repository ist wie folgt organisiert:
    - Der Ordner **.ado** enthält Azure DevOps-YAML-Pipelines.
    - Der Ordner **.devcontainer** enthält ein Containersetup für die Entwicklung mithilfe von Containern (entweder lokal in VS Code oder über GitHub Codespaces).
    - Der Ordner **infra** enthält eine Bicep&ARM-Infrastruktur als Codevorlagen, die in einigen Labszenarien verwendet werden.
    - Der Ordner **.github** enthält YAML GitHub-Workflow-Definitionen.
    - Der Ordner **src** enthält die .NET 8-Website, die in den Labszenarios verwendet wird.

#### Aufgabe 3: (überspringen, wenn erledigt) Legen Sie den Mainbranch als Standardbranch fest

1. Wechseln Sie zu **Repos > Branches**.
1. Bewegen Sie den Mauszeiger auf den **Main**-Branch und klicken Sie dann rechts neben der Spalte auf die Auslassungspunkte.
1. Klicken Sie auf **Als Mainbranch festlegen**.

### Übung 1: Importieren und Ausführen der CI-Pipeline

In dieser Übung konfigurieren Sie die Dienstverbindung mit Ihrem Azure-Abonnement, importieren die CI-Pipeline und führen sie dann aus.

#### Aufgabe 1: Importieren und Ausführen der CI-Pipeline

1. Wechseln Sie zu **Pipelines > Pipelines**.
1. Klicken Sie auf die Schaltfläche **Neue Pipeline** (oder **Pipeline erstellen**, wenn Sie noch keine anderen Pipelines erstellt haben)
1. Wählen Sie **Azure Repos Git (YAML)** aus.
1. Wählen Sie das **eShopOnWeb**-Repository
1. Wählen Sie die Option **Vorhandene Azure Pipelines-YAML-Datei** aus.
1. Wählen Sie die **Haupt**-Verzweigung und die Datei **/.ado/eshoponweb-ci-docker.yml** aus, und klicken Sie dann auf **Weiter**.
1. Passen Sie in der YAML-Pipelinedefinition Folgendes an:
   - Ersetzen Sie **YOUR-SUBSCRIPTION-ID** durch Ihre Azure-Abonnement-ID.
   - Ersetzen Sie **resourceGroup** durch den während der Erstellung der Dienstverbindung verwendeten Ressourcengruppennamen, z. B. **AZ400-RG1**.

1. Lesen Sie die Pipelinedefinition. Die CI-Definition umfasst die folgenden Aufgaben:
    - **Ressourcen**: Sie lädt die Repositorydateien herunter, die in den folgenden Aufgaben verwendet werden.
    - **AzureResourceManagerTemplateDeployment**: Stellt die Azure-Containerregistrierung mithilfe der Bicep-Vorlage bereit.
    - **PowerShell**: Abrufen des Werts des **ACR-Anmeldeserver** aus der Ausgabe der vorherigen Aufgabe und Erstellen eines neuen Parameters namens **acrLoginServer**
    - [**Docker**](https://learn.microsoft.com/azure/devops/pipelines/tasks/reference/docker-v0?view=azure-pipelines)**– Build**: Erstellen des Docker-Images und Erstellen von zwei Tags (Neueste und aktuelle BuildID)
    - **Docker - Push**:Übertragen des Docker-Images an Azure Container Registry per Pushvorgang.

1. Klicken Sie auf **Speichern und Ausführen**.

1. Öffnen Sie die Pipelineausführung. Wenn die Warnmeldung „Diese Pipeline benötigt die Berechtigung zum Zugriff auf eine Ressource, bevor diese Ausführung mit der Erstellung fortfahren kann“ angezeigt wird, klicken Sie auf **Anzeigen**, **Zulassen** und erneut **Zulassen**. Dadurch kann die Pipeline auf das Azure-Abonnement zugreifen.

    > **Hinweis**: Die Bereitstellung kann einige Minuten dauern.

1. Ihre Pipeline bekommt einen Namen basierend auf dem Projektnamen. Wir **benennen sie um**, damit wir die Pipeline besser identifizieren können. Wechseln Sie zu **Pipelines > Pipelines**, und klicken Sie auf die kürzlich erstellte Pipeline. Klicken Sie auf die Auslassungspunkte und die Option **Umbenennen/Verschieben**. Nennen Sie sie **eshoponweb-ci-docker** und klicken Sie auf **Speichern**.

1. Navigieren Sie zum [**Azure-Portal**](https://portal.azure.com), und suchen Sie in der kürzlich erstellten Ressourcengruppe nach der Azure-Containerregistrierung (sie sollte **AZ400-RG1** heißen). Klicken Sie auf der linken Seite unter **Dienste** auf **Repositorys** und stellen Sie sicher, dass das Repository **eshoponweb/web** erstellt wurde. Wenn Sie auf den Repositorylink klicken, sollten zwei Tags angezeigt werden (einer davon ist **neueste**), dies sind die gepushten Images. Wenn diese nicht angezeigt wird, überprüfen Sie den Status Ihrer Pipeline.

### Übung 2: Importieren und Ausführen der CD-Pipeline

In dieser Übung konfigurieren Sie die Dienstverbindung mit Ihrem Azure-Abonnement und führen dann die CD-Pipeline aus.

#### Aufgabe 1: Importieren und Ausführen der CD-Pipeline

In dieser Aufgabe importieren Sie die CD-Pipeline und führen sie aus.

1. Wechseln Sie zu **Pipelines > Pipelines**.
1. Klicken Sie auf die Schaltfläche **Neue Pipeline**
1. Wählen Sie **Azure Repos Git (YAML)** aus.
1. Wählen Sie das **eShopOnWeb**-Repository
1. Wählen Sie die Option **Vorhandene Azure Pipelines-YAML-Datei** aus
1. Wählen Sie die **Haupt**-Verzweigung und die Datei **/.ado/eshoponweb-cd-webapp-docker.yml** aus, und klicken Sie dann auf **Weiter**.
1. Passen Sie in der YAML-Pipelinedefinition Folgendes an:
   - Ersetzen Sie **YOUR-SUBSCRIPTION-ID** durch Ihre Azure-Abonnement-ID.
   - Ersetzen Sie **resourceGroup** durch den während der Erstellung der Dienstverbindung verwendeten Ressourcengruppennamen, z. B. **AZ400-RG1**.
   - Ersetzen Sie **Speicherort** durch die Azure-Region, in der die Ressourcen bereitgestellt werden.

1. Lesen Sie die Pipelinedefinition. Die CD-Definition besteht aus den folgenden Aufgaben:
    - **Ressourcen**: Sie lädt die Repositorydateien herunter, die in den folgenden Aufgaben verwendet werden.
    - **AzureResourceManagerTemplateDeployment**: Stellt den Azure App Service mithilfe der Bicep-Vorlage bereit.
    - **AzureResourceManagerTemplateDeployment**:Hinzufügen von Azure-Rollenzuweisungen mithilfe von Bicep

1. Klicken Sie auf **Speichern und Ausführen**.

1. Öffnen Sie die Pipelineausführung. Wenn die Warnmeldung „Diese Pipeline benötigt die Berechtigung zum Zugriff auf eine Ressource, bevor diese Ausführung mit der Bereitstellung fortfahren kann“ angezeigt wird, klicken Sie auf **Anzeigen**, **Zulassen** und erneut **Zulassen**. Dadurch kann die Pipeline auf das Azure-Abonnement zugreifen.

    > **Wichtig:** Wenn Sie die Pipeline beim Konfigurieren nicht autorisieren, treten während der Ausführung Berechtigungsfehler auf. Häufige Fehlermeldungen sind u. a. „Diese Pipeline benötigt die Berechtigung für den Zugriff auf eine Ressource“ oder „Pipelineausführung aufgrund unzureichender Berechtigungen nicht erfolgreich“. Um diese Fehler zu beheben, navigieren Sie zur Pipelineausführung, klicken neben der Berechtigungsanforderung auf **Anzeigen** und klicken dann auf **Zulassen**, um den erforderlichen Zugriff auf Ihr Azure-Abonnement und Ihre Ressourcen zu gewähren.

    > **Hinweis**: Die Bereitstellung kann einige Minuten dauern.

    > [!IMPORTANT]
    > Wenn Sie die Fehlermeldung „TF402455: Pushes in diesen Branch sind nicht zulässig. Sie müssen eine Pull-Anforderung verwenden, um diesen Branch zu aktualisieren.“ sehen, müssen Sie die Branch-Schutzregel „Mindestanzahl von Reviewern erforderlich“ deaktivieren, die in den vorherigen Labs aktiviert war.

1. Ihre Pipeline bekommt einen Namen basierend auf dem Projektnamen. Wir **benennen sie um**, damit wir die Pipeline besser identifizieren können. Wechseln Sie zu **Pipelines > Pipelines**, und zeigen Sie auf die kürzlich erstellte Pipeline. Klicken Sie auf die Auslassungspunkte und die Option **Umbenennen/Verschieben**. Nennen Sie sie **eshoponweb-cd-webapp-docker** und klicken Sie auf **Speichern**.

    > **Hinweis 1**: Die Verwendung der Vorlage **/infra/webapp-docker.bicep** erstellt einen App Service-Plan, eine Web-App mit aktivierter systemseitig zugewiesener verwalteter Identität und verweist auf das zuvor gepushte Docker-Image **${acr.properties.loginServer}/eshoponweb/web:latest**.

    > **Hinweis 2**: Die Verwendung der Vorlage **/infra/webapp-to-acr-roleassignment.bicep** erstellt eine neue Rollenzuweisung für die Web-App mit AcrPull-Rolle, um das Docker-Image abrufen zu können. Dies kann in der ersten Vorlage erfolgen, da die Rollenzuweisung jedoch einige Zeit in Anspruch nehmen kann, ist es ratsam, beide Aufgaben separat auszuführen.

#### Aufgabe 2: Testen der Lösung

1. Navigieren Sie im Azure-Portal zur zuletzt erstellten Ressourcengruppe. Nun sollten drei Ressourcen (App Service, App Service Plan und Containerregistrierung) angezeigt werden.

1. Navigieren Sie zum App Service und klicken Sie auf **Durchsuchen**, damit gelangen Sie zur Website.

1. Stellen Sie sicher, dass die eShopOnWeb-Anwendung erfolgreich ausgeführt wird. Nach der Bestätigung haben Sie das Lab erfolgreich abgeschlossen.

> [!IMPORTANT]
> Denken Sie daran, die im Azure-Portal erstellten Ressourcen zu löschen, um unnötige Gebühren zu vermeiden.

## Überprüfung

In diesem Lab haben Sie erfahren, wie Sie mithilfe einer Azure DevOps CI/CD Pipeline ein benutzerdefiniertes Docker-Image erstellen, es an Azure Container Registry übergeben und es als Container für Azure App Service bereitstellen.
