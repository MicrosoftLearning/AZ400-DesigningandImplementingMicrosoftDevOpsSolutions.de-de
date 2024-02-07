---
lab:
  title: Überprüfen der Laborumgebung
  module: 'Module 0: Welcome'
---

# Überprüfen der Laborumgebung

## Lab-Handbuch für Kursteilnehmer

## Anweisungen zum Erstellen einer Azure DevOps-Organisation (Sie müssen dies nur einmal tun)

### Beginnen Sie hier, wenn Sie noch kein Azure-Abonnement haben:
1. Holen Sie sich einen neuen **Azure Pass Promocode** von der*m Kursleiter*in oder einer anderen Quelle.
1. Verwenden Sie eine private Browsersitzung, um ein neues **persönliches Microsoft-Konto (MSA)** unter [https://account.microsoft.com](https://account.microsoft.com) zu erhalten.
1. Gehen Sie in derselben Browsersitzung zu [https://www.microsoftazurepass.com](https://www.microsoftazurepass.com), um Ihren Azure-Pass über Ihr Microsoft-Konto (MSA) einzulösen. Weitere Informationen finden Sie unter [Einlösen eines Microsoft Azure-Passes](https://www.microsoftazurepass.com/Home/HowTo?Length=5). Befolgen Sie die Anweisungen zur Einlösung.

### Beginnen Sie hier, wenn Sie ein Azure-Abonnement haben:

1. Öffnen Sie einen Browser und navigieren Sie zu [https://portal.azure.com](https://portal.azure.com), dann suchen Sie oben auf dem Azure-Portal nach **Azure DevOps**. Klicken Sie auf der resultierenden Seite auf **Azure DevOps-Organisationen**.
1. Klicken Sie anschließend auf den Link **Meine Azure DevOps-Organisationen** oder navigieren Sie direkt zu [https://aex.dev.azure.com](https://aex.dev.azure.com).
1. Wählen Sie auf der Seite **Wir brauchen noch ein paar Details** die Option **Fortfahren**.
1. Wählen Sie in der Dropdown-Box links **Standardverzeichnis** anstelle von „Microsoft-Konto“.
1. Wenn Sie dazu aufgefordert werden (*„Wir brauchen noch ein paar Angaben“*), geben Sie Ihren Namen, Ihre E-Mail-Adresse und Ihren Standort an und klicken Sie auf **Fortfahren**.
1. Zurück bei [https://aex.dev.azure.com](https://aex.dev.azure.com) mit der Auswahl **Standardverzeichnis** klicken Sie auf die blaue Schaltfläche **Neue Organisation erstellen**.
1. Akzeptieren Sie die *Nutzungsbedingungen*, indem Sie auf **Fortfahren** klicken.
1. Wenn Sie dazu aufgefordert werden (*„Fast fertig“*), lassen Sie den Namen für die Azure DevOps-Organisation auf dem Standardwert (es muss ein global eindeutiger Name sein) und wählen Sie einen Hosting-Standort in Ihrer Nähe aus der Liste aus.
1. Sobald die neu erstellte Organisation in **Azure DevOps** geöffnet ist, klicken Sie auf **Organisationseinstellungen** in der linken unteren Ecke.
1. Klicken Sie auf dem Bildschirm **Organisationseinstellungen** auf **Abrechnung** (das Öffnen dieses Bildschirms dauert ein paar Sekunden).
1. Klicken Sie auf **Abrechnung einrichten** und wählen Sie auf der rechten Seite des Bildschirms das Abonnement **Azure Pass – Sponsoring** aus und klicken Sie auf **Speichern**, um das Abonnement mit der Organisation zu verknüpfen.
1. Sobald der Bildschirm die verknüpfte Azure-Abonnement-ID oben anzeigt, ändern Sie die Anzahl der **bezahlten parallelen Jobs** für **MS Hosted CI/CD** von 0 auf **1**. Klicken Sie dann auf die Schaltfläche **SPEICHERN** am unteren Rand.
1. Gehen Sie in **Organisationseinstellungen** zum Abschnitt **Pipelines** und klicken Sie auf **Einstellungen**.
1. Schalten Sie den Schalter auf **Aus** für **Erstellung von klassischen Build-Pipelines deaktivieren** und **Erstellung von klassischen Release-Pipelines deaktivieren**.
    > Hinweis: Der Schalter **Erstellung klassischer Release-Pipelines deaktivieren**, der auf **Ein** gesetzt ist, blendet die Optionen für die Erstellung klassischer Release-Pipelines wie das Menü **Release** im Abschnitt **Pipeline** von DevOps-Projekten aus.
1. Gehen Sie in **Organisationseinstellungen** zum Abschnitt **Sicherheit** und klicken Sie auf **Richtlinien**.
1. Schalten Sie den Schalter auf **Ein** für **Zugriff von Drittanbieteranwendungen über OAuth**
    > Hinweis: Die OAuth-Einstellung ermöglicht es Tools wie dem DemoDevOpsGenerator, Erweiterungen zu registrieren. Andernfalls kann es vorkommen, dass mehrere Labore mangels der erforderlichen Erweiterungen scheitern.
1. Schalten Sie den Schalter für **Öffentliche Projekte zulassen** auf **Ein**
    > Hinweis: Für die in einigen Labors verwendeten Erweiterungen ist möglicherweise ein öffentliches Projekt erforderlich, damit die kostenlose Version verwendet werden kann.
1. **Warten Sie mindestens 3 Stunden, bevor Sie die CI/CD-Funktionen nutzen**, damit die neuen Einstellungen im Backend berücksichtigt werden. Andernfalls wird weiterhin die Meldung *„Es wurde keine gehostete Parallelität erworben oder gewährt“* angezeigt.

## Anweisungen zum Erstellen des Azure DevOps-Beispielprojekts (Sie müssen dies nur einmal tun)

### Übung 0: Konfigurieren der Voraussetzungen für das Lab

> **Hinweis**: Stellen Sie sicher, dass Sie die Schritte zur Erstellung Ihrer Azure DevOps Organisation abgeschlossen haben, bevor Sie mit diesen Schritten fortfahren.

In dieser Übung richten Sie die Voraussetzungen für das Lab ein, das aus einem neuen Azure DevOps-Projekt mit einem Repository basierend auf dem [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb) besteht.

#### Aufgabe 1:  Erstellen und konfigurieren Sie das Teamprojekt

In dieser Aufgabe erstellen Sie ein **eShopOnWeb** Azure DevOps-Projekt, das von mehreren Labs verwendet werden soll.

1. Öffnen Sie auf Ihrem Lab-Computer in einem Browserfenster Ihre Azure DevOps-Organisation. Klicken Sie auf **Neues Projekt**. Geben Sie Ihrem Projekt die folgenden Einstellungen:
    - Name: **eShopOnWeb**
    - Sichtbarkeit: **Privat**
    - Erweitert: Versionskontrolle: **Git**
    - Erweitert: Work Item Process: **Scrum**

2. Klicken Sie auf **Erstellen**.

    ![Erstellen eines Projekts](images/create-project.png)

#### Aufgabe 2: eShopOnWeb Git-Repository importieren

Bei dieser Aufgabe importieren Sie das eShopOnWeb Git-Repository, das von mehreren Labs verwendet wird.

1. Öffnen Sie auf Ihrem Lab-Computer in einem Browserfenster Ihre Azure DevOps-Organisation und das zuvor erstellte **eShopOnWeb**-Projekt. Klicken Sie auf **Repos>Dateien**, **Repository importieren**. Klicken Sie auf **Importieren**. Fügen Sie im Fenster **Git Repository importieren** die folgende URL https://github.com/MicrosoftLearning/eShopOnWeb.git ein, und klicken Sie auf **Importieren**:

    ![Importieren eines Repositorys](images/import-repo.png)

2. Das Repository ist wie folgt organisiert:
    - Der Ordner **.ado** enthält Azure DevOps-YAML-Pipelines.
    - Der Ordner **.devcontainer** enthält ein Containersetup für die Entwicklung mithilfe von Containern (entweder lokal in VS Code oder über GitHub Codespaces).
    - Der Ordner **.azure** enthält eine Bicep- und ARM-Infrastruktur als Codevorlagen, die in einigen Labs verwendet werden.
    - Der Ordner **.github** enthält YAML GitHub-Workflow-Definitionen.
    - Der Ordner **src** enthält die .NET 7-Website, die für die Labszenarien verwendet wird.

Sie haben nun die notwendigen Voraussetzungen erfüllt, um mit den verschiedenen Einzelübungen für diesen AZ-400-Kurs fortzufahren.
