---
lab:
  title: 'Lab: Integrieren von Azure Key Vault mit Azure DevOps'
  module: 'Module 05: Implement a secure continuous deployment using Azure Pipelines'
---

# Lab: Integrieren von Azure Key Vault mit Azure DevOps

## Lab-Handbuch für Kursteilnehmer

## Labanforderungen

- Für dieses Lab ist **Microsoft Edge** oder ein [von Azure DevOps unterstützter Browser](https://learn.microsoft.com/azure/devops/server/compatibility) erforderlich.

- **Einrichten einer Azure DevOps-Organisation**: Wenn Sie nicht bereits eine Azure DevOps-Organisation haben, die Sie für dieses Lab verwenden können, müssen Sie diese erstellen, indem Sie die unter [Erstellen einer Organisation oder Projektsammlung](https://learn.microsoft.com/azure/devops/organizations/accounts/create-organization) beschriebenen Anweisungen befolgen.

- Identifizieren Sie ein vorhandenes Azure-Abonnement, oder erstellen Sie ein neues Abonnement.

## Übersicht über das Labor

Azure Key Vault bietet eine sichere Speicherung und Verwaltung von vertraulichen Daten, wie z. B. Schlüsseln, Passwörtern und Zertifikaten. Azure Key Vault unterstützt Hardware-Sicherheitsmodule und eine Reihe von Verschlüsselungsalgorithmen und Schlüssellängen. Mit Azure Key Vault können Sie die Möglichkeit der Offenlegung vertraulicher Daten über den Quellcode minimieren, was ein häufiger Fehler von Entwicklern ist. Der Zugriff auf Azure Key Vault erfordert eine ordnungsgemäße Authentifizierung und Autorisierung, die fein gestrichelte Berechtigungen für seine Inhalte unterstützt.

In diesem Lab werden Sie sehen, wie Sie Azure Key Vault mit Hilfe der folgenden Schritte in eine Azure-Pipeline integrieren können:

- Erstellen Sie einen Azure Key Vault, um ein ACR-Kennwort als Geheimnis zu speichern.
- Erstellen Sie einen Azure-Dienstprinzipal, um den Zugriff auf die Geheimnisse von Azure Key Vault zu ermöglichen.
- Konfigurieren Sie die Berechtigungen, damit der Dienstprinzipal das Geheimnis lesen kann.
- Konfigurieren Sie die Pipeline so, dass das Kennwort von Azure Key Vault abgerufen und an nachfolgende Aufgaben weitergegeben wird.

## Ziele

In diesem Lab lernen Sie Folgendes:

- Erstellen Sie einen Microsoft Entra-Dienstprinzipal.
- Erstellen Sie eine Azure Key Vault-Instanz.

## Geschätzte Zeit: 40 Minuten

## Anweisungen

### Übung 0: Konfigurieren der Voraussetzungen für das Lab.

In dieser Übung richten Sie die Voraussetzungen für das Labor ein, das aus einem neuen Azure DevOps-Projekt mit einem Repository basierend auf dem [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb) besteht.

#### Aufgabe 1: (überspringen, wenn fertig) Erstellen und Konfigurieren des Teamprojekts

In dieser Aufgabe erstellen Sie ein **eShopOnWeb** Azure DevOps-Projekt, das von mehreren Laboren verwendet werden soll.

1. Öffnen Sie auf Ihrem Laborcomputer in einem Browserfenster Ihre Azure DevOps-Organisation. Klicken auf „Neues Projekt“ Weisen Sie Ihrem Projekt den Namen **"eShopOnWeb** " zu, und lassen Sie die anderen Felder standardmäßig. Klicken Sie auf **Erstellen**.

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

### Übung 1: Einrichten einer CI-Pipeline zum Erstellen eines eShopOnWeb-Containers

Einrichten der CI YAML-Pipeline für:

- Pushen des Containerimages in die Azure-Containerregistrierung
- Verwenden von Docker Compose zum Erstellen und Pushen **von eshoppublicapi** und **eshopwebmvc-Containerimages** . Nur **eshopwebmvc-Container** wird bereitgestellt.

#### Aufgabe 1: (überspringen, wenn erledigt) Erstellen eines Dienstprinzipals

In dieser Aufgabe erstellen Sie einen Dienstprinzipal mithilfe der Azure CLI, der Azure DevOps zulässt:

- Bereitstellen von Ressourcen in einem Azure-Abonnement
- Haben Sie Lesezugriff auf die später erstellten Schlüsseltresorschlüssel.

> **Hinweis**: Wenn Sie bereits über einen Dienstprinzipal verfügen, können Sie direkt mit der nächsten Aufgabe fortfahren.

Sie benötigen einen Dienstprinzipal, um Azure-Ressourcen aus Azure-Pipelines bereitzustellen. Da wir geheime Schlüssel in einer Pipeline abrufen werden, müssen wir dem Dienst die Berechtigung erteilen, wenn wir den Azure Key Vault erstellen.

Ein Dienstprinzipal wird automatisch von Azure Pipelines erstellt, wenn Sie eine Verbindung mit einem Azure-Abonnement innerhalb einer Pipelinedefinition herstellen oder wenn Sie eine neue Dienst-Verbinden ion über die Projekteinstellungsseite (automatische Option) erstellen. Sie können den Dienstprinzipal auch manuell über das Portal erstellen oder Azure CLI verwenden und es für alle Projekte wiederverwenden.

1. Starten Sie auf Ihrem Labcomputer einen Webbrowser, navigieren Sie zum [**Azure-Portal**](https://portal.azure.com), und melden Sie sich an. Verwenden Sie hierzu die Anmeldeinformationen eines Benutzerkontos, das in dem Abonnement, das Sie in diesem Lab verwenden, über die Rolle „Besitzer“ und in dem Azure AD-Mandanten, der dem Azure-Abonnement zugeordnet ist, über die Rolle „Globaler Administrator“ verfügt.
2. Öffnen Sie im Azure-Portal den Bereich **Cloud Shell**, indem Sie auf das Symbolleistensymbol rechts neben dem Textfeld für die Suche klicken.
3. Wählen Sie bei Aufforderung zur Auswahl von **Bash** oder **PowerShell** die Option **Bash** aus.

   >**Hinweis**: Wenn Sie **Cloud Shell** zum ersten Mal starten und die Meldung **Für Sie wurde kein Speicher bereitgestellt** angezeigt wird, wählen Sie das in diesem Lab verwendete Abonnement aus, und klicken Sie dann auf **Speicher erstellen**.

4. Führen Sie in der **Bash-Eingabeaufforderung** im **Cloud Shell-Bereich** die folgenden Befehle aus, um die Werte der Azure-Abonnement-ID und der Abonnementnamenattribute abzurufen:

    ```bash
    az account show --query id --output tsv
    az account show --query name --output tsv
    ```

    > **Hinweis**: Kopieren Sie beide Werte in eine Textdatei. Sie werden sie später in diesem Lab benötigen.

5. Führen Sie in der Bash-Eingabeaufforderung** im **Cloud Shell-Bereich** den folgenden Befehl aus, um einen Dienstprinzipal zu erstellen (ersetzen Sie den **myServicePrincipalName** durch eine beliebige Zeichenfolge aus Buchstaben und Ziffern) und **mySubscriptionID** durch Ihre Azure subscriptionId**:

    ```bash
    az ad sp create-for-rbac --name myServicePrincipalName \
                         --role contributor \
                         --scopes /subscriptions/mySubscriptionID
    ```

    > **Hinweis**: Der Befehl generiert eine JSON-Ausgabe. Speichern der Ausgabe in einer Textdatei Sie benötigen ihn später in diesem Lab.

6. Starten Sie als Nächstes auf dem Laborcomputer einen Webbrowser, navigieren Sie zum Azure DevOps **eShopOnWeb-Projekt** . Klicken Sie auf **Project Einstellungen>Service-Verbinden ionen (unter Pipelines)** und **Verbinden ion** neuer Dienst.

    ![Neue Dienstverbindung](images/new-service-connection.png)

7. Wählen Sie im Bildschirm **Neue Dienstverbindung** die Option **Azure Resource Manager** und anschließend **Weiter** aus.

8. Wählen Sie **den Dienstprinzipal (manuell)** aus, und klicken Sie auf **"Weiter"**.

9. Füllen Sie die leeren Felder mit den informationen aus, die während der vorherigen Schritte gesammelt wurden:
    - Abonnement-ID oder -Name.
    - Dienstprinzipal-ID (appId), Dienstprinzipalschlüssel (Kennwort) und Mandanten-ID (Mandant).
    - Geben Sie unter **"Dienstverbindungsname****" Azure-Untere ein**. Auf diesen Namen wird in YAML-Pipelines verwiesen, wenn ein Azure DevOps Service Verbinden ion erforderlich ist, um mit Ihrem Azure-Abonnement zu kommunizieren.

    ![Azure-Serviceverbindung](images/azure-service-connection.png)

10. Klicken Sie auf **Überprüfen und speichern**.

#### Aufgabe 2: Einrichten und Ausführen der CI-Pipeline

In dieser Aufgabe importieren Sie eine vorhandene CI YAML-Pipelinedefinition, ändern und ausführen sie. Sie erstellt eine neue Azure Container Registry (ACR) und erstellt/veröffentlicht die eShopOnWeb-Containerimages.

1. Starten Sie auf dem Laborcomputer einen Webbrowser, navigieren Sie zum Azure DevOps **eShopOnWeb-Projekt** . Wechseln Sie zu **Pipelines>Pipelines**, und klicken Sie auf "Pipeline** erstellen" (oder **"** Neue Pipeline").**

2. Wählen Sie **im **Fenster "Wo befindet Sich Ihr Code?** " Azure Repos Git (YAML)** aus, und wählen Sie das **eShopOnWeb-Repository** aus.

3. Wählen Sie auf der Registerkarte **Konfigurieren** die Option aus, um eine **Vorhandene Azure Pipelines YAML-Datei** zu verwenden. Geben Sie den folgenden Pfad **/.ado/eshoponweb-ci-dockercompose.yml** an, und klicken Sie auf **"Weiter"**.

    ![Wählen Sie Pipeline aus.](images/select-ci-container-compose.png)

4. Passen Sie in der YAML-Pipelinedefinition Ihren Ressourcengruppennamen an, indem Sie NAME** unter AZ400-EWebShop-NAME** ersetzen **und IHRE-SUBSCRIPTION-ID** durch ihre eigene Azure-subscriptionId ersetzen****.

5. Klicken Sie auf " **Speichern und Ausführen** ", und warten Sie, bis die Pipeline erfolgreich ausgeführt wird.

    > Der Erstellungsvorgang nimmt einige Minuten in Anspruch. Die Buildpipeline besteht aus den folgenden Aufgaben:
    - **AzureResourceManagerTemplateDeployment** verwendet **bicep** zum Bereitstellen einer Azure-Containerregistrierung.
    - **Die PowerShell-Aufgabe** übernimmt die Bicep-Ausgabe (acr-Anmeldeserver) und erstellt eine Pipelinevariable.
    - **DockerCompose-Aufgabenbuilds** und pushen die Containerimages für eShopOnWeb in die Azure Container Registry.

6. Ihre Pipeline nimmt einen Namen auf der Grundlage des Projektnamens an. Hiermit können Sie **sie umbenennen** , um die Pipeline besser zu identifizieren. Wechseln Sie zu **Pipelines>Pipelines** , und klicken Sie auf die kürzlich erstellte Pipeline. Klicken Sie auf die Elipse und **die Option "Umbenennen/Entfernen"** . Nennen Sie es **eshoponweb-ci-dockercompose** , und klicken Sie auf **"Speichern"**.

7. Nachdem die Ausführung abgeschlossen ist, öffnen Sie im Azure-Portal zuvor definierte Ressourcengruppe, und Sie sollten eine Azure Container Registry (ACR) mit den erstellten Containerimages **eshoppublicapi** und **eshopwebmvc** finden. Sie verwenden **eshopwebmvc** nur in der Bereitstellungsphase.

    ![Containerimages in ACR](images/azure-container-registry.png)

8. Klicken Sie auf **Zugriffstasten** , und kopieren Sie den **Kennwortwert** , sie wird in der folgenden Aufgabe verwendet, da wir ihn als geheimer Schlüssel im Azure Key Vault beibehalten.

    ![ACR-Kennwort](images/acr-password.png)

#### Aufgabe 3: Erstellen einer Azure Key Vault-Instanz

In dieser Aufgabe erstellen Sie einen Azure Key Vault mithilfe der Azure-Portal.

In diesem Übungsszenario haben wir eine Azure Container Instance (ACI), die ein Containerimage abruft und ausführt, das in der Azure Container Registry (ACR) gespeichert ist. Wir beabsichtigen, das Kennwort für die ACR als geheimen Schlüssel im Schlüsseltresor zu speichern.

1. Geben Sie im Azure-Portal oben auf der Azure-Portalseite im Textfeld **Nach Ressourcen, Diensten und Dokumenten suchen** den Begriff **Ressourcen** ein, und drücken Sie die **EINGABETASTE**.
2. Wählen Sie auf dem Blatt **Schlüsseltresore** die Option **Schlüsseltresor erstellen** aus.
3. Geben Sie auf dem Blatt **Arbeitsbereich erstellen** auf der Registerkarte **Grundeinstellungen** die folgenden Einstellungen an, und wählen Sie Weiter:

    | Einstellung | Wert |
    | --- | --- |
    | Subscription | Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden. |
    | Resource group | Der Name einer neuen Ressourcengruppe **az104-10-rg1**. |
    | Name des Schlüsseltresors | beliebiger eindeutiger gültiger Name, z **. B. ewebshop-kv-NAME** (name ersetzen) |
    | Region | Wählen Sie die Azure-Region aus, die dem Standort Ihrer Laborumgebung am nächsten ist. |
    | Tarif | **Standard** |
    | Aufbewahrungsdauer für gelöschte Tresore in Tagen | **7** |
    | Bereinigungsschutz | Aktivieren des Löschschutzes |

4. Wählen Sie auf der **Registerkarte "Access-Konfiguration**" des **Blatts "Schlüsseltresor** erstellen" die Option "Tresorzugriffsrichtlinie **" aus, und klicken Sie dann im **Abschnitt "Access-Richtlinien**" auf "**+ Erstellen**", um eine neue Richtlinie **einzurichten.

    > Sie müssen den Zugriff auf Ihre wichtigsten Tresore sichern, indem Sie nur autorisierte Anwendungen und Benutzer zulassen. Um auf die Daten aus dem Tresor zuzugreifen, müssen Sie Leseberechtigungen (Get/List) für den zuvor erstellten Dienstprinzipal bereitstellen, den Sie für die Authentifizierung in der Pipeline verwenden. 

    1. Überprüfen Sie auf dem **Blatt "Berechtigung"** unter "**Geheime Berechtigungen**" die Berechtigung "Abrufen** und **Auflisten****". Klicken Sie auf **Weiter**.
    2. Suchen Sie auf dem **Blatt "Prinzipal** " nach dem **zuvor erstellten Dienstprinzipal**, entweder mithilfe der angegebenen ID oder des angegebenen Namens, und wählen Sie ihn aus der Liste aus. Klicken Sie auf **"Weiter", "** Weiter **", ****"Erstellen**" (Zugriffsrichtlinie).
    3. Klicken Sie auf dem Blatt **Überprüfen und erstellen** auf **Erstellen**.

5. Zurück auf dem **Blatt "Schlüsseltresor** erstellen" klicken Sie auf " **Überprüfen" + "Erstellen" > Erstellen**

    > **Hinweis**: Warten Sie, bis der Azure Load Balancer bereitgestellt wurde. Das sollte weniger als eine Minute dauern.

6. Klicken Sie auf der Seite **Ihre Bereitstellung wurde abgeschlossen** auf **Zu Ressource wechseln**.
7. Klicken Sie auf dem Blatt Azure Key Vault (ewebshop-kv-NAME) im vertikalen Menü auf der linken Seite des Blatts im **Abschnitt "Objekte** " auf **"Geheime Schlüssel"**.
8. Klicken Sie auf dem **Blatt "Geheime Schlüssel** " auf " **Generieren/Importieren"**.
9. Geben Sie auf dem Blatt **Neuer Container** die folgenden Einstellungen an (übernehmen Sie die Standardwerte für andere Einstellungen), und klicken Sie dann auf **Erstellen**:

    | Einstellung | Wert |
    | --- | --- |
    | Uploadoptionen | **Manuell** |
    | Name | **acr-secret** |
    | Wert | ACR-Zugriffskennwort, das in der vorherigen Aufgabe kopiert wurde |

#### Aufgabe 3: Erstellen einer variablen Gruppe, die mit Azure Key Vault verbunden ist

In dieser Aufgabe erstellen Sie eine Variable Gruppe in Azure DevOps, die den ACR-Kennwortschlüssel mithilfe des Dienst-Verbinden ion (Dienstprinzipal) aus dem Key Vault abruft.

1. Starten Sie auf Ihrem Laborcomputer einen Webbrowser, und navigieren Sie zum Azure DevOps-Projekt **eShopOnWeb**.

2. Wählen Sie **im vertikalen Navigationsbereich des Azure DevOps-Portals Pipelines>Library** aus. Klicken Sie auf „Variablengruppen“.

3. Geben Sie auf dem Blatt **Neue Dateifreigabe** die folgenden Einstellungen an:

    | Einstellung | Wert |
    | --- | --- |
    | Variablengruppenname | **eshopweb-vg** |
    | Verlinken von Geheimnissen aus einem Azure-Schlüsseltresor | **enable** |
    | Azure-Abonnement | Verfügbare Azure-Dienstverbindungen |
    | Name des Schlüsseltresors | Der Name Ihres Schlüsseltresors|

4. Klicken Sie unter **"Variablen**" auf " **+Hinzufügen"** , und wählen Sie den **geheimen Schlüssel "acr-secret** " aus. Klicken Sie auf **OK**.
5. Klicken Sie auf **Speichern**.

    ![Variable Gruppe erstellen](images/vg-create.png)

#### Aufgabe 4: Einrichten der CD-Pipeline zum Bereitstellen von Containern in Azure Container Instance (ACI)

In dieser Aufgabe importieren Sie eine CD-Pipeline, passen sie an und führen sie aus, um das zuvor in einer Azure-Containerinstanz erstellte Containerimage bereitzustellen.

1. Starten Sie auf dem Laborcomputer einen Webbrowser, navigieren Sie zum Azure DevOps **eShopOnWeb-Projekt** . Wechseln Sie zu **Pipelines>Pipelines** , und klicken Sie auf **"Neue Pipeline"**.

2. Wählen Sie **im **Fenster "Wo befindet Sich Ihr Code?** " Azure Repos Git (YAML)** aus, und wählen Sie das **eShopOnWeb-Repository** aus.

3. Wählen Sie auf der Registerkarte **Konfigurieren** die Option aus, um eine **Vorhandene Azure Pipelines YAML-Datei** zu verwenden. Geben Sie den folgenden Pfad **/.ado/eshoponweb-cd-aci.yml** an, und klicken Sie auf 'Weiter'****.

4. Passen Sie in der YAML-Pipelinedefinition Folgendes an:

    - Ersetzen Sie <yourSubscriptionId> durch Ihre Azure-Abonnement-ID.
    - **az400eshop-NAME** ersetzt NAME, um ihn global eindeutig zu machen.
    - **** YOUR-ACR.azurecr.io und **ACR-USERNAME** mit Ihrem ACR-Anmeldeserver (beide benötigen den ACR-Namen, können auf den ACR>Access Keys überprüft werden).
    - **AZ400-EWebShop-NAME** mit dem zuvor im Labor definierten Ressourcengruppennamen.

5. Klicken Sie auf **Speichern und ausführen**.
6. Öffnen Sie die Pipeline, und warten Sie, bis sie erfolgreich ausgeführt wird.

    > **Wichtig**: Wenn die Meldung "Diese Pipeline benötigt Berechtigungen für den Zugriff auf Ressourcen benötigt, bevor diese Ausführung weiterhin zu Docker Compose to ACI führen kann" angezeigt wird, klicken Sie erneut auf "Anzeigen", "Genehmigung" und "Zulassen". Dies ist erforderlich, damit die Pipeline die Ressource erstellen kann.

    > Die Bereitstellung kann einige Minuten dauern. Die CD-Definition besteht aus den folgenden Aufgaben:
    - **Ressourcen** : Es ist bereit, basierend auf dem Abschluss der CI-Pipeline automatisch auszulösen. Außerdem wird das Repository für die Bicep-Datei heruntergeladen.
    - **Variablen (für die Bereitstellungsphase)** stellen eine Verbindung mit der Variablengruppe zur Nutzung des geheimen Azure Key Vault-Schlüssels "acr-secret **" bereit.**
    - **AzureResourceManagerTemplateDeployment** stellt die Azure Container Instance (ACI) mithilfe der Bicep-Vorlage bereit und stellt die ACR-Anmeldeparameter bereit, damit ACI das zuvor erstellte Containerimage aus der Azure Container Registry (ACR) herunterladen kann.

7. Ihre Pipeline nimmt einen Namen auf der Grundlage des Projektnamens an. Hiermit können Sie **sie umbenennen** , um die Pipeline besser zu identifizieren. Wechseln Sie zu **Pipelines>Pipelines** , und klicken Sie auf die kürzlich erstellte Pipeline. Klicken Sie auf die Auslassungspunkte und **die Option "Umbenennen/Entfernen"** . Nennen Sie es **eshoponweb-cd-aci** , und klicken Sie auf **"Speichern"**.

### Übung 2: Entfernen der Azure-Lab-Ressourcen.

In dieser Übung entfernen Sie die in dieser Übung bereitgestellten Azure-Ressourcen, um unerwartete Gebühren zu beseitigen.

>**Hinweis**: Denken Sie daran, alle neu erstellten Azure-Ressourcen zu entfernen, die Sie nicht mehr verwenden. Durch das Entfernen nicht verwendeter Ressourcen wird sichergestellt, dass keine unerwarteten Kosten anfallen.

#### Übung 4: Entfernen der Azure Lab-Ressourcen

In dieser Aufgabe verwenden Sie Azure Cloud Shell, um die in dieser Übung bereitgestellten Azure-Ressourcen zu entfernen, um unnötige Gebühren zu vermeiden.

1. Navigieren Sie im Azure-Portal zu Ihrer Ressourcengruppe, und klicken Sie auf **Ressourcengruppe löschen**.

## Überprüfung

In diesem Lab werden Sie sehen, wie Sie Azure Key Vault mit Hilfe der folgenden Schritte in eine Azure-Pipeline integrieren können:

- Es wurde ein Azure-Dienstprinzipal erstellt, um Zugriff auf einen geheimen Azure Key Vault-Schlüssel bereitzustellen und die Bereitstellung von Azure DevOps bei Azure zu authentifizieren.
- Es wurden zwei YAML-Pipelines ausgeführt, die aus einem Git-Repository importiert wurden.
- Konfigurierte eine Pipeline, um das Kennwort aus Azure Key Vault mithilfe einer Variablengruppe abzurufen und für nachfolgende Aufgaben zu verwenden.
