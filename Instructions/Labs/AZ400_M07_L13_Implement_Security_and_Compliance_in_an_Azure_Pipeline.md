---
lab:
  title: Implementieren von Sicherheit und Compliance in einer Azure DevOps-Pipeline
  module: 'Module 07: Implement security and validate code bases for compliance'
---

# Implementieren von Sicherheit und Compliance in einer Azure DevOps-Pipeline

## Lab-Handbuch für Kursteilnehmer

## Labanforderungen

- Für dieses Lab ist **Microsoft Edge** oder ein von [Azure DevOps unterstützter Browser](https://learn.microsoft.com/azure/devops/server/compatibility) erforderlich.

- **Einrichten einer Azure DevOps-Organisation**: Wenn Sie nicht bereits eine Azure DevOps-Organisation haben, die Sie für dieses Lab verwenden können, müssen Sie diese erstellen, indem Sie die unter [Erstellen einer Organisation oder Projektsammlung](https://learn.microsoft.com/azure/devops/organizations/accounts/create-organization) beschriebenen Anweisungen befolgen.

## Übersicht über das Labor

In diesem Lab verwenden Sie **Mend Bolt (früher WhiteSource)**, um anfällige Open-Source-Komponenten, veraltete Bibliotheken und Probleme mit der Lizenzeinhaltung in Ihrem Code automatisch zu erkennen. Sie verwenden WebGoat, eine absichtlich unsichere, von OWASP verwaltete Webanwendung, um allgemeine Sicherheitsprobleme der Webanwendung zu veranschaulichen.

[Mend](https://www.mend.io/) ist führend im Bereich der kontinuierlichen Open-Source-Software-Sicherheits- und Complianceverwaltung. WhiteSource lässt sich unabhängig von Ihren Programmiersprachen, Buildtools oder Entwicklungsumgebungen in Ihren Buildprozess integrieren. Es arbeitet automatisch, kontinuierlich und unauffällig im Hintergrund und überprüft die Sicherheit, Lizenzierung und Qualität Ihrer Open-Source-Komponenten anhand der von WhiteSource ständig aktualisierten Datenbank mit Open-Source-Repositorys.

Mend bietet mit Mend Bolt, eine schlanke Open-Source-Sicherheits- und Verwaltungslösung, die speziell für die Integration mit Azure DevOps und Azure DevOps Server entwickelt wurde. Beachten Sie, dass Mend Bolt pro Projekt arbeitet und keine Echtzeit-Warnfunktionen bietet, wofür die **Vollständige Plattform** erforderlich ist, die im Allgemeinen für größere Entwicklungsteams empfohlen wird, die ihre Open-Source-Verwaltung über den gesamten Lebenszyklus der Softwareentwicklung (von den Repositories bis zu den Phasen nach der Bereitstellung) und über alle Projekte und Produkte hinweg automatisieren möchten.

Die Azure DevOps-Integration in Mend Bolt ermöglicht Ihnen Folgendes:

- Erkennen und Behandeln anfälliger Open-Source-Komponenten.
- Generieren umfassender Open-Source-Bestandsberichte pro Projekt oder Build.
- Erzwingen der Open-Source-Lizenzkompatibilität, einschließlich der Lizenzen von Abhängigkeiten.
- Identifizieren veralteter Open-Source-Bibliotheken mit Aktualisierungsempfehlungen.

## Ziele

In diesem Lab lernen Sie Folgendes:

- Aktivieren von Mend Bolt.
- Ausführen einer Buildpipeline und Überprüfen des Mend-Sicherheits- und Complianceberichts.

## Geschätzte Dauer: 45 Minuten

## Anweisungen

### Übung 0: Konfigurieren der Voraussetzungen für das Lab

In dieser Übung richten Sie die Voraussetzungen für das Lab ein, das aus einem neuen Azure DevOps-Projekt mit einem Repository basierend auf dem [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb) besteht.

#### Aufgabe 1: (überspringen, wenn fertig) Erstellen und Konfigurieren des Teamprojekts

In dieser Aufgabe erstellen Sie ein **eShopOnWeb** Azure DevOps-Projekt, das von mehreren Labs verwendet werden soll.

1. Öffnen Sie auf Ihrem Lab-Computer in einem Browserfenster Ihre Azure DevOps-Organisation. Klicken Sie auf **Neues Projekt**. Weisen Sie Ihrem Projekt den Namen **eShopOnWeb** zu, und lassen Sie die anderen Felder auf den Standardwerten. Klicken Sie auf **Erstellen**.

    ![Erstellen eines Projekts](images/create-project.png)

#### Aufgabe 2: (überspringen, wenn erledigt) Importieren von eShopOnWeb Git Repository

Bei dieser Aufgabe importieren Sie das eShopOnWeb Git-Repository, das von mehreren Labs verwendet wird.

1. Öffnen Sie auf Ihrem Lab-Computer in einem Browserfenster Ihre Azure DevOps-Organisation und das zuvor erstellte **eShopOnWeb**-Projekt. Klicken Sie auf **Repos>Files**, **Importieren**. Fügen Sie im Fenster **Git Repository importieren** die folgende URL <https://github.com/MicrosoftLearning/eShopOnWeb.git> ein, und klicken Sie auf **Importieren**:

    ![Importieren eines Repositorys](images/import-repo.png)

1. Das Repository ist wie folgt organisiert:
    - Der Ordner **.ado** enthält Azure DevOps-YAML-Pipelines.
    - Der Ordner **.devcontainer** enthält ein Containersetup für die Entwicklung mithilfe von Containern (entweder lokal in VS Code oder über GitHub Codespaces).
    - Der Ordner **infra** enthält eine Bicep&ARM-Infrastruktur als Codevorlagen, die in einigen Labszenarien verwendet werden.
    - Der Ordner **.github** enthält YAML GitHub-Workflow-Definitionen.
    - Der Ordner **src** enthält die .NET 8-Website, die in den Labszenarios verwendet wird.

### Übung 1: Implementieren von Sicherheit und Compliance in einer Azure DevOps-Pipeline mithilfe von Mend Bolt

Nutzen Sie in dieser Übung Mend Bolt, um den Projektcode auf Sicherheitsrisiken und Compliance-Probleme mit der Lizenzierung zu überprüfen und den resultierenden Bericht anzuzeigen.

#### Aufgabe 1: Aktivieren der Mend Bolt-Erweiterung

In dieser Aufgabe aktivieren Sie WhiteSource Bolt im neu generierten Azure Devops-Projekt.

1. Klicken Sie auf Ihrem Lab-Computer im Webbrowserfenster, in dem das Azure DevOps-Portal mit geöffnetem **eShopOnWeb**-Projekt angezeigt wird, auf das Marketplace-Symbol > **Marketplace durchsuchen**.

    ![Marketplace durchsuchen](images/browse-marketplace.png)

1. Suchen Sie auf dem MarketPlace nach **Mend Bolt (ehemals WhiteSource),** und öffnen Sie es. Mend Bolt ist die kostenlose Version des zuvor bekannten WhiteSource-Tools, das alle Ihre Projekte überprüft und Open Source-Komponenten, deren Lizenz und bekannte Sicherheitsrisiken erkennt.

    > Warnung: Stellen Sie sicher, dass Sie die Mend **Bolt**-Option (die **kostenlose**) auswählen!

1. Klicken Sie auf der Seite **Mend Bolt (ehemals WhiteSource)** auf **Kostenlos abrufen**.

    ![Mend Bolt abrufen](images/mend-bolt.png)

1. Wählen Sie auf der nächsten Seite die gewünschte Azure DevOps-Organisation aus und **Installieren**. **Gehen Sie nach der Installation zur Organisation**.

1. Navigieren Sie in Azure DevOps zu **Organisationseinstellungen** und wählen Sie **Mend** unter **Erweiterungen** aus. Geben Sie Ihre geschäftliche E-Mail (**Ihr persönliches Konto im Lab**, z. B. <AZ400learner@outlook.com> anstelle von <student@microsoft.com>) Firmenname und andere Details an, und klicken Sie auf die Schaltfläche **Konto erstellen**, um mit der kostenlosen Version zu beginnen.

    ![Mend-Konto abrufen](images/mend-account.png)

#### Aufgabe 2: Erstellen und Auslösen eines Builds

In dieser Aufgabe erstellen Sie eine CI-Buildpipeline innerhalb des Azure DevOps-Projekts und starten diese. Die **Mend Bolt**-Erweiterung verwenden Sie, um anfällige OSS-Komponenten zu identifizieren, die in diesem Code vorhanden sind.

1. Navigieren Sie auf Ihrem Lab-Computer vom Azure DevOps-Projekt **eShopOnWeb** in der vertikalen Menüleiste auf der linken Seite zum Abschnitt **Pipelines>Pipelines**, klicken Sie auf **Pipeline erstellen** (oder **Neue Pipeline**).

1. Im Fenster **Wo befindet Sich Ihr Code?** wählen Sie **Azure Repos Git (YAML)** aus. Wählen Sie dann das **eShopOnWeb-Repository** aus.

1. Wählen Sie auf der Registerkarte **Konfigurieren** die Option **Vorhandene Azure Pipelines YAML-Datei** aus. Verzweigung auswählen: **Haupt**, geben Sie den Pfad **/.ado/eshoponweb-ci-mend.yml** an, und klicken Sie auf **Weiter**.

    ![Wählen Sie Pipeline aus.](images/select-pipeline.png)

1. Überprüfen Sie die Pipeline, und klicken Sie auf **Ausführen**. Die Ausführung dauert einige Minuten.
    > **Hinweis**: Es kann einige Minuten dauern, bis der Build abgeschlossen ist. Die Buildpipeline besteht aus den folgenden Aufgaben:
    - **DotnetCLI**-Aufgabe zum Wiederherstellen, Build-Erstellen, Testen und Veröffentlichen des Dotnet-Projekts.
    - **Whitesource**-Aufgabe (behält weiterhin den alten Namen bei), um die Mend-Toolanalyse von OSS-Bibliotheken auszuführen.
    - **Artefakte veröffentlichen**: die Agents, die diese Pipeline ausführen, laden das veröffentlichte Webprojekt hoch.

1. Während die Pipeline ausgeführt wird, können Sie sie **umbenennen** , um sie einfacher zu identifizieren (da das Projekt für mehrere Labs verwendet werden kann). Wechseln Sie zum Abschnitt **Pipelines/Pipelines** im Azure DevOps-Projekt, klicken Sie auf den Namen der ausgeführten Pipeline (er erhält einen Standardnamen), und suchen Sie im Symbol mit den Auslassungszeichen nach **Umbenennen/Verschieben**. Benennen Sie sie in **eshoponweb-ci-mend** um, und klicken Sie auf **Speichern**.

    ![Benennen Sie die Pipeline um.](images/rename-pipeline.png)

1. Nachdem die Pipelineausführung abgeschlossen ist, können Sie die Ergebnisse überprüfen. Öffnen Sie die neueste Ausführung für die Pipeline **eshoponweb-ci-mend**. Auf der Registerkarte Zusammenfassung werden die Protokolle der Ausführung zusammen mit verwandten Details angezeigt, z. B. verwendeter Repository-Version (Commit), Triggertyp, veröffentlichte Artefakte, Testabdeckung usw.

1. Auf der Registerkarte **Mend Bolt** können Sie die OSS-Sicherheitsanalyse überprüfen. Sie zeigt Ihnen Details zu dem verwendeten Bestand, zu gefundenen Sicherheitsrisiken (und zur Lösung) und einen interessanten Bericht zu bibliotheksbezogenen Lizenzen. Nehmen Sie sich etwas Zeit, um den Bericht zu überprüfen.

    ![Mend-Egebnisse](images/mend-results.png)

## Überprüfung

In dieser Übung werden Sie **Mend Bolt mit Azure DevOps** verwenden, um automatisch verwundbare Open-Source-Komponenten, veraltete Bibliotheken und Lizenzkonformitätsprobleme in Ihrem Code zu erkennen.
