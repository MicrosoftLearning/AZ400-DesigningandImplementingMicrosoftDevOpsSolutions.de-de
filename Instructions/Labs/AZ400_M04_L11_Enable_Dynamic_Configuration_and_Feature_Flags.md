---
lab:
  title: Aktivieren von Featureflags und der dynamischen Konfiguration
  module: 'Module 04: Implement a secure continuous deployment using Azure Pipelines'
---

# Aktivieren von Featureflags und der dynamischen Konfiguration

## Lab-Handbuch für Kursteilnehmer*innen

## Labanforderungen

- Für dieses Lab ist **Microsoft Edge** oder ein von [Azure DevOps unterstützter Browser](https://learn.microsoft.com/azure/devops/server/compatibility) erforderlich.

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

### Übung 1: (überspringen, wenn erledigt) Importieren und Ausführen von CI/CD-Pipelines

In dieser Übung werden Sie die CI-Pipeline importieren und ausführen, die Dienstverbindung mit Ihrem Azure-Abonnement konfigurieren und dann die CD-Pipeline importieren und ausführen.

#### Aufgabe 1: Importieren und Ausführen der CI-Pipeline

Beginnen wir mit dem Importieren der CI-Pipeline mit dem Namen [eshoponweb-ci.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-ci.yml).

1. Navigieren Sie zu **Pipelines>Pipelines**.
1. Klicken Sie auf die Schaltfläche **Pipeline erstellen** (wenn keine Pipelines vorhanden sind) oder auf die Schaltfläche **Neue Pipeline** (wenn es bereits erstellte Pipelines gibt).
1. Wählen Sie **Azure Repos Git (Yaml)**.
1. Wählen Sie das Repository **eShopOnWeb** aus.
1. Wählen Sie die Option **Vorhandene Azure Pipelines-YAML-Datei** aus.
1. Wählen Sie die **Haupt**-Verzweigung und die Datei **/.ado/eshoponweb-ci.yml** aus, und klicken Sie dann auf **Weiter**.
1. Klicken Sie auf die Schaltfläche **Ausführen**, um die Pipeline auszuführen.
1. Ihre Pipeline bekommt einen Namen basierend auf dem Projektnamen. Wir **benennen sie um**, damit wir die Pipeline besser identifizieren können. Wechseln Sie zu **Pipelines>Pipelines** , und klicken Sie auf die kürzlich erstellte Pipeline. Klicken Sie auf die Auslassungspunkte und die Option **Umbenennen/Entfernen**. Nennen Sie ihn **eshoponweb-ci** und klicken Sie auf **Speichern**.

#### Aufgabe 2: Verwalten der Dienstverbindung

Sie können eine Verbindung zwischen Azure Pipelines und externen Diensten und Remotediensten herstellen, um Aufgaben in einem Auftrag auszuführen.

In dieser Aufgabe erstellen Sie einen Dienstprinzipal mithilfe der Azure CLI, was Azure DevOps Folgendes erlaubt:

- Bereitstellen von Ressourcen in Ihrem Azure-Abonnement
- Bereitstellen der eShopOnWeb-Anwendung

> **Hinweis**: Wenn Sie bereits über einen Dienstprinzipal verfügen, können Sie direkt mit der nächsten Aufgabe fortfahren.

Sie benötigen einen Dienstprinzipal, um Azure-Ressourcen aus Azure-Pipelines bereitzustellen.

Ein Dienstprinzipal wird automatisch von Azure Pipeline erstellt, wenn Sie eine Verbindung mit einem Azure-Abonnement innerhalb einer Pipelinedefinition herstellen oder wenn Sie eine neue Dienstverbindung über die Seite mit den Projekteinstellungen (automatische Option) erstellen. Sie können den Dienstprinzipal auch manuell über das Portal erstellen oder Azure CLI verwenden und es für alle Projekte wiederverwenden.

1. Starten Sie auf Ihrem Labcomputer einen Webbrowser, navigieren Sie zum [**Azure-Portal**](https://portal.azure.com), und melden Sie sich an. Verwenden Sie hierzu die Anmeldeinformationen eines Benutzerkontos, das in dem Abonnement, das Sie in diesem Lab verwenden, und das in dem in dem Microsoft Entra-Mandanten, der dem Abonnement zugeordnet ist, über die Rolle „Globaler Administrator“ verfügt.
1. Klicken Sie im Azure-Portal auf das Symbol **Cloud Shell**, das sich direkt rechts neben dem Textfeld für die Suche im oberen Bereich der Seite befindet.
1. Wählen Sie bei Aufforderung zur Auswahl von **Bash** oder **PowerShell** die Option **Bash** aus.

   >**Hinweis**: Wenn Sie **Cloud Shell** zum ersten Mal starten und die Meldung **Für Sie wurde kein Speicher bereitgestellt** angezeigt wird, wählen Sie das in diesem Lab verwendete Abonnement aus, und klicken Sie dann auf **Speicher erstellen**.

1. Führen Sie in der **Bash**-Eingabeaufforderung im **Cloud Shell**-Bereich die folgenden Befehle aus, um die Werte des Azure-Abonnement-ID-Attributs abzurufen:

    ```sh
    subscriptionName=$(az account show --query name --output tsv)
    subscriptionId=$(az account show --query id --output tsv)
    echo $subscriptionName
    echo $subscriptionId
    ```

    > **Hinweis**: Kopieren Sie beide Werte in eine Textdatei. Sie werden sie später in diesem Lab benötigen.

1. Führen Sie in der **Bash**-Eingabeaufforderung im Bereich **Cloud Shell** den folgenden Befehl aus, um einen Dienstprinzipal zu erstellen:

    ```sh
    az ad sp create-for-rbac --name sp-az400-azdo --role contributor --scopes /subscriptions/$subscriptionId
    ```

    > **Hinweis**: Der Befehl generiert eine JSON-Ausgabe. Kopieren Sie die Ausgabe in eine Textdatei. Sie benötigen diese später in diesem Lab.

1. Starten Sie als Nächstes auf dem Laborcomputer einen Webbrowser, navigieren Sie zum Azure DevOps **eShopOnWeb**-Projekt. Klicken Sie auf **Project Einstellungen>Dienstverbindungen (unter Pipelines)** und **Neue Dienstverbindung**.

1. Wählen Sie im Bildschirm **Neue Dienstverbindung** die Option **Azure Resource Manager** und anschließend **Weiter** aus (Sie müssen möglicherweise scrollen).

1. Wählen Sie **Dienstprinzipal (manuell)** aus, und klicken Sie auf **Weiter**.

1. Füllen Sie die leeren Felder mit den Informationen aus, die während der vorherigen Schritte gesammelt wurden:
    - Abonnement-ID und -Name
    - Dienstprinzipal-ID (oder ClientId), Schlüssel (oder Kennwort) und TenantId.
    - Geben Sie in **Name der Dienstverbindung** **azure subs** ein. Auf diesen Namen wird in YAML-Pipelines verwiesen, wenn eine Azure DevOps-Dienstverbindung erforderlich ist, um mit Ihrem Azure-Abonnement zu kommunizieren.

1. Klicken Sie auf **Überprüfen und speichern**.

#### Aufgabe 3: Importieren und Ausführen der CD-Pipeline

Importieren Sie nun die CD-Pipeline mit dem Namen [eshoponweb-cd-webapp-code.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-cd-webapp-code.yml).

1. Navigieren Sie zu **Pipelines>Pipelines**.
1. Klicken Sie auf die Schaltfläche **Neue Pipeline**.
1. Wählen Sie **Azure Repos Git (Yaml)**.
1. Wählen Sie das Repository **eShopOnWeb** aus.
1. Wählen Sie die Option **Vorhandene Azure Pipelines-YAML-Datei** aus.
1. Wählen Sie die **Haupt**-Verzweigung und die Datei **/.ado/eshoponweb-cd-webapp-code.yml** aus, und klicken Sie dann auf **Weiter**.
1. Passen Sie in der YAML-Pipelinedefinition Folgendes an:
   - Ersetzen Sie **IHRE-ABONNEMENT-ID** durch Ihre Azure-Abonnement-ID.
   - **az400eshop-NAME** ersetzen Sie NAME, um ihn global eindeutig zu machen.
   - **AZ400-EWebShop-NAME** mit dem zuvor im Labor definierten Namen der Ressourcengruppe.

1. Klicken Sie auf **Speichern und Ausführen**, und warten Sie, bis die Pipeline erfolgreich ausgeführt wird.

    > **Hinweis**: Die Bereitstellung kann einige Minuten dauern.

    Die CD-Definition besteht aus den folgenden Aufgaben:
    - **Ressourcen**: Sie ist darauf vorbereitet, automatisch nach Abschluss der CI-Pipeline ausgelöst zu werden. Außerdem wird das Repository für die Bicep-Datei heruntergeladen.
    - **AzureResourceManagerTemplateDeployment**: Stellt die Azure Web App mithilfe der Bicep-Vorlage bereit.

1. Ihre Pipeline bekommt einen Namen basierend auf dem Projektnamen. Wir **benennen sie um**, damit wir die Pipeline besser identifizieren können. Wechseln Sie zu **Pipelines>Pipelines** , und klicken Sie auf die kürzlich erstellte Pipeline. Klicken Sie auf die Auslassungspunkte und die Option **Umbenennen/Entfernen**. Nennen Sie es **eshoponweb-cd-webapp-code** und klicken Sie auf **Speichern**.

### Übung 2: Verwalten der Azure App-Konfiguration

In dieser Übung erstellen Sie die App-Konfigurationsressource in Azure, aktivieren die verwaltete Identität und testen dann die vollständige Lösung.

> **Hinweis**: Für diese Übung sind keine Programmierkenntnisse erforderlich. Der Code der Website implementiert bereits Azure-App Konfigurationsfunktionen.

Wenn Sie wissen möchten, wie Sie dies in Ihrer Anwendung implementieren können, werfen Sie einen Blick auf diese Tutorials: [Dynamische Konfiguration in einer ASP.NET Core-App verwenden](https://learn.microsoft.com/azure/azure-app-configuration/enable-dynamic-configuration-aspnet-core) und [Funktionsflags in Azure App Configuration verwalten](https://learn.microsoft.com/azure/azure-app-configuration/manage-feature-flags).

#### Aufgabe 1: Erstellen Sie die App-Konfigurationsressource

1. Suchen Sie im Azure-Portal nach dem Dienst **App-Konfiguration**.
1. Klicken Sie auf **App-Konfiguration erstellen** und wählen Sie dann:
    - Ihr Azure-Abonnement.
    - Die zuvor erstellte Ressourcengruppe (sie sollte **AZ400-EWebShop-NAME** heißen).
    - Der Speicherort.
    - Zum Beispiel ein eindeutiger Name wie **appcs-NAME-REGION**.
    - Wählen Sie den Tarif **Kostenlos**.
1. Klicken Sie auf **Überprüfen+ erstellen** und danach auf **Erstellen**.
1. Wechseln Sie nach dem Erstellen des App-Konfigurationsdiensts zu **Übersicht**, und kopieren/speichern Sie den Wert des **Endpunkts**.

#### Aufgabe 2: Verwaltete Identität aktivieren

1. Gehen Sie zu der Web-App, die mit der Pipeline bereitgestellt wurde (sie sollte **az400-webapp-NAME** heißen).
1. Klicken Sie im Abschnitt **Einstellungen** auf **Identität** und schalten Sie dann im Abschnitt **Vom System zugewiesen** den Status auf **Ein**. Klicken Sie auf **Speichern > Ja** und warten Sie einige Sekunden, bis der Vorgang abgeschlossen ist.
1. Gehen Sie zurück zum App-Konfigurationsdienst und klicken Sie auf **Zugriffskontrolle** und dann auf **Rollenzuweisung hinzufügen**.
1. Wählen Sie im Abschnitt **Rolle** die Option **App Configuration-Datenleser** aus.
1. Aktivieren Sie im Abschnitt **Mitglieder** das Kontrollkästchen **Identität verwalten** und wählen Sie dann die verwaltete Identität Ihrer Web-App aus (sie sollten denselben Namen haben).
1. Klicken Sie auf **Überprüfen und zuweisen**.

#### Aufgabe 5: Konfigurieren der Web-App

Um sicherzustellen, dass Ihre Website auf die App-Konfiguration zugreift, müssen Sie die Konfiguration aktualisieren.

1. Gehen Sie zurück zu Ihrer Web-App.
1. Klicken Sie im Abschnitt **Einstellungen** auf **Konfiguration**.
1. Fügen Sie zwei neue Anwendungseinstellungen hinzu:
    - Erste App-Einstellung
        - **Name:** UseAppConfig
        - **Wert: true**
    - Zweite App-Einstellung
        - **Name:** AppConfigEndpoint
        - **Wert:***der Wert, den Sie zuvor aus dem App Configuration Endpoint gespeichert/kopiert haben. Er sollte wie <https://appcs-NAME-REGION.azconfig.io>* aussehen

1. Klicken Sie auf **Ok** und dann auf **Speichern** und warten Sie, bis die Einstellungen aktualisiert werden.
1. Gehen Sie zu **Übersicht** und klicken Sie auf **Durchsuchen**
1. In diesem Schritt werden keine Änderungen auf der Website angezeigt, da die App-Konfiguration keine Daten enthält. Dies ist das, was Sie in den nächsten Aufgaben tun werden.

#### Aufgabe 4: Testen der Konfigurationsverwaltung

1. Wählen Sie auf Ihrer Website in der Dropdownliste **Marke** den Eintrag **Visual Studio**, und klicken Sie auf die Pfeilschaltfläche (**>**).
1. Sie sehen die Meldung *ES GIBT KEINE ERGEBNISSE, DIE IHRER SUCHE ENTSPRECHEN*. Ziel dieses Labs ist es, diesen Wert zu aktualisieren, ohne den Code der Website zu aktualisieren oder ihn erneut bereitzustellen.
1. Um dies zu versuchen, wechseln Sie zurück zur Appkonfiguration.
1. Wählen Sie im Abschnitt **Vorgänge** die Option **Konfigurationsexplorer**.
1. Klicken Sie auf **Erstellen > Schlüsselwert**, und fügen Sie dann Folgendes hinzu:
    - **Schlüssel:** eShopWeb:Einstellungen:NoResultsMessage
    - **Wert:***Geben Sie Ihre benutzerdefinierte Nachricht ein.*
1. Klicken Sie auf **Anwenden**, gehen Sie zurück zu Ihrer Website, und aktualisieren Sie die Seite.
1. Ihre neue Nachricht sollte anstelle des alten Standardwerts angezeigt werden.

Herzlichen Glückwunsch! In dieser Aufgabe haben Sie den **Konfigurationsexplorer** in Azure App Configuration getestet.

#### Aufgabe 5: Testen Sie das Featureflag

Testen wir weiter den Featuremanager.

1. Um dies zu versuchen, wechseln Sie zurück zur Appkonfiguration.
1. Wählen Sie im Abschnitt **Operations** (Vorgänge) die Option **Feature Manager** (Feature-Manager) aus.
1. Klicken Sie auf **Erstellen**, und fügen Sie dann hinzu:
    - **Featureflag aktivieren:** geprüft
    - **Featureflag-Name:** SalesWeekend
1. Klicken Sie auf **Anwenden**, gehen Sie zurück zu Ihrer Website, und aktualisieren Sie die Seite.
1. Sie sollten ein Bild mit dem Text „ALL T-SHIRTS ON SALE THIS WEEKEND“ sehen.
1. Sie können dieses Feature in der Appkonfiguration deaktivieren – dann sehen Sie, dass das Bild ausgeblendet wird.

Herzlichen Glückwunsch! In dieser Aufgabe haben Sie den **Featuremanager** in der Azure-Appkonfiguration getestet.

### Übung 3: Entfernen der Azure-Laborressourcen

In dieser Übung entfernen Sie die in diesem Lab bereitgestellten Azure-Ressourcen, um unerwartete Gebühren zu vermeiden.

>**Hinweis**: Denken Sie daran, alle neu erstellten Azure-Ressourcen zu entfernen, die Sie nicht mehr verwenden. Durch das Entfernen nicht verwendeter Ressourcen wird sichergestellt, dass keine unerwarteten Gebühren anfallen.

#### Aufgabe 1: Entfernen der Azure Lab-Ressourcen

In dieser Aufgabe verwenden Sie Azure Cloud Shell, um die in diesem Lab bereitgestellten Azure-Ressourcen zu entfernen, um unnötige Gebühren zu vermeiden.

1. Öffnen Sie im Azure-Portal die **Bash**-Shell-Sitzung im Bereich **Cloud Shell**.
1. Listen Sie alle Ressourcengruppen auf, die während der Labs in diesem Modul erstellt wurden, indem Sie den folgenden Befehl ausführen:

    ```sh
    az group list --query "[?starts_with(name,'AZ400-EWebShop-')].name" --output tsv
    ```

1. Löschen Sie alle Ressourcengruppen, die Sie während der praktischen Übungen in diesem Modul erstellt haben, indem Sie den folgenden Befehl ausführen:

    ```sh
    az group list --query "[?starts_with(name,'AZ400-EWebShop-')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**Hinweis**: Der Befehl wird (dem --nowait-Parameter entsprechend) asynchron ausgeführt. Dies bedeutet, dass Sie zwar einen weiteren Azure CLI-Befehl in derselben Bash-Sitzung direkt im Anschluss ausführen können, es jedoch einige Minuten dauert, bis die Ressourcengruppen tatsächlich entfernt werden.

## Überprüfung

In diesem Lab haben Sie erfahren, wie Sie Featureflags dynamisch aktivieren und verwalten.
