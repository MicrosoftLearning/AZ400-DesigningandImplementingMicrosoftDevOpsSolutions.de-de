---
lab:
  title: Aktivieren von Featureflags und der dynamischen Konfiguration
  module: 'Module 05: Implement a secure continuous deployment using Azure Pipelines'
---

# Aktivieren von Featureflags und der dynamischen Konfiguration

## Lab-Handbuch für Kursteilnehmer

## Labanforderungen

- Für dieses Lab ist **Microsoft Edge** oder ein [von Azure DevOps unterstützter Browser](https://learn.microsoft.com/azure/devops/server/compatibility) erforderlich.

- **Einrichten einer Azure DevOps-Organisation**: Wenn Sie nicht bereits eine Azure DevOps-Organisation haben, die Sie für dieses Lab verwenden können, müssen Sie diese erstellen, indem Sie die unter [Erstellen einer Organisation oder Projektsammlung](https://learn.microsoft.com/azure/devops/organizations/accounts/create-organization?view=azure-devops) beschriebenen Anweisungen befolgen.

- Identifizieren Sie ein vorhandenes Azure-Abonnement, oder erstellen Sie ein neues Abonnement.

- Vergewissern Sie sich, dass Sie über ein Microsoft- oder ein Microsoft Entra-Konto mit der Rolle „Mitwirkender“ oder „Besitzer“ im Azure-Abonnement verfügen. Ausführliche Informationen finden Sie in den Artikeln zum [Auflisten von Azure-Rollenzuweisungen mithilfe des Azure-Portals](https://learn.microsoft.com/azure/role-based-access-control/role-assignments-list-portal) und [Anzeigen und Zuweisen von Administratorrollen in Azure Active Directory](https://learn.microsoft.com/azure/active-directory/roles/manage-roles-portal).

## Übersicht über das Labor

[Azure App Configuration](https://learn.microsoft.com/azure/azure-app-configuration/overview) bietet einen Dienst zur zentralen Verwaltung von Anwendungseinstellungen und Featureflags. Moderne Programme, vor allem in einer Cloud, verfügen über viele verteilte Komponenten. Für alle diese Komponenten Konfigurationseinstellungen festzulegen, kann während der Anwendungsbereitstellung zu Fehlern führen, die sich nur schwer beheben lassen. Mit App Configuration können Sie alle Einstellungen für Ihre Anwendung speichern und den Zugriff darauf an einem zentralen Ort schützen.

## Ziele

In diesem Lab lernen Sie Folgendes:

- Aktivieren der dynamischen Konfiguration
- Verwalten von Featureflags

## Geschätzte Zeit: 60 Minuten

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

### Übung 1: Importieren und Ausführen von CI/CD-Pipelines

In dieser Übung importieren und ausführen Sie die CI-Pipeline, konfigurieren die Dienstverbindung mit Ihrem Azure-Abonnement und importieren und führen dann die CD-Pipeline aus.

#### Übung 2: Importieren und Ausführen der CI-Pipeline

Beginnen wir mit dem Importieren der CI-Pipeline mit dem Namen [eshoponweb-ci.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-ci.yml).

1. Navigieren Sie in Azure Pipelines zu PipelinesPipelines.
2. Klicken Sie auf die **Schaltfläche "Pipeline** erstellen" (wenn keine Pipelines vorhanden sind) oder **auf die Schaltfläche "Neue Pipeline** " (sofern bereits Pipelines erstellt wurden).
3. Wählen Sie **Azure Repos Git** (YAML) aus.
4. Wählen Sie das **eShopOnWeb-Repository** aus.
5. Wählen Sie die Option **Vorhandene Azure Pipelines-YAML-Datei** aus.
6. Wählen Sie die **Datei "/.ado/eshoponweb-ci.yml** " aus, und klicken Sie dann auf **"Weiter"**.
7. Klicken Sie auf die Schaltfläche **Ausführen**, um die Pipeline auszuführen.
8. Ihre Pipeline nimmt einen Namen auf der Grundlage des Projektnamens an. **Benennen** wir sie um, um die Pipeline besser zu identifizieren. Wechseln Sie zu **Pipelines>Pipelines** , und klicken Sie auf die kürzlich erstellte Pipeline. Klicken Sie auf die Auslassungspunkte und **die Option "Umbenennen/Entfernen"** . Nennen Sie es **eshoponweb-ci** , und klicken Sie auf **"Speichern"**.

#### Wählen Sie auf der Taskleiste Verbindungen verwalten:

Sie können eine Verbindung zwischen Azure Pipelines und externen Diensten und Remotediensten herstellen, um Aufgaben in einem Auftrag auszuführen.

In dieser Aufgabe erstellen Sie einen Dienstprinzipal mithilfe der Azure CLI, der Azure DevOps zulässt:

- Bereitstellen von Ressourcen in einem Azure-Abonnement
- Bereitstellen der eShopOnWeb-Anwendung

> **Hinweis**: Wenn Sie bereits über einen Dienstprinzipal verfügen, können Sie direkt mit der nächsten Aufgabe fortfahren.

Sie benötigen einen Dienstprinzipal, um Azure-Ressourcen aus Azure-Pipelines bereitzustellen.

Ein Dienstprinzipal wird automatisch von Azure Pipeline erstellt, wenn Sie eine Verbindung mit einem Azure-Abonnement innerhalb einer Pipelinedefinition herstellen oder wenn Sie eine neue Dienstverbindung über die Seite mit den Projekteinstellungen (automatische Option) erstellen. Sie können den Dienstprinzipal auch manuell über das Portal erstellen oder Azure CLI verwenden und es für alle Projekte wiederverwenden.

1. Starten Sie auf Ihrem Labcomputer einen Webbrowser, navigieren Sie zum [**Azure-Portal**](https://portal.azure.com), und melden Sie sich an. Verwenden Sie hierzu die Anmeldeinformationen eines Benutzerkontos, das in dem Abonnement, das Sie in diesem Lab verwenden, über die Rolle „Besitzer“ und in dem Azure AD-Mandanten, der dem Azure-Abonnement zugeordnet ist, über die Rolle „Globaler Administrator“ verfügt.
2. Öffnen Sie im Azure-Portal den Bereich **Cloud Shell**, indem Sie auf das Symbolleistensymbol rechts neben dem Textfeld für die Suche klicken.
3. Wählen Sie bei Aufforderung zur Auswahl von **Bash** oder **PowerShell** die Option **Bash** aus.

   >**Hinweis**: Wenn Sie **Cloud Shell** zum ersten Mal starten und die Meldung **Für Sie wurde kein Speicher bereitgestellt** angezeigt wird, wählen Sie das in diesem Lab verwendete Abonnement aus, und klicken Sie dann auf **Speicher erstellen**.

4. Führen Sie in der **Bash-Eingabeaufforderung** im **Cloud Shell-Bereich** die folgenden Befehle aus, um die Werte des Azure-Abonnement-ID-Attributs abzurufen:

    ```sh
    subscriptionName=$(az account show --query name --output tsv)
    subscriptionId=$(az account show --query id --output tsv)
    echo $subscriptionName
    echo $subscriptionId
    ```

    > **Hinweis**: Kopieren Sie beide Werte in eine Textdatei. Sie werden sie später in diesem Lab benötigen.

5. Führen Sie in der **Bash-Eingabeaufforderung** im **Cloud Shell-Bereich** den folgenden Befehl aus, um einen Dienstprinzipal zu erstellen:

    ```sh
    az ad sp create-for-rbac --name sp-az400-azdo --role contributor --scopes /subscriptions/$subscriptionId
    ```

    > **Hinweis**: Der Befehl generiert eine JSON-Ausgabe. Speichern der Ausgabe in einer Textdatei Sie benötigen ihn später in diesem Lab.

6. Starten Sie als Nächstes auf dem Laborcomputer einen Webbrowser, navigieren Sie zum Azure DevOps **eShopOnWeb-Projekt** . Klicken Sie auf **Project Einstellungen>Service-Verbinden ionen (unter Pipelines)** und **Verbinden ion** neuer Dienst.

7. Wählen Sie im Bildschirm **Neue Dienstverbindung** die Option **Azure Resource Manager** und anschließend **Weiter** aus.

8. Wählen Sie **den Dienstprinzipal (manuell)** aus, und klicken Sie auf **"Weiter"**.

9. Füllen Sie die leeren Felder mit den informationen aus, die während der vorherigen Schritte gesammelt wurden:
    - Abonnement-ID oder -Name.
    - Dienstprinzipal-ID (oder ClientId), Schlüssel (oder Kennwort) und TenantId.
    - Geben Sie unter **"Dienstverbindungsname****" Azure-Untere ein**. Auf diesen Namen wird in YAML-Pipelines verwiesen, wenn ein Azure DevOps Service Verbinden ion erforderlich ist, um mit Ihrem Azure-Abonnement zu kommunizieren.

10. Klicken Sie auf **Überprüfen und speichern**.

#### Übung 3: Importieren und Ausführen der CD-Pipeline

Importieren wir nun die CD-Pipeline mit dem Namen [eshoponweb-cd-webapp-code.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-cd-webapp-code.yml).

1. Navigieren Sie in Azure Pipelines zu PipelinesPipelines.
2. Klicken Sie auf die Schaltfläche Neue Pipeline**.
3. Wählen Sie **Azure Repos Git** (YAML) aus.
4. Wählen Sie das **eShopOnWeb-Repository** aus.
5. Wählen Sie die Option **Vorhandene Azure Pipelines-YAML-Datei** aus.
6. Wählen Sie die **Datei "/.ado/eshoponweb-cd-webapp-code.yml** " aus, und klicken Sie dann auf **"Weiter"**.
7. Passen Sie in der YAML-Pipelinedefinition Folgendes an:
   - Ersetzen Sie <yourSubscriptionId> durch Ihre Azure-Abonnement-ID.
   - **az400eshop-NAME** ersetzt NAME, um ihn global eindeutig zu machen.
   - **AZ400-EWebShop-NAME** mit dem zuvor im Labor definierten Ressourcengruppennamen.

8. Klicken Sie auf " **Speichern und Ausführen** ", und warten Sie, bis die Pipeline erfolgreich ausgeführt wird.

    > Die Bereitstellung kann einige Minuten dauern.

    Die CD-Definition besteht aus den folgenden Aufgaben:
    - **Ressourcen**: Sie ist bereit, basierend auf dem Abschluss der CI-Pipeline automatisch auszulösen. Außerdem wird das Repository für die Bicep-Datei heruntergeladen.
    - **AzureResourceManagerTemplateDeployment**: Stellt die Azure Web App mithilfe der Bicep-Vorlage bereit.

9. Ihre Pipeline nimmt einen Namen auf der Grundlage des Projektnamens an. **Benennen** wir sie um, um die Pipeline besser zu identifizieren. Wechseln Sie zu **Pipelines>Pipelines** , und klicken Sie auf die kürzlich erstellte Pipeline. Klicken Sie auf die Auslassungspunkte und **die Option "Umbenennen/Entfernen"** . Nennen Sie ihn **eshoponweb-cd-webapp-code**, und klicken Sie auf "Speichern"****.

### Übung 2: Verwalten von Azure App Configuration

In dieser Übung erstellen Sie die App-Konfigurationsressource in Azure, aktivieren die verwaltete Identität und testen dann die vollständige Lösung.

> **Hinweis**: Für diese Übung sind keine Codierungskenntnisse erforderlich. Der Code der Website implementiert bereits Azure-App Konfigurationsfunktionen.

Wenn Sie wissen möchten, wie Sie dies in Ihrer Anwendung implementieren können, sehen Sie sich die folgenden Lernprogramme an: [Verwenden Sie die dynamische Konfiguration in einer ASP.NET Core-App](https://learn.microsoft.com/azure/azure-app-configuration/enable-dynamic-configuration-aspnet-core) und [Verwalten von Featurekennzeichnungen in Azure-App-Konfiguration](https://learn.microsoft.com/azure/azure-app-configuration/manage-feature-flags).

#### Aufgabe 1: Erstellen der Contoso-Ressourcengruppe

1. Suchen Sie auf der Registerkarte Aufgaben nach der Aufgabe Azure App Configuration.
2. Wählen Sie eine Konfiguration aus, und klicken Sie auf **Entfernen**.
    - Ihr Azure-Abonnement.
    - Die zuvor erstellte Ressourcengruppe (sie sollte AZ400-EWebShop-NAME **** genannt werden).
    - Der Speicherort.
    - Ein eindeutiger Name wie **"appcs-NAME-REGION** ".
    - Wählen Sie die **Preisstufe**"Free".
3. Klicken Sie auf **Überprüfen + erstellen** und danach auf **Erstellen**.
4. Wechseln Sie nach dem Erstellen des App-Konfigurationsdiensts zu **"Übersicht"** , und kopieren/speichern Sie den Wert des **Endpunkts**.

#### Aktivieren einer verwalteten Identität für eine Aufgabe

1. Wechseln Sie mit der Pipeline zu der Web App, die mit der Pipeline bereitgestellt wird (sie sollte az400-webapp-NAME **genannt** werden).
2. Klicken Sie im **Abschnitt Einstellungen** auf **"Identität**", und wechseln Sie dann im **Abschnitt "System zugewiesen**" auf **"Ein**", klicken Sie auf "Speichern" > "Ja **", und warten Sie **einige Sekunden, bis der Vorgang abgeschlossen ist.
3. Wechseln Sie zurück zum App-Konfigurationsdienst, und klicken Sie auf access-Steuerelement****, und fügen Sie dann **die Rollenzuweisung** hinzu.
4. Wählen Sie unter **Rolle** die Option **App Configuration-Datenleser** aus.
5. Aktivieren Sie im **Abschnitt "Mitglieder** " die Option **"Identität** verwalten", und wählen Sie dann die verwaltete Identität Ihrer Web App aus (sie sollten denselben Namen haben).
6. Klicken Sie auf " **Überprüfen und Zuweisen"**.

#### Aufgabe 5: Konfigurieren der Web-App

Um sicherzustellen, dass Ihre Website auf die App-Konfiguration zugreift, müssen Sie die Konfiguration aktualisieren.

1. Wechseln Sie zu Ihrer Web-App.
2. Klicken Sie im Abschnitt **Einstellungen** auf **Konfigurieren**.
3. Neue Anwendungseinstellung hinzufügen
    - Erste App-Einstellung
        - **Name:** UseAppConfig
        - **Wert: true**
    - Wichtigste App-Einstellungen
        - **Name:** AppConfigEndpoint
        - **Wert: Der Wert,** *den Sie zuvor vom App-Konfigurationsendpunkt gespeichert/kopiert haben. Es sollte wie folgt aussehen: https://appcs-NAME-REGION.azconfig.io*

4. Klicken Sie auf **** "OK **", und warten Sie**, bis die Einstellungen aktualisiert werden.
5. Wechseln Sie zur **Übersicht** , und klicken Sie auf " **Durchsuchen".**
6. In diesem Schritt werden keine Änderungen auf der Website angezeigt, da die App-Konfiguration keine Daten enthält. Dies ist das, was Sie in den nächsten Aufgaben tun werden.

#### Aufgabe 4: Testen der Konfigurationsverwaltung

1. Wählen Sie **in Ihrer Website Visual Studio** in der **Dropdownliste "Marke**" aus, und klicken Sie auf die Pfeilschaltfläche (>****).
2. Es wird eine Meldung mit der *Meldung "ES SIND KEINE ERGEBNISSE VORHANDEN, DIE IHRER SUCHE ENTSPRECHEN"*. Ziel dieser Übung ist es, diesen Wert zu aktualisieren, ohne den Code der Website zu aktualisieren oder ihn erneut bereitzustellen.
3. Um dies zu versuchen, wechseln Sie zurück zur App-Konfiguration.
4. Wählen Sie im Abschnitt **Vorgänge** die Option **Konfigurations-Explorer** aus.
5. Klicken Sie auf **"> Schlüsselwert** erstellen", und fügen Sie dann Folgendes hinzu:
    - **Schlüssel:** eShopWeb:Einstellungen:NoResultsMessage
    - **Wert:** *Geben Sie Ihre benutzerdefinierte Nachricht ein.*
6. Klicken Sie auf "Übernehmen"**, und wechseln Sie **zurück zu Ihrer Website, und aktualisieren Sie die Seite.
7. Die neue Nachricht sollte anstelle des alten Standardwerts angezeigt werden.

Herzlichen Glückwunsch! In dieser Aufgabe haben Sie den **Konfigurations-Explorer** in Azure-App Konfiguration getestet.

#### Testen des Featureflags

Lassen Sie uns den Feature-Manager weiterhin testen.

1. Um dies zu versuchen, wechseln Sie zurück zur App-Konfiguration.
2. Wählen Sie im Abschnitt **Operations** (Vorgänge) die Option **Feature Manager** (Feature-Manager) aus.
3. Klicken Sie auf **"Erstellen** ", und fügen Sie folgendes hinzu:
    - **Featurekennzeichnung aktivieren:** Aktiviert
    - **Featurekennzeichnungsname:** SalesWeekend
4. Klicken Sie auf "Übernehmen"**, und wechseln Sie **zurück zu Ihrer Website, und aktualisieren Sie die Seite.
5. Sie sollten ein Bild mit dem Text "ALL T-SHIRTS ON SALE THIS WEEKEND" sehen.
6. Sie können dieses Feature in der App-Konfiguration deaktivieren und dann sehen, dass das Bild ausgeblendet wird.

Herzlichen Glückwunsch! In dieser Aufgabe haben Sie den **Feature-Manager** in Azure-App Konfiguration getestet.

### Übung 3: Entfernen der Azure-Lab-Ressourcen.

In dieser Übung entfernen Sie die in dieser Übung bereitgestellten Azure-Ressourcen, um unerwartete Gebühren zu beseitigen.

>**Hinweis**: Denken Sie daran, alle neu erstellten Azure-Ressourcen zu entfernen, die Sie nicht mehr verwenden. Durch das Entfernen nicht verwendeter Ressourcen wird sichergestellt, dass keine unerwarteten Kosten anfallen.

#### Übung 4: Entfernen der Azure Lab-Ressourcen

In dieser Aufgabe verwenden Sie Azure Cloud Shell, um die in dieser Übung bereitgestellten Azure-Ressourcen zu entfernen, um unnötige Gebühren zu vermeiden.

1. Öffnen Sie im Azure-Portal im **Cloud Shell**-Bereich die **Bash**-Sitzung.
2. Listen Sie alle Ressourcengruppen auf, die während der Labs in diesem Modul erstellt wurden, indem Sie den folgenden Befehl ausführen:

    ```sh
    az group list --query "[?starts_with(name,'AZ400-EWebShop-')].name" --output tsv
    ```

3. Löschen Sie alle Ressourcengruppen, die Sie während der praktischen Übungen in diesem Modul erstellt haben, indem Sie den folgenden Befehl ausführen:

    ```sh
    az group list --query "[?starts_with(name,'AZ400-EWebShop-')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**Hinweis**: Der Befehl wird (dem --nowait-Parameter entsprechend) asynchron ausgeführt. Dies bedeutet, dass Sie zwar einen weiteren Azure CLI-Befehl in derselben Bash-Sitzung direkt im Anschluss ausführen können, es jedoch einige Minuten dauert, bis die Ressourcengruppen tatsächlich entfernt werden.

## Überprüfung

In dieser Übung haben Sie erfahren, wie Sie Featurekennzeichnungen dynamisch aktivieren und verwalten.
