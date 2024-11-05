---
lab:
  title: Aktivieren von Continuous Integration mit Azure Pipelines
  module: 'Module 02: Implement CI with Azure Pipelines and GitHub Actions'
---

# Aktivieren von Continuous Integration mit Azure Pipelines

## Labanforderungen

- Für dieses Lab ist **Microsoft Edge** oder ein von [Azure DevOps unterstützter Browser](https://docs.microsoft.com/azure/devops/server/compatibility) erforderlich.

- **Einrichten einer Azure DevOps-Organisation**: Wenn Sie nicht bereits eine Azure DevOps-Organisation haben, die Sie für dieses Lab verwenden können, müssen Sie diese erstellen, indem Sie die unter [Erstellen einer Organisation oder Projektsammlung](https://docs.microsoft.com/azure/devops/organizations/accounts/create-organization) beschriebenen Anweisungen befolgen.

## Übersicht über das Labor

In diesem Lab erfahren Sie, wie Sie Buildpipelines in Azure DevOps mithilfe von YAML definieren.
Die Pipelines werden in zwei Szenarios verwendet:

- Im Rahmen des Überprüfungsprozesses für Pull Requests
- Im Rahmen der Continuous Integration-Implementierung

## Ziele

In diesem Lab lernen Sie Folgendes:

- Einschließen der Buildvalidierung als Teil eines Pull Requests
- Konfigurieren der CI-Pipeline als Code mit YAML

## Geschätzte Zeit: 30 Minuten

## Anweisungen

### Übung 0: (überspringen, falls bereits erledigt) Konfigurieren Sie die Lab-Voraussetzungen

In dieser Übung richten Sie die Voraussetzungen für das Lab ein, das aus einem neuen Azure DevOps-Projekt mit einem Repository basierend auf dem [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb) besteht.

#### Aufgabe 1: (überspringen, wenn fertig) Erstellen und Konfigurieren des Teamprojekts

In dieser Aufgabe erstellen Sie ein **eShopOnWeb** Azure DevOps-Projekt, das von mehreren Labs verwendet werden soll.

1. Öffnen Sie auf Ihrem Lab-Computer in einem Browserfenster Ihre Azure DevOps-Organisation. Klicken Sie auf **Neues Projekt**. Weisen Sie Ihrem Projekt den Namen **eShopOnWeb** zu, und lassen Sie die anderen Felder auf den Standardwerten. Klicken Sie auf **Erstellen**.

#### Aufgabe 2: (überspringen, wenn erledigt) Importieren von eShopOnWeb Git Repository

Bei dieser Aufgabe importieren Sie das eShopOnWeb Git-Repository, das von mehreren Labs verwendet wird.

1. Öffnen Sie auf Ihrem Lab-Computer in einem Browserfenster Ihre Azure DevOps-Organisation und das zuvor erstellte **eShopOnWeb**-Projekt. Klicken Sie auf **Repos > Dateien** , **Importiere ein Repository**. Klicken Sie auf **Importieren**. Fügen Sie im Fenster **Git Repository importieren** die folgende URL <https://github.com/MicrosoftLearning/eShopOnWeb.git> ein, und klicken Sie auf **Importieren**:

1. Das Repository ist wie folgt organisiert:
    - Der Ordner **.ado** enthält Azure DevOps-YAML-Pipelines.
    - Der Ordner **.devcontainer** enthält ein Containersetup für die Entwicklung mithilfe von Containern (entweder lokal in VS Code oder über GitHub Codespaces).
    - Der Ordner **infra** enthält die Bicep- und ARM-Infrastruktur als Codevorlagen, die in einigen Labszenarios verwendet werden.
    - Der Ordner **.github** enthält YAML-GitHub-Workflowdefinitionen.
    - Der Ordner **src** enthält die .NET 6-Website, die in den Labszenarien verwendet wird.

#### Aufgabe 3: (überspringen, wenn erledigt) Legen Sie den Mainbranch als Standardbranch fest

1. Wechseln Sie zu **Repos > Branches**.
1. Bewegen Sie den Mauszeiger auf den **Main**-Branch und klicken Sie dann rechts neben der Spalte auf die Auslassungspunkte.
1. Klicken Sie auf **Als Mainbranch festlegen**.

### Übung 1: Einschließen der Buildvalidierung als Teil einerPull-Anforderung

In dieser Übung fügen Sie die Buildvalidierung ein, um eine Pull-Anforderung zu überprüfen.

#### Aufgabe 1: Importieren der YAML-Builddefinition

In dieser Aufgabe importieren Sie die YAML-Builddefinition, die als Branch-Richtlinie zum Überprüfen der Pull-Anforderungen verwendet wird.

Beginnen wir mit dem Importieren der Buildpipeline mit dem Namen [eshoponweb-ci-pr.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-ci-pr.yml).

1. Wechseln Sie zu **Pipelines > Pipelines**
1. Klicken Sie auf die Schaltfläche **Pipeline erstellen** oder **Neue Pipeline**
1. Wählen Sie **Azure Repos Git (YAML)** aus.
1. Wählen Sie das **eShopOnWeb**-Repository
1. Wählen Sie die Option **Vorhandene Azure Pipelines-YAML-Datei** aus
1. Wählen Sie die **Haupt**-Verzweigung und die Datei **/.ado/eshoponweb-ci-pr.yml** aus, und klicken Sie dann auf **Weiter**.

    Die Buildpipeline besteht aus den folgenden Aufgaben:
    - **DotNet Restore**: Mit NuGetPackage Restore können Sie alle Abhängigkeiten Ihres Projekts installieren, ohne sie in der Quellcodeverwaltung speichern zu müssen.
    - **DotNet Build**: Erstellt ein Projekt und alle seine Abhängigkeiten
    - **DotNet Test**: .NET-Testtreiber, der verwendet wird, um Komponententests auszuführen.
    - **DotNet Publish**: Veröffentlicht die Anwendung und ihre Abhängigkeiten in einem Ordner für die Bereitstellung auf einem Hostsystem. In diesem Fall ist das **Build.ArtifactStagingDirectory**.

1. Klicken Sie auf die Schaltfläche **Speichern**, um Ihre Pipelinedefinition zu speichern.
1. Ihre Pipeline bekommt einen Namen basierend auf dem Projektnamen. Wir **benennen sie um**, damit wir die Pipeline besser identifizieren können. Wechseln Sie zu **Pipelines > Pipelines**, und klicken Sie auf die kürzlich erstellte Pipeline. Klicken Sie auf die Auslassungspunkte und die Option **Umbenennen/Verschieben**. Nennen Sie es **eshoponweb-ci-pr** und klicken Sie auf **Speichern**.

#### Aufgabe 2: Branchrichtlinien

In dieser Aufgabe fügen Sie dem Mainbranch Richtlinien hinzu und lassen nur Änderungen mithilfe von Pull Requests zu, die den definierten Richtlinien entsprechen. Sie möchten sicherstellen, dass Änderungen in einem Branch überprüft werden, bevor sie zusammengeführt werden.

1. Wechseln Sie zum Abschnitt **Repos > Branches**.
1. Zeigen Sie auf der Registerkarte **Eigene** im Bereich **Verzweigungen** mit dem Mauszeiger auf den Eintrag **Haupt**-Verzweigung, um das Auslassungszeichen auf der rechten Seite anzuzeigen.
1. Klicken Sie auf die Auslassungspunkte, und wählen Sie im Popupmenü **Verzweigungsrichtlinien** aus.
1. Aktivieren Sie auf der Registerkarte **Haupt** der Repositoryeinstellungen die Option für **Mindestanzahl der Prüfer erforderlich machen**. Fügen Sie **1** Prüfer hinzu, und aktivieren Sie das Kontrollkästchen **Anforderern erlauben, ihre eigenen Änderungen zu genehmigen** (da Sie der einzige Benutzer in Ihrem Projekt für das Lab sind)
1. Klicken Sie auf der Registerkarte **Haupt** der Repositoryeinstellungen im Abschnitt **Buildüberprüfung** auf **+**(Neue Buildrichtlinie hinzufügen), wählen Sie in der Liste **Build-Pipeline** die Option **eshoponweb-ci-pr** aus, und klicken Sie dann auf **Speichern**.

#### Aufgabe 3: Arbeiten mit Pull Requests

In dieser Aufgabe verwenden Sie das Azure DevOps-Portal, um einen Pull Request zu erstellen, indem Sie eine neue Verzweigung verwenden, um eine Änderung mit der geschützten **Hauptverzweigung** zusammenzuführen.

1. Navigieren Sie im eShopOnWeb-Navigationsbereich zum Abschnitt **Repositorys** und klicken Sie auf **Verzweigungen**.
1. Erstellen Sie eine neue Verzweigung mit dem Namen **Feature01** basierend auf der **Hauptverzweigung**.
1. Klicken Sie auf **Feature01** und navigieren Sie innerhalb des **Feature01-Branches** zur Datei **/eShopOnWeb/src/Web/Program.cs**.
1. Klicken Sie rechts oben auf die Schaltfläche **Bearbeiten**.
1. Nehmen Sie die folgende Änderung an der ersten Zeile vor:

    ```csharp
    // Testing my PR
    ```

1. Klicken Sie auf **Commit > Commit** (Standard-Commit-Nachricht belassen).
1. Eine Nachricht wird eingeblendet und schlägt vor, einen Pull Request zu erstellen (da Ihr **Feature01**-Branch im Vergleich zum **Mainbranch** aktuellere Änderungen enthält). Klicken Sie auf **Pull Request erstellen**.
1. Belassen Sie die Standardeinstellungen auf der Registerkarte **Neuer Pull Request** und klicken Sie auf **Erstellen**.
1. Der Pull Request zeigt einige ausstehende Anforderungen basierend auf den Richtlinien an, die für die Ziel-**Hauptverzweigung** gelten.
    - Mindestens 1 Benutzer*in sollte die Änderungen überprüfen und genehmigen.
    - Buildüberprüfung, Sie werden sehen, dass der Build **eshoponweb-ci-pr** automatisch ausgelöst wurde

1. Nachdem alle Überprüfungen erfolgreich waren, klicken Sie oben mit der rechten Maustaste auf **Genehmigen**. Jetzt können Sie im Dropdownmenü **AutoVervollständigen festlegen** auf **Vervollständigen** klicken.
1. Klicken Sie auf der Registerkarte **Vollständiger Pull Request** auf **Zusammenführen abschließen**.

### Übung 2: Konfigurieren der CI-Pipeline als Code mit YAML

In dieser Übung werden Sie eine CI Pipeline als Code mit YAML konfigurieren.

#### Aufgabe 1: Importieren der YAML-Builddefinition für CI

In dieser Aufgabe fügen Sie die YAML-Builddefinition hinzu, die zum Implementieren der Continuous Integration verwendet wird.

Beginnen wir mit dem Importieren der CI-Pipeline mit dem Namen [eshoponweb-ci.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-ci.yml).

1. Navigieren Sie zu **Pipelines > Pipelines**.
1. Klicken Sie auf die Schaltfläche **Neue Pipeline**.
1. Wählen Sie **Azure Repos Git** (YAML) aus.
1. Wählen Sie das Repository **eShopOnWeb** aus.
1. Wählen Sie die Option **Vorhandene Azure Pipelines-YAML-Datei** aus.
1. Wählen Sie die **Haupt**-Verzweigung und die Datei **/.ado/eshoponweb-ci.yml** aus, und klicken Sie dann auf **Weiter**.

    Die CI-Definition umfasst die folgenden Aufgaben:
    - **DotNet Restore**: Mit NuGetPackage Restore können Sie alle Abhängigkeiten Ihres Projekts installieren, ohne sie in der Quellcodeverwaltung speichern zu müssen.
    - **DotNet Build**: Erstellt ein Projekt und alle seine Abhängigkeiten
    - **DotNet Test**: .NET-Testtreiber, der verwendet wird, um Komponententests auszuführen.
    - **DotNet Publish**: Veröffentlicht die Anwendung und ihre Abhängigkeiten in einem Ordner für die Bereitstellung auf einem Hostsystem. In diesem Fall ist das **Build.ArtifactStagingDirectory**.
    - **Artefakt veröffentlichen – Website**: Veröffentlichen Sie das App-Artefakt (erstellt im vorherigen Schritt), und stellen Sie es als Pipelineartefakt zur Verfügung.
    - **Artefakt veröffentlichen – Bicep**: Veröffentlichen Sie das Infrastrukturartefakt (Bicep-Datei), und stellen Sie es als Pipelineartefakt zur Verfügung.

1. Klicken Sie auf **Ausführen** und warten Sie, bis die Pipeline erfolgreich ausgeführt wird.

#### Aufgabe 2: Aktivieren von Continuous Integration

Die Standardmäßige Buildpipelinedefinition aktiviert keine Continuous Integration.

1. Klicken Sie rechts oben auf die Schaltfläche **Bearbeiten**.
1. Jetzt müssen Sie die Zeilen **#-trigger:** und **# - main** ersetzen durch den folgenden Code:

    ```YAML
    trigger:
      branches:
        include:
        - main
      paths:
        include:
        - src/web/*
    ```

    Dadurch wird die Buildpipeline automatisch ausgelöst, wenn Änderungen an der Hauptverzweigung und dem Webanwendungscode (src/web folder) vorgenommen werden.

    Da Sie Verzweigungsrichtlinien aktiviert haben, müssen Sie von einem Pull Request übergeben werden, um Ihren Code zu aktualisieren.

1. Klicken Sie auf die Schaltfläche **Speichern** (nicht **Speichern und ausführen**), um die Pipelinedefinition zu speichern.
1. Wählen Sie **Für diesen Commit neuen Branch erstellen**.
1. Übernehmen Sie den Standardnamen der Verzweigung und belassen Sie **Pull Request starten** markiert.
1. Klicken Sie auf **Speichern**.
1. Ihre Pipeline bekommt einen Namen basierend auf dem Projektnamen. Wir **benennen sie um**, damit wir die Pipeline besser identifizieren können. Wechseln Sie zu **Pipelines > Pipelines**, und klicken Sie auf die kürzlich erstellte Pipeline. Klicken Sie auf die Auslassungspunkte und die Option **Umbenennen/Verschieben**. Nennen Sie ihn **eshoponweb-ci** und klicken Sie auf **Speichern**.
1. Gehen Sie zu **Repos > Pull Requests**.
1. Klicken Sie auf die Pull Request **„eshoponweb-ci.yml für Azure Pipelines aktualisieren“**.
1. Nachdem alle Überprüfungen erfolgreich waren, klicken Sie oben mit der rechten Maustaste auf **Genehmigen**. Jetzt können Sie auf **Abschließen** klicken.
1. Klicken Sie auf der Registerkarte **Pull Request abschließen** auf **Zusammenführen abschließen**.

#### Aufgabe 3: Testen der CI-Pipeline

In dieser Aufgabe erstellen Sie einen Pull Request mit einem neuen Branch, um eine Änderung mit dem geschützten **Mainbranch** zusammenzuführen und die CI-Pipeline automatisch auszulösen.

1. Navigieren Sie zum Abschnitt **Repos** und klicken Sie auf **Verzweigungen**.
1. Erstellen Sie eine neue Verzweigung mit dem Namen **Feature02** basierend auf der **Haupt**-Verzweigung.
1. Klicken Sie auf die neue Verzweigung **Feature02**.
1. Navigieren Sie zur Datei **/eShopOnWeb/src/Web/Program.cs**, und klicken Sie oben rechts auf **Bearbeiten**.
1. Entfernen Sie die erste Zeile:

    ```csharp
    // Testing my PR
    ```

1. Klicken Sie auf **Commit > Commit** (Standard-Commit-Nachricht belassen).
1. Eine Nachricht wird angezeigt und schlägt vor, einen Pull Request zu erstellen (da Ihre Verzweigung **Feature02** im Vergleich zur **auptverzweigung** jetzt akuellere Änderungen enthält).
1. Klicken Sie auf **Pull Request erstellen**.
1. Belassen Sie die Standardeinstellungen auf der Registerkarte **Neuer Pull Request** und klicken Sie auf **Erstellen**.
1. Der Pull Request zeigt einige ausstehende Anforderungen basierend auf den Richtlinien an, die für die Ziel-**Hauptverzweigung** gelten.
1. Nachdem alle Überprüfungen erfolgreich waren, klicken Sie oben mit der rechten Maustaste auf **Genehmigen**. Jetzt können Sie im Dropdownmenü **AutoVervollständigen festlegen** auf **Vervollständigen** klicken.
1. Klicken Sie auf der Registerkarte **Pull Request abschließen** auf **Zusammenführen abschließen**.
1. Wechseln Sie zurück zu **Pipelines > Pipelines**. Sie werden feststellen, dass der Build **eshoponweb-ci** automatisch ausgelöst wurde, nachdem der Code zusammengeführt wurde.
1. Klicken Sie auf den **eshoponweb-ci** Build, und wählen Sie dann die letzte Ausführung aus.
1. Klicken Sie nach erfolgreicher Ausführung auf **Zugehörige > Veröffentlicht**, um die veröffentlichten Artefakte zu überprüfen:
    - Bicep: das Infrastrukturartefakt.
    - Website: das App-Artefakt.

## Überprüfung

In dieser Übung haben Sie die Überprüfung des Pull Request mithilfe einer Builddefinition und konfigurierter CI-Pipeline als Code mit YAML in Azure DevOps aktiviert.
