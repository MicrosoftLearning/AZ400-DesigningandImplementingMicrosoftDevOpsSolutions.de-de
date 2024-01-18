---
lab:
  title: Konfigurieren von Agentenpools und Verstehen von Pipelinestilen
  module: 'Module 03: Implement CI with Azure Pipelines and GitHub Actions'
---

# Konfigurieren von Agentenpools und Verstehen von Pipelinestilen

# Lab-Handbuch für Kursteilnehmer

## Labanforderungen

- Für dieses Lab ist **Microsoft Edge** oder ein von [Azure DevOps unterstützter Browser](https://docs.microsoft.com/en-us/azure/devops/server/compatibility?view=azure-devops#web-portal-supported-browsers) erforderlich.

- **Einrichten einer Azure DevOps-Organisation**: Wenn Sie nicht bereits eine Azure DevOps-Organisation haben, die Sie für dieses Lab verwenden können, müssen Sie diese erstellen, indem Sie die unter [Erstellen einer Organisation oder Projektsammlung](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/create-organization?view=azure-devops) beschriebenen Anweisungen befolgen.

- [Git for Windows](https://gitforwindows.org/): Downloadseite. Diese wird als Teil der Voraussetzungen für diese Übung installiert.

- [Visual Studio Code](https://code.visualstudio.com/). Diese wird als Teil der Voraussetzungen für diese Übung installiert.

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

### Übung 0: Konfigurieren der Voraussetzungen für das Lab

In dieser Übung werden Sie die Voraussetzung für die Übung einrichten, die aus dem vorkonfigurierten Parts Unlimited-Teamprojekt auf der Grundlage einer Azure DevOps Demo Generator-Vorlage besteht.

#### Aufgabe 1: Konfigurieren Sie das Teamprojekt

In dieser Aufgabe verwenden Sie den Azure DevOps Demo Generator, um ein neues Projekt basierend auf der Vorlage **PartsUnlimited** zu erstellen.

1. Starten Sie auf Ihrem Laborcomputer einen Webbrowser und navigieren Sie zu [Azure DevOps Demo Generator](https://azuredevopsdemogenerator.azurewebsites.net). Diese Hilfswebsite automatisiert den Prozess des Erstellens eines neuen Azure DevOps-Projekts innerhalb Ihres Kontos, das mit Inhalten (Arbeitsaufgaben, Repositorys usw.) vorgefüllt wird, die für das Labor erforderlich sind.

    > **Anmerkung**: Für weitere Informationen über die Website siehe <https://docs.microsoft.com/en-us/azure/devops/demo-gen>.

1. Klicken Sie auf **Anmelden** und melden Sie sich mit dem Microsoft-Konto an, das mit Ihrem Azure DevOps-Abonnement verbunden ist.
1. Klicken Sie bei Bedarf auf der Seite **Azure DevOps Demo Generator** auf **Annehmen**, um die Berechtigungsanfragen für den Zugriff auf Ihr Azure DevOps-Abonnement zu akzeptieren.
1. Geben Sie auf der Seite **Neues Projekt erstellen** in das Textfeld **Neuer Projektname** den Text **Agentenpools konfigurieren und Pipelinestile verstehen** ein, wählen Sie in der Dropdown-Liste **Organisation auswählen** Ihre Azure DevOps-Organisation aus und klicken Sie dann auf **Vorlage auswählen**.
1. Klicken Sie auf der Seite **Wählen Sie eine Vorlage** auf die Vorlage **PartsUnlimited** und dann auf **Vorlage auswählen**.
1. Klicken Sie auf **Projekt erstellen**

    > **Hinweis**: Warten Sie, bis der Vorgang abgeschlossen ist. Dieser Vorgang dauert etwa zwei Minuten. Sollte der Vorgang fehlschlagen, navigieren Sie zu Ihrer DevOps-Organisation, löschen Sie das Projekt und versuchen Sie es erneut.

1. Klicken Sie auf der Seite **Neues Projekt erstellen** auf **Navigieren zu Projekt**.

### Übung 1: Erstellen von YAML-basierten Azure Pipelines.

In dieser Übung werden Sie eine klassische Azure-Pipeline in eine YAML-basierte umwandeln.

#### Aufgabe 1: Erstellen einer Azure DevOps YAML-Pipeline

In dieser Aufgabe werden Sie eine vorlagenbasierte Azure DevOps YAML-Pipeline erstellen.

1. Klicken Sie im Webbrowser, der das Azure DevOps-Portal anzeigt, bei geöffnetem Projekt **Agentpools konfigurieren und Pipelinestile verstehen**, im vertikalen Navigationsbereich auf der linken Seite auf **Pipelines**.
1. Klicken Sie auf der Registerkarte **Aktuell** des Bereichs **Pipelines** auf **Neue Pipeline**.
1. Klicken Sie im Bereich **Wo befindet sich Ihr Code?** auf **Azure Repos Git**.
1. Klicken Sie im Bereich **Wählen Sie ein Repository** auf **PartsUnlimited**.
1. Überprüfen Sie im Bereich **Überprüfen Sie Ihre Pipeline-YAML** die Beispielpipeline, klicken Sie auf das nach unten zeigende Caret-Symbol neben der Schaltfläche **Ausführen** und dann auf **Speichern**.

### Übung 2: Verwalten von Azure DevOps-Agentpools.

In dieser Übung implementieren Sie einen selbst gehosteten Azure DevOps Agent.

#### Aufgabe 1: Konfigurieren Sie einen selbst gehosteten Azure DevOps-Agent.

In dieser Aufgabe konfigurieren Sie die LOD-VM als Azure DevOps Self-Hosting-Agent und verwenden ihn zum Ausführen einer Buildpipeline.

1. Starten Sie in der Lab Virtual Machine (Lab VM) oder auf Ihrem eigenen Computer einen Webbrowser, navigieren Sie zum [Azure DevOps-Portal](https://dev.azure.com) und melden Sie sich mit dem Microsoft-Konto an, das mit Ihrer Azure DevOps-Organisation verbunden ist.
1. Klicken Sie im Azure DevOps-Portal in der oberen rechten Ecke der Azure DevOps-Seite auf das Symbol **Benutzereinstellungen**. Je nachdem, ob Sie die Vorschaufunktionen aktiviert haben oder nicht, sollten Sie entweder ein Element **Sicherheit** oder **Persönliche Zugriffstoken** im Menü sehen. Wenn Sie **Sicherheit** sehen, klicken Sie darauf und wählen Sie dann **Persönliche Zugriffstoken**. Klicken Sie im Bereich **Personal Access Tokens** auf **+ New Token**.
1. Klicken Sie im Bereich **Neues persönlichen Zugriffstoken erstellen** auf den Link **Alle Bereiche anzeigen** und geben Sie die folgenden Einstellungen an. Klicken Sie dann auf **Erstellen** (bei alle anderen Standardwerte lassen):

    | Einstellung | Wert |
    | --- | --- |
    | Name | **Konfigurieren von Agent-Pools und Verstehen von Pipelinearten** |
    | Anwendungsbereich (benutzerdefiniert) | **Agent Pools** (Option „Weitere Bereiche anzeigen“ unten, falls erforderlich)|
    | Berechtigungen | **Lesen und Verwalten** |

1. Kopieren Sie im Bereich **Erfolg** den Wert des persönlichen Zugriffstokens in die Zwischenablage.

    > **Anmerkung**: Stellen Sie sicher, dass Sie das Token kopieren. Sie können ihn nicht mehr abrufen, nachdem Sie diesen Bereich geschlossen haben.

1. Klicken Sie im Bereich **Erfolg** auf **Schließen**.
1. Klicken Sie im Bereich **Personal Access Token** des Azure DevOps-Portals auf das Symbol **Azure DevOps** in der oberen linken Ecke und dann auf das Label **Organization settings** in der unteren linken Ecke.
1. Klicken Sie auf der linken Seite des Bereichs **Übersicht** im vertikalen Menü im Abschnitt **Pipelines** auf **Agent-Pools**.
1. Klicken Sie im Bereich **Agent-Pools** in der oberen rechten Ecke auf **Pool hinzufügen**.
1. Wählen Sie im Bereich **Agentpoolhinzufügen** in der Dropdown-Liste **Pooltyp** die Option **Selbst gehostet**, geben Sie in das Textfeld **Name** den Wert **az400m05l05a-pool** ein und klicken Sie auf **Erstellen**.
1. Klicken Sie im Bereich **Agentpools** auf den Eintrag für den neu erstellten **az400m05l05a-Pool**.
1. Klicken Sie auf der Registerkarte **Jobs** des Fensters **az400m05l05a-Pool** auf die Schaltfläche **Neuer Agent**.
1. Vergewissern Sie sich im Bereich **Abrufen des Agenten**, dass die Registerkarten **Windows** und **x64** ausgewählt sind, und klicken Sie auf **Herunterladen**, um das Zip-Archiv mit den Agent-Binärdateien in den lokalen Ordner **Downloads** in Ihrem Benutzerprofil herunterzuladen.

    > **Hinweis**: Wenn Sie an dieser Stelle eine Fehlermeldung erhalten, die Sie darauf hinweist, dass die aktuellen Systemeinstellungen das Herunterladen der Datei verhindern, klicken Sie im Fenster des Internet Explorers in der oberen rechten Ecke auf das Zahnradsymbol, das die Kopfzeile des Menüs **Einstellungen** bezeichnet. Im Dropdown-Menü wählen Sie **Internetoptionen**, klicken Sie im Dialogfeld **Internetoptionen** auf **Erweitert**, danach in der Registerkarte **Erweitert** auf **Zurücksetzen** und im Dialogfeld **Internet Explorer-Einstellungen zurücksetzen** erneut auf **Zurücksetzen**. Klicken Sie abschließend auf **Schließen** und versuchen Sie den Download erneut.

1. Starten Sie Windows PowerShell als Administrator*in und führen Sie in der **Administrator: Windows PowerShell**-Konsole die folgenden Zeilen aus, um das Verzeichnis **C:\\agent** zu erstellen und den Inhalt des heruntergeladenen Archivs in dieses zu extrahieren.

    ```powershell
    cd \
    mkdir agent ; cd agent
    $TARGET = Get-ChildItem "$Home\Downloads\vsts-agent-win-x64-*.zip"
    Add-Type -AssemblyName System.IO.Compression.FileSystem
    [System.IO.Compression.ZipFile]::ExtractToDirectory($TARGET, "$PWD")
    ```

1. Führen Sie in der gleichen Konsole**Administrator: Windows PowerShell** Folgendes aus, um den Agenten zu konfigurieren:

    ```powershell
    .\config.cmd
    ```

1. Wenn Sie dazu aufgefordert werden, geben Sie die Werte der folgenden Einstellungen an:

    | Einstellung | Wert |
    | ------- | ----- |
    | Server-URL eingeben | die URL Ihrer Azure DevOps-Organisation im Format **<https://dev.azure.com/>`<organization_name>`**, wobei `<organization_name>` den Namen Ihrer Azure DevOps-Organisation darstellt |
    | Authentifizierungsart eingeben (für PAT die Eingabetaste drücken) | **EINGABETASTE** |
    | Persönliches Zugriffstoken eingeben | Das Zugriffstoken, das Sie zuvor in dieser Aufgabe aufgezeichnet haben |
    | Geben Sie den Agent-Pool ein (drücken Sie die Eingabetaste für die Standardeinstellung) | **az400m05l05a-pool** |
    | Name des Agent eingeben | **az400m05-vm0** |
    | Arbeitsordner eingeben (für _work die Eingabetaste drücken) | **EINGABETASTE** |
    | **(Nur wenn angezeigt)** Führen Sie für jede Aufgabe eines Schritts das Entpacken durch. (drücken Sie die Eingabetaste für N) | **WARNUNG**: nur **Eingabe** drücken, wenn die Meldung angezeigt wird|
    | Agent als Dienst ausführen? (J/N) (drücken Sie die Eingabetaste für N) | **Y** |
    | Eingeben der Aktivierung von SERVICE_SID_TYPE_UNRESTRICTED (J/N) (drücken Sie die Eingabetaste für N) | **Y** |
    | Geben Sie das für den Dienst zu verwendende Benutzerkonto ein (drücken Sie die Eingabetaste für NT AUTHORITY\NETWORK SERVICE) | **EINGABETASTE** |
    | Geben Sie an, ob der Dienst nicht sofort nach Abschluss der Konfiguration gestartet werden soll. (J/N) (drücken Sie die Eingabetaste für N) | **EINGABETASTE** |

    > **Hinweis**: Sie können den selbstgehosteten Agent entweder als Dienst oder als interaktiven Prozess ausführen. Sie sollten mit dem interaktiven Modus beginnen, da dies die Überprüfung der Agent-Funktionalität vereinfacht. Für den produktiven Einsatz sollten Sie in Erwägung ziehen, den Agent entweder als Dienst oder als interaktiven Prozess mit aktivierter automatischer Anmeldung laufen zu lassen, da in beiden Fällen der Status des Agent erhalten bleibt und sichergestellt wird, dass der Agent bei einem Neustart des Betriebssystems automatisch startet.

1. Wechseln Sie zu dem Browserfenster, in dem das Azure DevOps-Portal angezeigt wird, und schließen Sie das Fenster **Agent abrufen**.
1. Zurück auf der Registerkarte **Agenten** des Fensters **az400m05l05a-Pool** beachten Sie, dass der neu konfigurierte Agent mit dem Status **Online** aufgeführt ist.
1. Klicken Sie in dem Webbrowser-Fenster, in dem das Azure DevOps-Portal angezeigt wird, in der oberen linken Ecke auf die Bezeichnung **Azure DevOps**.
1. Klicken Sie im Browserfenster, das die Liste der Projekte anzeigt, auf die Kachel, die Ihr Projekt **Konfigurieren von Agent-Pools und Verstehen von Pipelinestilen** darstellt.
1. Klicken Sie im Bereich **Konfigurieren von Agentenpools und Verstehen von Pipelinestilen** im vertikalen Navigationsbereich auf der linken Seite im Abschnitt **Pipelines** auf **Pipelines**.
1. Wählen Sie auf der Registerkarte **Aktuell** des Bereichs **Pipelines** den Eintrag **PartsUnlimited** und auf dem Bereich **PartsUnlimited** den Eintrag **Bearbeiten**.
1. Ersetzen Sie im Bearbeitungsbereich **PartsUnlimited** in der vorhandenen YAML-basierten Pipeline die Zeile `vmImage: windows-2019`, die den Ziel-Agentenpool bezeichnet, durch den folgenden Inhalt, der den neu erstellten selbst gehosteten Agentenpool bezeichnet:

    ```yaml
    name: az400m05l05a-pool
    demands:
    - agent.name -equals az400m05-vm0
    ```
    > **WARNUNG**: Seien Sie vorsichtig beim Kopieren/Einfügen, stellen Sie sicher, dass Sie die gleiche Einrückung wie oben gezeigt haben.


1. Klicken Sie bei `Task: NugetToolInstaller@0` auf **Einstellungen (Link, der oberhalb des Tasks in grauer Farbe angezeigt wird)**, ändern Sie **Version von NuGet.exe zur Installation von** > **4.0.0** und klicken Sie auf **Hinzufügen**.
1.  Klicken Sie im Bearbeitungsfenster **PartsUnlimited** in der oberen rechten Ecke des Fensters auf **Speichern** und im Fenster **Speichern** erneut auf **Speichern**. Dadurch wird der Build automatisch auf der Grundlage dieser Pipeline ausgelöst.
1.  Klicken Sie im Azure DevOps-Portal im vertikalen Navigationsbereich auf der linken Seite im Abschnitt **Pipelines** auf **Pipelines**.
1.  Klicken Sie auf der Registerkarte **Aktuell** des Bereichs **Pipelines** auf den Eintrag **PartsUnlimited**, wählen Sie auf der Registerkarte **Läufe** des Bereichs **PartsUnlimited** den jüngsten Lauf aus, Scrollen Sie im Bereich **Zusammenfassung** des Laufs nach unten. Klicken Sie danach im Abschnitt **Jobs** auf **Phase 1** und überwachen Sie den Job bis zu seinem erfolgreichen Abschluss.



## Überprüfung

In dieser Übung haben Sie gelernt, wie Sie selbst gehostete Agents mit YAML-Pipelines implementieren und nutzen können.
