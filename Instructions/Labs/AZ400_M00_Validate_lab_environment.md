---
lab:
  title: Überprüfen der Umgebung
  module: 'Module 0: Welcome'
---

# Überprüfen der Umgebung

## Lab-Handbuch für Kursteilnehmer

## Anweisungen zum Erstellen einer Azure DevOps-Organisation (Sie müssen dies nur einmal tun)

### Vorgehensweise ohne Azure-Abonnement
1. Rufen Sie einen neuen **Azure Pass Promocode** vom Kursleiter oder einer anderen Quelle ab.
1. Verwenden Sie eine private Browsersitzung, um ein neues **persönliches Microsoft-Konto (MSA)** zu erhalten unterhttps://account.microsoft.com[](https://account.microsoft.com) .
1. Wechseln Sie mit derselben Browsersitzung zum [https://www.microsoftazurepass.com](https://www.microsoftazurepass.com) Einlösen Ihres Azure Pass mithilfe Ihres Microsoft-Kontos (MSA). Ausführliche Informationen finden Sie unter [Einlösen eines Microsoft Azure Pass](https://www.microsoftazurepass.com/Home/HowTo?Length=5). Befolgen Sie die Anweisungen zur Korrektur.

### Vorgehensweise mit Azure-Abonnement

1. Öffnen Sie einen Browser, und navigieren Sie zu [https://portal.azure.com](https://portal.azure.com), und suchen Sie dann am oberen Rand des Azure-Portal Bildschirms nach **Azure DevOps.** Klicken Sie auf der resultierenden Seite auf **Azure DevOps-Organisationen**.
1. Klicken Sie als Nächstes auf den Link mit der Bezeichnung **"Meine Azure DevOps-Organisationen**", oder navigieren Sie direkt zuhttps://aex.dev.azure.com[](https://aex.dev.azure.com) .
1. Wählen Sie auf der **Seite "Weitere Details**" die Option "Weiter"** aus**.
1. Wählen Sie **im Dropdownfeld auf der linken Seite "Standardverzeichnis**" anstelle von "Microsoft-Konto" aus.
1. Wenn Sie dazu aufgefordert werden (*"Wir benötigen einige weitere Details"*), geben Sie Ihren Namen, Ihre E-Mail-Adresse und Ihren Standort an, und klicken Sie auf " **Weiter"**.
1. Zurück mit [https://aex.dev.azure.com**](https://aex.dev.azure.com) ausgewähltem Standardverzeichnis** klicken Sie auf die blaue Schaltfläche "**Neue Organisation** erstellen".
1. Akzeptieren Sie die *Nutzungsbedingungen*, indem Sie auf "Weiter"** klicken**.
1. Wenn Sie dazu aufgefordert werden (*"Fast fertig"*), behalten Sie den Namen der Azure DevOps-Organisation standardmäßig bei (es muss sich um einen global eindeutigen Namen handeln), und wählen Sie einen Hostingspeicherort in der Nähe der Liste aus.
1. Nachdem die neu erstellte Organisation in **Azure DevOps** geöffnet wurde, klicken Sie in der unteren linken Ecke auf **"Organisationseinstellungen** ".
1. Klicken Sie auf dem **Bildschirm "Organisationseinstellungen** " auf **"Abrechnung"** (das Öffnen dieses Bildschirms dauert einige Sekunden).
1. Klicken Sie auf "Abrechnung einrichten"**, und wählen Sie **auf der rechten Seite des Bildschirms das **Azure Pass - Sponsoring-Abonnement** aus, und klicken Sie auf "**Speichern**", um das Abonnement mit der Organisation zu verknüpfen.
1. Sobald der Bildschirm die verknüpfte Azure-Abonnement-ID oben anzeigt, ändern Sie die Anzahl der **kostenpflichtigen parallelen Aufträge** für **MS Hosted CI/CD** von 0 auf **1**. Klicken Sie dann auf die Schaltfläche  unten.
1. **Wechseln Sie in** der Organisation Einstellungen zu Abschnittspipelinen****, und klicken Sie auf **Einstellungen**.
1. Aktivieren Sie im Abschnitt **Allgemein** die Optionen **Deaktivieren der Erstellung von klassischen Buildpipelines** und **Deaktivieren der Erstellung von klassischen Releasepipelines**.
    > Hinweis: Die **Option "Deaktivieren der Erstellung von klassischen Releasepipelinen"** wird auf **"Ein** " festgelegt, um optionen für die Klassische Releasepipelineerstellung wie das **Menü "Release"** im **Abschnitt "Pipeline** " von DevOps-Projekten auszublenden.
1. **Wechseln Sie in "Organisations-Einstellungen**" zum Abschnitt **"Sicherheit**", und klicken Sie auf "**Richtlinien"**.
1. Umschalten des Schalters auf **"Ein** " für **den Anwendungszugriff von Drittanbietern über OAuth**
    > Hinweis: Mit der OAuth-Einstellung können Sie Tools wie demoDevOpsGenerator zum Registrieren von Erweiterungen aktivieren. Ohne dies können mehrere Labore aufgrund eines Mangels an erforderlichen Erweiterungen fehlschlagen.
1. Umschalten des Schalters auf **"Ein** " für **öffentliche Projekte zulassen**
    > Hinweis: Erweiterungen, die in einigen Laboren verwendet werden, erfordern möglicherweise ein öffentliches Projekt, um die Verwendung der kostenlosen Version zuzulassen.
1. **Warten Sie mindestens 3 Stunden, bevor Sie die CI/CD-Funktionen** verwenden, damit die neuen Einstellungen im Back-End wiedergegeben werden. Andernfalls wird weiterhin die Meldung *"Es wurde keine gehostete Parallelität gekauft oder gewährt"* angezeigt.

## Anweisungen zum Erstellen des Azure DevOps-Beispielprojekts (Sie müssen dies nur einmal tun)

### Übung 0: Konfigurieren der Voraussetzungen für das Lab.

> **Hinweis**: Stellen Sie sicher, dass Sie die Schritte zum Erstellen Ihrer Azure DevOps-Organisation abgeschlossen haben, bevor Sie mit diesen Schritten fortfahren.

In dieser Übung richten Sie die Voraussetzungen für das Labor ein, das aus einem neuen Azure DevOps-Projekt mit einem Repository basierend auf dem [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb) besteht.

#### Erstellen und Konfigurieren des Projekts

In dieser Aufgabe erstellen Sie ein **eShopOnWeb** Azure DevOps-Projekt, das von mehreren Laboren verwendet werden soll.

1. Öffnen Sie auf Ihrem Laborcomputer in einem Browserfenster Ihre Azure DevOps-Organisation. Klicken auf „Neues Projekt“ Geben Sie Ihrem Projekt die folgenden Einstellungen:
    - Name: eShopOnWeb
    - Sichtbarkeit von Druckern
    - Erweitert: Versionssteuerung: **Git**
    - Erweitert: Arbeitsaufgabenprozess: **Scrum**

2. Klicken Sie auf **Erstellen**.

    ![Erstellen eines Projekts](images/create-project.png)

#### Aufgabe 2: Importieren des eShopOnWeb Git-Repositorys

Bei dieser Aufgabe importieren Sie das eShopOnWeb Git-Repository, das von mehreren Labs verwendet wird.

1. Öffnen Sie auf Ihrem Laborcomputer in einem Browserfenster Ihre Azure DevOps-Organisation und das zuvor erstellte **eShopOnWeb-Projekt** . Klicken Sie auf **Repos>Files** , **Import a Repository**. Wählen Sie **Importieren** aus. Fügen Sie im **Fenster "Git Repository** importieren" die folgende URL https://github.com/MicrosoftLearning/eShopOnWeb.git  ein, und klicken Sie auf " **Importieren**":

    ![Importieren eines Repositorys](images/import-repo.png)

2. Das Repository ist wie folgt organisiert:
    - Der Ordner „.ado“ enthält Azure DevOps-YAML-Pipelines.
    - Der Ordner „.devcontainer“ enthält ein Containersetup für die Entwicklung mithilfe von Containern (entweder lokal in VS Code oder über GitHub Codespaces).
    - **Azure-Ordner** enthält Bicep&ARM-Infrastruktur als Codevorlagen, die in einigen Lab-Szenarien verwendet werden.
    - **GITHUB-Ordnercontainer-YAML-GitHub-Workflowdefinitionen** .
    - Der Ordner „src“ enthält die .NET 6-Website, die in den Labszenarien verwendet wird.

Sie haben nun die erforderlichen Erforderlichen Schritte abgeschlossen, um mit den verschiedenen Einzellaboren für diesen AZ-400-Kurs fortzufahren.
