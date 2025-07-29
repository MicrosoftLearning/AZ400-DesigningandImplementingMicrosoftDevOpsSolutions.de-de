---
lab:
  title: Integrieren von Azure Key Vault in Azure DevOps
  module: 'Module 04: Implement a secure continuous deployment using Azure Pipelines'
---

# Integrieren von Azure Key Vault in Azure DevOps

## Labanforderungen

- Für dieses Lab ist **Microsoft Edge** oder ein von [Azure DevOps unterstützter Browser](https://learn.microsoft.com/azure/devops/server/compatibility) erforderlich.

- **Führen Sie die Überprüfung der Labumgebung aus:** Stellen Sie vor dem Starten dieses Labs sicher, dass Sie [Überprüfen der Labumgebung](AZ400_M00_Validate_lab_environment.md) abgeschlossen haben. In diesem Schritt wurden die Azure DevOps-Organisation, das Projekt und die Dienstverbindung eingerichtet, die für dieses Lab erforderlich sind.

- **Einrichten einer Azure DevOps-Organisation**: Wenn Sie nicht bereits eine Azure DevOps-Organisation haben, die Sie für dieses Lab verwenden können, müssen Sie diese erstellen, indem Sie die unter [Erstellen einer Organisation oder Projektsammlung](https://learn.microsoft.com/azure/devops/organizations/accounts/create-organization) beschriebenen Anweisungen befolgen.
- Identifizieren Sie ein vorhandenes Azure-Abonnement, oder erstellen Sie ein neues Abonnement.

## Übersicht über das Labor

Azure Key Vault bietet eine sichere Speicherung und Verwaltung von vertraulichen Daten, wie z. B. Schlüsseln, Passwörtern und Zertifikaten. Azure Key Vault unterstützt sowohl Hardware-Sicherheitsmodule als auch eine Reihe von Verschlüsselungsalgorithmen und Schlüssellängen. Durch die Verwendung von Azure Key Vault können Sie die Möglichkeit der Offenlegung sensibler Daten über den Quellcode minimieren, was ein häufiger Fehler von Entwicklern ist. Der Zugriff auf Azure Key Vault erfordert eine ordnungsgemäße Authentifizierung und Autorisierung und unterstützt fein abgestufte Berechtigungen für den Inhalt.

In dieser Übung sehen Sie, wie Sie Azure Key Vault mit Azure Pipelines integrieren können, indem Sie die folgenden Schritte durchführen:

- Erstellen Sie einen Azure Key Vault, um ein ACR-Kennwort als Geheimnis zu speichern.
- Ermöglichen Sie den Zugriff auf Geheimnisse im Azure Key Vault.
- Konfigurieren Sie die Berechtigungen zum Lesen des Geheimnisses.
- Konfigurieren Sie die Pipeline, um das Kennwort aus dem Azure Key Vault abzurufen und an nachfolgende Aufgaben weiterzugeben.

## Ziele

In diesem Lab lernen Sie Folgendes:

- Erstellen Sie eine Azure Key Vault-Instanz.
- Abrufen eines Geheimnisses aus Azure Key Vault in einer Azure DevOps Pipeline.
- Verwenden Sie das Geheimnis in einer nachfolgenden Aufgabe in der Pipeline.
- Stellen Sie ein Containerimage auf Azure Containerinstanz (ACI) bereit, indem Sie das Geheimnis verwenden.

## Geschätzte Zeit: 40 Minuten

## Anweisungen

### Übung 0: (Überspringen, wenn bereits abgeschlossen) Konfigurieren der Lab-Voraussetzungen

In dieser Übung richten Sie die Voraussetzungen für das Lab ein, das aus einem neuen Azure DevOps-Projekt mit einem Repository basierend auf dem [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb) besteht.

#### Aufgabe 1: (überspringen, wenn fertig) Erstellen und Konfigurieren des Teamprojekts

In dieser Aufgabe erstellen Sie ein **eShopOnWeb** Azure DevOps-Projekt, das von mehreren Labs verwendet werden soll.

1. Öffnen Sie auf Ihrem Lab-Computer in einem Browserfenster Ihre Azure DevOps-Organisation. Klicken Sie auf **Neues Projekt**. Weisen Sie Ihrem Projekt den Namen **eShopOnWeb** zu, und lassen Sie die anderen Felder auf den Standardwerten. Klicken Sie auf **Erstellen**.

    ![Screenshot des Bereichs „Neues Projekt erstellen“.](images/create-project.png)

#### Aufgabe 2: (überspringen, wenn erledigt) Importieren von eShopOnWeb Git Repository

Bei dieser Aufgabe importieren Sie das eShopOnWeb Git-Repository, das von mehreren Labs verwendet wird.

1. Öffnen Sie auf Ihrem Lab-Computer in einem Browserfenster Ihre Azure DevOps-Organisation und das zuvor erstellte **eShopOnWeb**-Projekt. Klicken Sie auf **Repos > Files**, **Importieren**. Fügen Sie im Fenster **Git Repository importieren** die folgende URL <https://github.com/MicrosoftLearning/eShopOnWeb.git> ein und klicken Sie auf **Importieren**:

    ![Screenshot der Schaltfläche „Repository importieren“](images/import-repo.png)

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

### Übung 1: Einrichten einer CI-Pipeline zur Erstellung des eShopOnWeb-Containers

In dieser Übung erstellen Sie eine CI-Pipeline, die die eShopOnWeb-Containerimages erstellt und in eine Azure Container Registry (ACR) überträgt. Die Pipeline verwendet Docker Compose, um die Images zu erstellen und sie an das ACR zu übertragen.

#### Aufgabe 1: Einrichten und Ausführen der CI-Pipeline

In dieser Aufgabe importieren Sie eine bestehende CI YAML-Pipeline-Definition, ändern sie und führen sie aus. Es wird eine neue Azure Container Registry (ACR) erstellt und die eShopOnWeb Container-Images erstellt/veröffentlicht.

1. Starten Sie auf dem Laborcomputer einen Webbrowser und navigieren Sie zum Azure DevOps **eShopOnWeb**-Projekt. Gehen Sie zu **Pipelines > Pipelines** und klicken Sie auf **Pipeline erstellen** (oder **Neue Pipeline**).

1. Im Fenster **Wo befindet Sich Ihr Code?** wählen Sie **Azure Repos Git (YAML)** aus. Wählen Sie dann das **eShopOnWeb-Repository** aus.

1. Wählen Sie auf der Registerkarte **Konfigurieren** die Option **Vorhandene Azure Pipelines YAML-Datei** aus. Verzweigung auswählen: **Haupt**, geben Sie den Pfad **/.ado/eshoponweb-ci-dockercompose.yml** an, und klicken Sie auf **Weiter**.

    ![Screenshot der bestehenden Pipeline-YAML.](images/select-ci-container-compose.png)

1. Passen Sie in der YAML-Pipelinedefinition Ihren Ressourcengruppennamen an, indem Sie **NAME** in **AZ400-EWebShop-NAME** durch einen eindeutigen Wert ersetzen und **YOUR-SUBSCRIPTION-ID** durch Ihre eigene Azure subscriptionId ersetzen.

1. Klicken Sie auf **Speichern und Ausführen**, und warten Sie, bis die Pipeline erfolgreich ausgeführt wird. Möglicherweise müssen Sie ein zweites Mal auf **Speichern und ausführen** klicken, um den Prozess zur Pipelineerstellung und -ausführung abzuschließen.

    > **Wichtig**: Wenn Sie die Meldung „Diese Pipeline benötigt eine Berechtigung für den Zugriff auf Ressourcen, bevor dieser Lauf mit Docker Compose to ACI fortgesetzt werden kann“ sehen, klicken Sie auf „Anzeigen“, „Genehmigen“ und „Zulassen“. Dies ist erforderlich, damit die Pipeline die Ressource erstellen kann. Sie müssen auf den Buildauftrag klicken, um die Berechtigungsmeldung anzuzeigen.

    > **Hinweis**: Es kann einige Minuten dauern, bis der Build abgeschlossen ist. Die Buildpipeline besteht aus den folgenden Aufgaben:
    - **AzureResourceManagerTemplateDeployment** verwendet **bicep** zur Bereitstellung einer Azure Container Registry.
    - **PowerShell** nimmt die Bicep-Ausgabe (acr login server) und erstellt eine Pipeline-Variable.
    - **DockerCompose** erstellt und überträgt die Container-Images für eShopOnWeb in die Azure Container Registry.

1. Ihre Pipeline bekommt einen Namen basierend auf dem Projektnamen. Lassen Sie sie uns **umbenennen**, um die Pipeline besser zu identifizieren. Gehen Sie zu **Pipelines > Pipelines** und klicken Sie auf die kürzlich erstellte Pipeline. Klicken Sie auf die Auslassungspunkte und die Option **Umbenennen/Entfernen**. Benennen Sie sie**`eshoponweb-ci-dockercompose`**, und klicken Sie auf **Speichern**.

1. Sobald die Ausführung abgeschlossen ist, öffnen Sie im Azure Portal die zuvor definierte Ressourcengruppe und Sie sollten eine Azure Container Registry (ACR) mit den erstellten Container-Images **eshoppublicapi** und **eshopwebmvc** finden. Sie werden **eshopwebmvc** nur in der Bereitstellungsphase verwenden.

    ![Screenshot von Containerimages in ACR.](images/azure-container-registry.png)

1. Klicken Sie auf **Zugriffsschlüssel**, aktivieren Sie **Administratorbenutzer**, falls noch nicht geschehen, und kopieren Sie den Wert von **password**. In der folgenden Aufgabe speichern wir ihn im Azure Key Vault als Geheimnis.

    ![Screenshot des ACR-Kennwortspeicherorts.](images/acr-password.png)

#### Aufgabe 2: Erstellen eines Azure Key Vault

In dieser Aufgabe erstellen Sie einen Azure Key Vault über das Azure-Portal.

In diesem Übungsszenario wird eine Azure Container Instance (ACI) verwendet, die ein in Azure Container Registry (ACR) gespeichertes Container-Image abruft und ausführt. Wir beabsichtigen, das Kennwort für den ACR als Geheimnis im Key Vault zu speichern.

1. Geben Sie im Azure-Portal in das Textfeld **Ressourcen, Dienste und Dokumente durchsuchen** **`Key vault`** ein und drücken Sie die **Eingabetaste**.
1. Wählen Sie das Blatt **Schlüsseltresor** aus, und klicken Sie auf **Erstellen > Schlüsseltresor**.
1. Geben Sie auf der Registerkarte **Grundlagen** des Blattes **Key Vault erstellen** die folgenden Einstellungen an und klicken Sie auf **Weiter**:

    | Einstellung | Wert |
    | --- | --- |
    | Subscription | Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden. |
    | Resource group | Der Name einer neuen Ressourcengruppe **AZ400-EWebShop-NAME** |
    | Name des Schlüsseltresors | Beliebiger eindeutiger gültiger Name, z. B. **ewebshop-kv-NAME** (NAME ersetzen) |
    | Region | Eine Azure-Region in der Nähe des Standorts Ihrer Lab-Umgebung |
    | Tarif | **Standard** |
    | Aufbewahrungsdauer für gelöschte Tresore in Tagen | **7** |
    | Bereinigungsschutz | **Bereinigungsschutz deaktivieren** |

1. Wählen Sie auf der Registerkarte **Zugangskonfiguration** des Blades **Key Vault erstellen** die Option **Tresorzugriffsrichtlinie** und klicken Sie dann im Abschnitt **Zugangsrichtlinien** auf **+ Erstellen**, um eine neue Richtlinie einzurichten.

    > **Hinweis**: Sie müssen den Zugriff auf Ihre Key Vaults sichern, indem Sie nur autorisierte Anwendungen und Benutzer*innen zulassen. Um auf die Daten aus dem Tresor zuzugreifen, müssen Sie der Dienstverbindung, die Sie bei der Labumgebungsüberprüfung für die Authentifizierung in der Pipeline erstellt haben, Leseberechtigungen (Get/List) erteilen.

    1. Überprüfen Sie auf dem Blatt **Berechtigung** unter **Geheime Berechtigungen** die Berechtigungen **Abrufen** und **Auflisten**. Klicken Sie auf **Weiter**.
    2. Suchen Sie auf dem Blatt **Prinzipal** nach der **Dienstverbindung des Azure-Abonnements** (die während der Überprüfung der Labumgebung erstellt wurde, in der Regel „azure subs“), und wählen Sie sie in der Liste aus. Sie finden den Dienstprinzipalnamen in Azure DevOps unter „Projekteinstellungen“ > „Dienstverbindungen“ > Azure-Abonnements > „Dienstprinzipal verwalten“. Wenn beim Auswählen des Azure-Abonnements ein Berechtigungsfehler auftritt, klicken Sie auf die Schaltfläche **Autorisieren**, die automatisch die Zugriffsrichtlinie für Sie im Schlüsseltresor erstellt. Klicken Sie auf **Weiter**, **Weiter**, **Erstellen** (Zugangsrichtlinie).
    3. Klicken Sie auf dem Blatt **Überprüfen + Erstellen** auf **Erstellen**

1. Zurück auf dem Blatt **Key Vault erstellen** klicken Sie auf **Überprüfen + Erstellen > Erstellen**

    > **Hinweis**: Warten Sie, bis der Azure Key Vault bereitgestellt wurde. Das sollte weniger als eine Minute dauern.

1. Klicken Sie auf dem Blatt **Ihre Bereitstellung ist abgeschlossen** auf **Zur Ressource**.
1. Klicken Sie auf dem Blatt Azure Key Vault (ewebshop-kv-NAME) im vertikalen Menü auf der linken Seite des Blatts im Abschnitt **Objekte** auf **Geheimnisse**.
1. Klicken Sie auf dem Blatt **Geheimnisse** auf **Erzeugen/Importieren**.
1. Geben Sie auf dem Blatt **Geheimnis erstellen** die folgenden Einstellungen an und klicken Sie auf **Erstellen** (belassen Sie die anderen bei ihren Standardwerten):

    | Einstellung | Wert |
    | --- | --- |
    | Uploadoptionen | **Manuell** |
    | Name | **acr-secret** |
    | Geheimniswert | ACR-Zugriffskennwort, das in der vorherigen Aufgabe kopiert wurde |

#### Aufgabe 3: Erstellen einer mit Azure Key Vault verbundenen Variablengruppe

In dieser Aufgabe erstellen Sie eine variable Gruppe in Azure DevOps, die das ACR-Kennwortgeheimnis aus Key Vault über die zuvor erstellte Dienstverbindung abruft.

1. Starten Sie auf Ihrem Lab-Computer einen Webbrowser und navigieren Sie zu dem Azure DevOps-Projekt **eShopOnWeb**.

1. Wählen Sie im vertikalen Navigationsbereich des Azure DevOps-Portals **Pipelines > Bibliothek**aus. Klicken Sie auf **+ Variablengruppe**.

1. Geben Sie auf dem Blatt **Neue Variablengruppe** die folgenden Einstellungen an:

    | Einstellung | Wert |
    | --- | --- |
    | Variablengruppenname | **eshopweb-vg** |
    | Verknüpfung von Geheimnissen aus einem Azure Key Vault | **enable** |
    | Azure-Abonnement | **Verfügbare Azure-Dienstverbindung > Azure subs** |
    | Name des Schlüsseltresors | Der Name Ihres Schlüsseltresors|

1. Klicken Sie unter **Variablen** auf **+ Hinzufügen** und wählen Sie das Geheimnis **acr-secret** aus. Klicken Sie auf **OK**.
1. Klicken Sie auf **Speichern**.

    ![Screenshot der Variablengruppenerstellung](images/vg-create.png)

#### Aufgabe 4: Einrichten der CD-Pipeline zum Bereitstellen von Containern in Azure Container Instance (ACI)

In dieser Aufgabe importieren Sie eine CD-Pipeline, passen sie an und führen sie aus, um das zuvor in einer Azure-Containerinstanz erstellte Containerimage bereitzustellen.

1. Starten Sie auf dem Laborcomputer einen Webbrowser und navigieren Sie zum Azure DevOps **eShopOnWeb**-Projekt. Gehen Sie zu **Pipelines > Pipelines** und klicken Sie auf **Neue Pipeline**.

1. Im Fenster **Wo befindet Sich Ihr Code?** wählen Sie **Azure Repos Git (YAML)** aus. Wählen Sie dann das **eShopOnWeb-Repository** aus.

1. Wählen Sie auf der Registerkarte **Konfigurieren** die Option **Vorhandene Azure Pipelines YAML-Datei** aus. Verzweigung auswählen: **Haupt**, geben Sie den Pfad **/.ado/eshoponweb-cd-aci.yml** an, und klicken Sie auf **Weiter**.

1. Passen Sie in der YAML-Pipelinedefinition Folgendes an:

    - Ersetzen Sie **IHRE-ABONNEMENT-ID** durch Ihre Azure-Abonnement-ID.
    - **az400eshop-NAME** ersetzen Sie NAME, um ihn global eindeutig zu machen.
    - **YOUR-ACR.azurecr.io** und **ACR-USERNAME** mit Ihrem ACR-Login-Server (beide benötigen den ACR-Namen, kann unter ACR > Zugriffstasten eingesehen werden).
    - **AZ400-EWebShop-NAME** mit dem zuvor im Labor definierten Namen der Ressourcengruppe.

1. Klicken Sie auf **Speichern und Ausführen**. Möglicherweise müssen Sie ein zweites Mal auf **Speichern und ausführen** klicken, um den Prozess zur Pipelineerstellung und -ausführung abzuschließen. Sie müssen auf den Buildauftrag klicken, um Berechtigungsmeldungen anzuzeigen.
1. Öffnen Sie die Pipeline, und warten Sie, bis sie erfolgreich ausgeführt wird.

    > **Wichtig**: Wenn Sie die Meldung „Diese Pipeline benötigt eine Berechtigung für den Zugriff auf Ressourcen, bevor dieser Lauf mit Docker Compose to ACI fortgesetzt werden kann“ sehen, klicken Sie auf „Anzeigen“, „Genehmigen“ und „Zulassen“. Dies ist erforderlich, damit die Pipeline die Ressource erstellen kann.

    > **Hinweis**: Die Bereitstellung kann einige Minuten dauern. Die CD-Definition besteht aus den folgenden Aufgaben:
    - **Ressourcen**: es ist darauf vorbereitet, automatisch nach Abschluss der CI-Pipeline ausgelöst zu werden. Außerdem wird das Repository für die Bicep-Datei heruntergeladen.
    - **Variablen (für die Bereitstellungsphase)** stellt eine Verbindung zur Variablengruppe her, um das Azure Key Vault-Geheimnis zu verwenden **acr-secret**.
    - **AzureResourceManagerTemplateDeployment** stellt die Azure Container Instance (ACI) unter Verwendung der Bicep-Vorlage bereit und stellt die ACR-Anmeldeparameter bereit, damit ACI das zuvor erstellte Container-Image von Azure Container Registry (ACR) herunterladen kann.

1. Um die Ergebnisse der Pipelinebereitstellung zu überprüfen, suchen Sie im Azure-Portal die Ressourcengruppe **AZ400-EWebShop-NAME**, und wählen Sie sie aus. Überprüfen Sie in der Liste der Ressourcen, ob die Containerinstanz **az400eshop** von der Pipeline erstellt wurde.

1. Ihre Pipeline bekommt einen Namen basierend auf dem Projektnamen. Lassen Sie sie uns **umbenennen**, um die Pipeline besser zu identifizieren. Gehen Sie zu **Pipelines > Pipelines** und klicken Sie auf die kürzlich erstellte Pipeline. Klicken Sie auf die Auslassungspunkte und die Option **Umbenennen/Entfernen**. Nennen Sie es **eshoponweb-cd-aci** und klicken Sie auf **Speichern**.

   > [!IMPORTANT]
   > Denken Sie daran, die im Azure-Portal erstellten Ressourcen zu löschen, um unnötige Gebühren zu vermeiden.

## Überprüfung

In dieser Übung haben Sie Azure Key Vault mit einer Azure DevOps-Pipeline integriert, indem Sie die folgenden Schritte durchgeführt haben:

- Erstellen Sie einen Azure Key Vault, um ein ACR-Kennwort als Geheimnis zu speichern.
- Zugriff auf Geheimnisse im Azure Key Vault.
- Konfigurierte Berechtigungen zum Lesen des geheimen Schlüssels.
- Konfigurieren Sie eine Pipeline, um das Kennwort aus dem Azure Key Vault abzurufen und es an nachfolgende Aufgaben weiterzugeben.
- Bereitstellen eines Containerimages auf Azure Containerinstanz (ACI) unter Verwendung des Geheimnisses.
- Erstellen Sie eine Variable Gruppe, die mit Azure Key Vault verbunden ist.
