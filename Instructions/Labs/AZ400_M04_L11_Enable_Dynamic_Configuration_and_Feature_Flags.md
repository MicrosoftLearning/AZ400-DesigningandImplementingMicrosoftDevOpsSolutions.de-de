---
lab:
  title: Aktivieren von Featureflags und der dynamischen Konfiguration
  module: 'Module 04: Implement a secure continuous deployment using Azure Pipelines'
---

# Aktivieren von Featureflags und der dynamischen Konfiguration

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

## Geschätzte Dauer: 45 Minuten

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

### Übung 1: (überspringen, wenn erledigt) Importieren und Ausführen von CI/CD-Pipelines

In dieser Übung importieren Sie CI/CD-Pipelines, um die eShopOnWeb-Anwendung zu erstellen und bereitzustellen. Die CI-Pipeline ist bereits für die Erstellung der Anwendung und die Durchführung von Tests vorbereitet. Die CD-Pipeline stellt die Anwendung in einer Azure Web-App bereit.

#### Aufgabe 1: Importieren und Ausführen der CI-Pipeline

Beginnen wir mit dem Importieren der CI-Pipeline mit dem Namen [eshoponweb-ci.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-ci.yml).

1. Navigieren Sie zu **Pipelines > Pipelines**.
1. Klicken Sie auf die Schaltfläche **Pipeline erstellen** (wenn keine Pipelines vorhanden sind) oder auf die Schaltfläche **Neue Pipeline** (wenn es bereits erstellte Pipelines gibt).
1. Wählen Sie **Azure Repos Git (Yaml)**.
1. Wählen Sie das Repository **eShopOnWeb** aus.
1. Wählen Sie die Option **Vorhandene Azure Pipelines-YAML-Datei** aus.
1. Wählen Sie die **Haupt**-Verzweigung und die Datei **/.ado/eshoponweb-ci.yml** aus, und klicken Sie dann auf **Weiter**.
1. Klicken Sie auf die Schaltfläche **Ausführen**, um die Pipeline auszuführen.
1. Ihre Pipeline bekommt einen Namen basierend auf dem Projektnamen. Wir **benennen sie um**, damit wir die Pipeline besser identifizieren können. Gehen Sie zu **Pipelines > Pipelines** und klicken Sie auf die kürzlich erstellte Pipeline. Klicken Sie auf die Auslassungspunkte und die Option **Umbenennen/Entfernen**. Nennen Sie ihn **eshoponweb-ci** und klicken Sie auf **Speichern**.

#### Übung 2: Importieren und Ausführen der CD-Pipeline

Importieren Sie nun die CD-Pipeline mit dem Namen [eshoponweb-cd-webapp-code.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-cd-webapp-code.yml).

1. Navigieren Sie zu **Pipelines > Pipelines**.
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

1. Ihre Pipeline bekommt einen Namen basierend auf dem Projektnamen. Wir **benennen sie um**, damit wir die Pipeline besser identifizieren können. Gehen Sie zu **Pipelines > Pipelines** und klicken Sie auf die kürzlich erstellte Pipeline. Klicken Sie auf die Auslassungspunkte und die Option **Umbenennen/Entfernen**. Nennen Sie es **eshoponweb-cd-webapp-code** und klicken Sie auf **Speichern**.

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
1. Klicken Sie im Abschnitt **Einstellungen** auf **Umgebungsvariablen**.
1. Fügen Sie zwei neue Anwendungseinstellungen hinzu:
    - Erste App-Einstellung
        - **Name:** UseAppConfig
        - **Wert: true**
    - Zweite App-Einstellung
        - **Name:** AppConfigEndpoint
        - **Wert:***der Wert, den Sie zuvor aus dem App Configuration Endpoint gespeichert/kopiert haben. Er sollte wie <https://appcs-NAME-REGION.azconfig.io>* aussehen

1. Klicken Sie auf **Anwenden** und dann **Bestätigen**, und warten Sie, bis die Einstellungen aktualisiert werden.
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

   > [!IMPORTANT]
   > Denken Sie daran, die im Azure-Portal erstellten Ressourcen zu löschen, um unnötige Kosten zu vermeiden. Achten Sie darauf, die **eshoponweb-cd-webapp-code** Pipeline zu deaktivieren, da sonst eine gelöschte Ressourcengruppe und die zugehörigen Ressourcen nach dem nächsten Lauf von **eshoponweb-ci** neu erstellt werden.

## Überprüfung

In diesem Lab haben Sie erfahren, wie Sie Featureflags dynamisch aktivieren und verwalten.
