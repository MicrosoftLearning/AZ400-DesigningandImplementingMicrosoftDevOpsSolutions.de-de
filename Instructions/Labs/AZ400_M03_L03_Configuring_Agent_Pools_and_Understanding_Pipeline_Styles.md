---
lab:
  title: 'Lab: Konfigurieren von Agent-Pools und Verstehen von Pipelinearten'
  module: 'Module 03: Implement CI with Azure Pipelines and GitHub Actions'
---

# Lab: Konfigurieren von Agent-Pools und Verstehen von Pipelinearten

## Lab-Handbuch für Kursteilnehmer

## Labanforderungen

- Für dieses Lab ist **Microsoft Edge** oder ein [von Azure DevOps unterstützter Browser](https://docs.microsoft.com/azure/devops/server/compatibility) erforderlich.

- **Einrichten einer Azure DevOps-Organisation**: Wenn Sie nicht bereits eine Azure DevOps-Organisation haben, die Sie für dieses Lab verwenden können, müssen Sie diese erstellen, indem Sie die unter [Erstellen einer Organisation oder Projektsammlung](https://docs.microsoft.com/azure/devops/organizations/accounts/create-organization) beschriebenen Anweisungen befolgen.

- [Git for Windows](https://gitforwindows.org/): Downloadseite. Diese wird als eine Voraussetzung für dieses Lab installiert.

- [Visual Studio Code](https://code.visualstudio.com/). Diese wird als eine Voraussetzung für dieses Lab installiert.

## Übersicht über das Labor

YAML-basierte Pipelines bieten Ihnen die Möglichkeit, CI/CD vollständig als Code zu implementieren. Dabei befinden sich die Pipelinedefinitionen im selben Repository wie der Code, der Bestandteil Ihres Azure DevOps-Projekts ist. YAML-basierte Pipelines unterstützen eine Vielzahl von Features, die auch zu klassischen Pipelines gehören, z. B. Pull Requests, Codeüberprüfungen, Verlauf, Verzweigungen und Vorlagen.

Ungeachtet der ausgewählten Pipelineart benötigen Sie einen Agent, um Ihren Code zu erstellen oder Ihre Lösung mithilfe von Azure Pipelines bereitzustellen. Ein Agent hostet Computeressourcen, die jeweils einen Auftrag ausführen. Aufträge können direkt auf dem Hostcomputer des Agents oder in einem Container ausgeführt werden. Sie haben die Möglichkeit, Ihre Aufträge mit von Microsoft gehosteten und verwalteten Agents auszuführen oder einen selbst gehosteten Agent zu implementieren, den Sie selbst einrichten und verwalten.

In diesem Lab erfahren Sie, wie Sie selbstgehostete Agents mit YAML-Pipelines implementieren und verwenden.

## Ziele

In diesem Lab lernen Sie Folgendes:

- Implementieren YAML-basierter Pipelines.
- Implementieren selbstgehosteter Agents.

## Geschätzte Dauer: 45 Minuten

## Anweisungen

### Übung 0: Konfigurieren der Voraussetzungen für das Lab.

In dieser Übung richten Sie die Voraussetzungen für das Labor ein, das aus einem neuen Azure DevOps-Projekt mit einem Repository basierend auf dem [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb) besteht.

#### Aufgabe 1: (überspringen, wenn fertig) Erstellen und Konfigurieren des Teamprojekts

In dieser Aufgabe erstellen Sie ein **eShopOnWeb** Azure DevOps-Projekt, das von mehreren Laboren verwendet werden soll.

1. Öffnen Sie auf Ihrem Laborcomputer in einem Browserfenster Ihre Azure DevOps-Organisation. Klicken auf „Neues Projekt“ Weisen Sie Ihrem Projekt den Namen **"eShopOnWeb** " zu, und lassen Sie die anderen Felder standardmäßig. Klicken Sie auf **Erstellen**.

#### Aufgabe 2: (überspringen, wenn erledigt) Importieren von eShopOnWeb Git Repository

Bei dieser Aufgabe importieren Sie das eShopOnWeb Git-Repository, das von mehreren Labs verwendet wird.

1. Öffnen Sie auf Ihrem Laborcomputer in einem Browserfenster Ihre Azure DevOps-Organisation und das zuvor erstellte **eShopOnWeb-Projekt** . Klicken Sie auf **Repos>Files** , **Import a Repository**. Wählen Sie **Importieren** aus. Fügen Sie im **Fenster "Git Repository** importieren" die folgende URL https://github.com/MicrosoftLearning/eShopOnWeb.git ein, und klicken Sie auf " **Importieren**":

2. Das Repository ist wie folgt organisiert:
    - Der Ordner „.ado“ enthält Azure DevOps-YAML-Pipelines.
    - Der Ordner „.devcontainer“ enthält ein Containersetup für die Entwicklung mithilfe von Containern (entweder lokal in VS Code oder über GitHub Codespaces).
    - **Azure-Ordner** enthält Bicep & ARM-Infrastruktur als Codevorlagen, die in einigen Laborszenarien verwendet werden.
    - **Der Ordner ".github** " enthält YAML-GitHub-Workflowdefinitionen.
    - Der Ordner „src“ enthält die .NET 6-Website, die in den Labszenarien verwendet wird.

### Übung 1: Erstellen von YAML-basierten Azure Pipelines.

In dieser Übung erstellen Sie eine Anwendungslebenszyklus-Buildpipeline mit einer YAML-basierten Vorlage.

#### Erstellen einer Azure DevOps-YAML-Pipeline

In dieser Aufgabe erstellen Sie eine vorlagenbasierte Azure DevOps YAML-Pipeline.

1. Klicken Sie im Webbrowser, in dem das Azure DevOps-Portal mit dem **geöffneten EShopOnWeb-Projekt** im vertikalen Navigationsbereich auf der linken Seite auf "Pipelines **" angezeigt **wird.
2. Klicken Sie auf der **Registerkarte "Zuletzt verwendet** " im **Bereich "Pipelines** " auf " **Neue Pipeline"**.
3. Wählen Sie auf der Seite **Wo befindet sich Ihr Code?** die Option **Azure Repos Git** aus.
4. Klicken Sie im **Bereich "Repository** auswählen" auf **"EShopOnWeb"**.
5. Wählen Sie auf dem Bildschirm **Pipeline konfigurieren** die Option **Vorhandene Azure Pipelines-YAML-Datei** aus.
6. Wählen Sie in der **Datei "Vorhandene YAML-Datei** auswählen **" Standard** für "Branch" und **"/.ado/eshoponweb-ci-pr.yml**" für den Pfad aus.
7. Klicken Sie auf **Weiter**.
8. Überprüfen Sie im **Bereich "YaML-Pipeline** überprüfen" die Beispielpipeline. Dies ist eine ziemlich gerade .NET-Anwendungsbuildpipeline, die die folgenden Aktionen ausführt:
- Eine einzelne Stufe: Erstellen
- Ein einzelner Auftrag: Erstellen
- 3 Aufgaben innerhalb des Buildauftrags:
- dotnet restore
- dotnet build
- dotnet publish

9. Klicken Sie im **Bereich "YaML-Pipeline** überprüfen" auf das nach unten weisende Caretsymbol neben der **Schaltfläche "Ausführen** ", und klicken Sie auf " **Speichern**".

    > Hinweis: Wir erstellen jetzt nur die Pipelinedefinition, ohne sie auszuführen. Sie richten zunächst einen Azure DevOps-Agentpool ein und führen die Pipeline in einer späteren Übung aus. 

### Übung 2: Verwalten von Azure DevOps-Agentpools.

In dieser Übung implementieren Sie einen selbst gehosteten Azure DevOps-Agent.

#### Konfigurieren Sie einen selbst gehosteten Azure DevOps-Agent.

In dieser Aufgabe konfigurieren Sie Ihren virtuellen Laborcomputer als Azure DevOps Self-Hosting-Agent und verwenden ihn zum Ausführen einer Buildpipeline.

1. Starten Sie innerhalb des virtuellen Lab-Computers (Lab VM) oder Ihres eigenen Computers einen Webbrowser, navigieren Sie zum [Azure DevOps-Portal](https://dev.azure.com) , und melden Sie sich mit dem Microsoft-Konto an, das Ihrer Azure DevOps-Organisation zugeordnet ist.

  > **Hinweis**: Der virtuelle Lab-Computer sollte alle erforderlichen Erforderlichen Software installiert haben. Wenn Sie auf Ihrem eigenen Computer installieren, müssen Sie .NET 7.0.x SDKs oder höher installieren, die zum Erstellen des Demoprojekts erforderlich sind. Siehe [.NET](https://dotnet.microsoft.com/download/dotnet) herunterladen.

1. Klicken Sie im Azure DevOps-Portal in der oberen rechten Ecke der Azure DevOps-Seite auf das Symbol "**Benutzereinstellungen**", je nachdem, ob Sie Vorschaufeatures aktiviert haben, im Menü entweder ein **Element für Sicherheits**- oder **persönliche Zugriffstoken** angezeigt werden, wenn "Sicherheit **" angezeigt **wird, klicken Sie darauf, und wählen Sie **dann "Persönliche Zugriffstoken"** aus. Klicken Sie im **Bereich "Persönliche Zugriffstoken** " auf **+Neues Token**.
2. Klicken Sie im **Bereich "Neuen persönlichen Zugriffstoken** erstellen" auf den **Link "Alle Bereiche anzeigen", und geben Sie die folgenden Einstellungen an, und klicken Sie auf "**Erstellen**" (alle anderen Personen mit ihren Standardwerten**):

    | Einstellung | Wert |
    | --- | --- |
    | Name | **eShopOnWeb** |
    | Bereich (benutzerdefinierter Bereich) | **Agentpools** (option "Weitere Bereiche anzeigen" unten bei Bedarf anzeigen)|
    | Berechtigungen | **Lesen und Verwalten** |

3. Kopieren Sie im **Bereich "Erfolg** " den Wert des persönlichen Zugriffstokens in die Zwischenablage.

    > **Hinweis**: Stellen Sie sicher, dass Sie das Token kopieren. Nachdem Sie diesen Bereich geschlossen haben, können Sie ihn nicht mehr abrufen.

4. Klicken Sie im **Bereich "Erfolg** " auf " **Schließen"**.
5. **Klicken Sie im Bereich "Persönliches Zugriffstoken**" im Azure DevOps-Portal in der oberen linken Ecke auf **das Azure DevOps-Symbol**, und klicken Sie dann in der unteren linken Ecke auf **die Bezeichnung "Organisationseinstellungen**".
6. Klicken Sie links im **Bereich "Übersicht**" im vertikalen Menü im **Abschnitt "Pipelines" auf **"Agentpools**"**.
7. Klicken Sie im **Bereich "Agentpools** " in der oberen rechten Ecke auf **"Pool hinzufügen"**.
8. **Wählen Sie im Bereich "Agentpool** hinzufügen" in der Dropdownliste "Pooltyp **" im **Textfeld "Name **" die Option **"Selbst gehostet**" aus**, geben Sie **"az400m03l03a-pool**" ein, und klicken Sie dann auf "**Erstellen**".
9.  Klicken Sie im **Bereich "Agentpools** " auf den Eintrag, der den neu erstellten **az400m03l03a-Pool** darstellt.
10. Klicken Sie auf der **Registerkarte "Aufträge** " des **Bereichs "az400m03l03a-pool** " auf die **Schaltfläche "Neuer Agent** ".
11. **Stellen Sie im Bereich "Agent** abrufen" sicher, dass die **Registerkarten Windows** und **x64** ausgewählt sind, und klicken Sie auf **"Herunterladen**", um das ZIP-Archiv mit den Agent-Binärdateien herunterzuladen, um es in den lokalen **Ordner "Downloads**" in Ihrem Benutzerprofil herunterzuladen.

    > **Hinweis**: Wenn an diesem Punkt eine Fehlermeldung angezeigt wird, die angibt, dass die aktuellen Systemeinstellungen verhindern, dass Sie die Datei herunterladen können, klicken Sie im Browserfenster in der oberen rechten Ecke auf das Zahnradsymbol, das die kopfzeile des **menüs Einstellungen**, im Dropdownmenü, im Dropdownmenü internetoptionen** auswählen**, im **Dialogfeld "Internetoptionen**" auf "Erweitert", klicken Sie auf **der **Registerkarte "Erweitert****",  klicken Sie auf "Zurücksetzen **", klicken Sie **im **Dialogfeld "Browser zurücksetzen" Einstellungen** erneut auf **"Zurücksetzen**", klicken Sie auf "Schließen **", und versuchen Sie **es erneut.

12. Starten Sie Windows PowerShell als Administrator und führen Sie in der **Administratorkonsole** die folgenden Zeilen aus, um das **C:\\Agent-Verzeichnis** zu erstellen und den Inhalt des heruntergeladenen Archivs darin zu extrahieren.

    ```powershell
    cd \
    mkdir agent ; cd agent
    $TARGET = Get-ChildItem "$Home\Downloads\vsts-agent-win-x64-*.zip"
    Add-Type -AssemblyName System.IO.Compression.FileSystem
    [System.IO.Compression.ZipFile]::ExtractToDirectory($TARGET, "$PWD")
    ```

14. Führen Sie in demselben **Administrator: Windows PowerShell-Konsole** Folgendes aus, um den Agent zu konfigurieren:

    ```powershell
    .\config.cmd
    ```

15. Wenn Sie dazu aufgefordert werden, geben Sie die Werte der folgenden Einstellungen an:

    | Einstellung | Wert |
    | ------- | ----- |
    | Server-URL eingeben | die URL Ihrer Azure DevOps-Organisation im Format****<https://dev.azure.com/>`<organization_name>`, in `<organization_name>` dem der Name Ihrer Azure DevOps-Organisation steht |
    | Geben Sie den Authentifizierungstyp ein (drücken Sie Enter für PAT) : Drücken Sie Enter. | **EINGABETASTE** |
    | Ihr persönliches Zugriffstoken | Das Zugriffstoken, das Sie zuvor in dieser Aufgabe aufgezeichnet haben |
    | Geben Sie den Agentpool ein (drücken Sie Enter für die Standardeinstellung) : Drücken Sie Enter. | **az400m03l03a-pool** |
    | Geben Sie den Namen eines Ereignisses ein. | **az104-06-vm0** |
    | Geben Sie den Arbeitsordner ein (drücken Sie die Enter für _work) : Drücken Sie Enter. | **EINGABETASTE** |
    | **(Nur wenn angezeigt)** Geben Sie für jeden Schritt eine Entpackung für Aufgaben ein. (drücken Sie die EINGABETASTE für N) | **WARNUNG**: Drücken Sie **nur die EINGABETASTE** , wenn die Nachricht angezeigt wird.|
    | Agent als Dienst ausführen? (J/N) (drücken Sie Enter für N) : Geben Sie J ein. | **Y** |
    | Geben Sie SERVICE_SID_TYPE_UNRESTRICTED für den Agentdienst (J/N) ein (drücken Sie Enter für N) : geben Sie J ein. | **Y** |
    | Geben Sie das Benutzerkonto ein, das für den Dienst verwendet werden soll (drücken Sie Enter für NT AUTHORITY\NETWORK SERVICE) : Drücken Sie Enter. | **EINGABETASTE** |
    | Geben Sie an, ob der Dienst sofort nach Abschluss der Konfiguration gestartet werden soll? (J/N) (drücken Sie Enter für N) : Geben Sie J ein. | **EINGABETASTE** |

    > Sie können Ihren selbstgehosteten Agent entweder als Dienst oder als interaktiven Prozess ausführen. Möglicherweise möchten Sie mit dem interaktiven Modus beginnen, da dies die Überprüfung der Agent-Funktionalität vereinfacht. Für die Produktionsverwendung sollten Sie entweder die Ausführung des Agents als Dienst oder als interaktiver Prozess mit aktivierter automatischer Anmeldung in Betracht ziehen, da beide ihren Ausführungsstatus beibehalten und sicherstellen, dass der Agent automatisch gestartet wird, wenn das Betriebssystem neu gestartet wird.

16. Wechseln Sie zum Browserfenster, in dem das Azure DevOps-Portal angezeigt wird, und schließen Sie den **Agentbereich** abrufen.
17. Zurück auf der **Registerkarte "Agents** " im **Bereich "az400m03l03a-pool** ", beachten Sie, dass der neu konfigurierte Agent mit dem **Onlinestatus** aufgeführt ist.
18. Klicken Sie im Webbrowserfenster, in dem das Azure DevOps-Portal angezeigt wird, in der oberen linken Ecke auf die **Azure DevOps-Bezeichnung** .
19. Klicken Sie im Browserfenster, in dem die Liste der Projekte angezeigt wird, auf die Kachel, die Das **Projekt "Agentpools konfigurieren" und "Grundlegendes zu Pipelineformatvorlagen"** darstellt.
20. Klicken Sie im **Bereich "EShopOnWeb** " im vertikalen Navigationsbereich auf der linken Seite im **Abschnitt "Pipelines** " auf **"Pipelines**".
21. Wählen Sie auf der **Registerkarte "Zuletzt verwendet**" im **Bereich "Pipelines**" die Option **"EShopOnWeb**" aus, und wählen Sie im **Bereich "EShopOnWeb**" die Option "Bearbeiten"** aus**.
22. **Ersetzen Sie im EShopOnWeb-Bearbeitungsbereich** in der vorhandenen YAML-basierten Pipeline Die Zeile 13, die besagt`vmImage: ubuntu-latest`, dass der Ziel-Agent-Pool den folgenden Inhalt entwerfen und den neu erstellten selbstgehosteten Agent-Pool entwerfen soll:

    ```yaml
    name: az400m03l03a-pool
    demands:
    - Agent.Name -equals az400m03-vm0
    ```

    > **WARNUNG**: Achten Sie beim Kopieren/Einfügen darauf, dass Sie den gleichen Einzug wie oben gezeigt haben.

23. Klicken Sie im **EShopOnWeb-Bearbeitungsbereich** in der oberen rechten Ecke des Bereichs auf " **Speichern** ", und klicken Sie im **Bereich "Speichern** " erneut auf " **Speichern** ". Dadurch wird die Buildpipeline automatisch gestartet.
24. Klicken Sie im Azure DevOps-Portal im vertikalen Navigationsbereich auf der linken Seite im **Abschnitt "Pipelines** " auf **"Pipelines**".
25. Klicken Sie auf der **Registerkarte "Zuletzt verwendet** " im **Bereich "Pipelines** " auf den **Eintrag "EShopOnWeb"** , auf der **Registerkarte "Ausführen** " des **Bereichs "EShopOnWeb** ", wählen Sie im **Bereich "Zusammenfassung** " der Ausführung einen Bildlauf nach unten im **Abschnitt "Aufträge** " auf " **Phase 1** " aus, und überwachen Sie den Auftrag bis zum erfolgreichen Abschluss.

### Übung 3: Entfernen der Azure-Lab-Ressourcen.

1. Beenden und entfernen Sie den Agentdienst, indem Sie über die Eingabeaufforderung ausgeführt `.\config.cmd remove` werden.
2. Löschen Sie den Agentpool.
3. Widerrufen sie das PAT-Token.

## Überprüfung

In diesem Lab erfahren Sie, wie Sie selbstgehostete Agents mit YAML-Pipelines implementieren und verwenden.
