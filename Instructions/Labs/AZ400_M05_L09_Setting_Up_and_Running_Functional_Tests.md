---
lab:
  title: Einrichten und Ausführen von Funktionstests
  module: 'Module 05: Implement a secure continuous deployment using Azure Pipelines'
---

# Einrichten und Ausführen von Funktionstests

## Lab-Handbuch für Kursteilnehmer

## Labanforderungen

- Für dieses Lab ist **Microsoft Edge** oder ein von [Azure DevOps unterstützter Browser](https://docs.microsoft.com/azure/devops/server/compatibility?view=azure-devops) erforderlich.

- **Einrichten einer Azure DevOps-Organisation**: Wenn Sie nicht bereits eine Azure DevOps-Organisation haben, die Sie für dieses Lab verwenden können, müssen Sie diese erstellen, indem Sie die unter [Erstellen einer Organisation oder Projektsammlung](https://docs.microsoft.com/azure/devops/organizations/accounts/create-organization) beschriebenen Anweisungen befolgen.

- Identifizieren Sie ein vorhandenes Azure-Abonnement, oder erstellen Sie ein neues Abonnement.

- Stellen Sie sicher, dass Sie über ein Microsoft-Konto oder ein Azure AD-Konto mit der Rolle „Besitzer“ im Azure-Abonnement und der Rolle „Globaler Administrator“ im Azure AD-Mandanten verfügen, der dem Azure-Abonnement zugeordnet ist. Ausführliche Informationen finden Sie in den Artikeln zum [Auflisten von Azure-Rollenzuweisungen mithilfe des Azure-Portals](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-list-portal) und [Anzeigen und Zuweisen von Administratorrollen in Azure Active Directory](https://docs.microsoft.com/azure/active-directory/roles/manage-roles-portal).

## Übersicht über das Labor

[Selenium](http://www.seleniumhq.org/) ist ein portables Open-Source-Framework, das Softwaretests für Webanwendungen ermöglicht. Es kann auf fast allen Betriebssystemen eingesetzt werden. Es unterstützt alle modernen Browser und mehrere Sprachen, einschließlich .NET (C#) und Java.

In diesem Lab erfahren Sie, wie Sie Selenium-Testfälle auf einer C#-Webanwendung als Teil der Azure DevOps Release-Pipeline ausführen.

## Ziele

In diesem Lab lernen Sie Folgendes:

- Konfigurieren Sie einen selbst gehosteten Azure DevOps-Agent.
- Übung 1: Konfigurieren der Releasepipeline.
- Auslösen von Build und Release.
- Führen Sie Tests in Chrome und Firefox aus.

## Geschätzte Zeit: 60 Minuten

## Anweisungen

### Übung 0: Konfigurieren der Voraussetzungen für das Lab

In dieser Übung werden Sie die Voraussetzungen für das Labor einrichten. Dazu gehören das vorkonfigurierte Parts Unlimited Team-Projekt, das auf einer Azure DevOps Demo Generator-Vorlage und Azure-Ressourcen basiert.

#### Aufgabe 1: Konfigurieren Sie das Teamprojekt

In dieser Aufgabe werden Sie den Azure DevOps Demo Generator verwenden, um ein neues Projekt auf der Grundlage der **Selenium**-Vorlage zu erstellen.

1. Starten Sie auf Ihrem Laborcomputer einen Webbrowser und navigieren Sie zu [Azure DevOps Demo Generator](https://azuredevopsdemogenerator.azurewebsites.net). Diese Hilfswebsite automatisiert den Prozess der Erstellung eines neuen Azure DevOps-Projekts in Ihrem Konto, das mit den für diese Übung erforderlichen Inhalten (Arbeitselemente, Repos usw.) vorbelegt ist.

    > **Hinweis**: Weitere Informationen zu dieser Website finden Sie unter [Was ist der Azure DevOps Services Demo Generator?](https://docs.microsoft.com/azure/devops/demo-gen).

2. Klicken Sie auf **Anmelden** und melden Sie sich mit dem Microsoft-Konto an, das mit Ihrem Azure DevOps-Abonnement verbunden ist.
3. Klicken Sie bei Bedarf auf der Seite **Azure DevOps Demo Generator** auf **Annehmen**, um die Berechtigungsanfragen für den Zugriff auf Ihr Azure DevOps-Abonnement zu akzeptieren.
4. Auf der Seite **Neues Projekt erstellen** geben Sie in das Textfeld **Neuer Projektname** den Text **Einrichten und Ausführen von Funktionstests** ein, wählen Sie in der Dropdown-Liste **Organisation auswählen** Ihre Azure DevOps-Organisation aus und klicken Sie dann auf **Vorlage auswählen**.
5. Klicken Sie in der Liste der Vorlagen in der Symbolleiste auf **DevOps Labs**, wählen Sie die Vorlage **Selenium** aus und klicken Sie auf **Vorlage auswählen**.
6. Zurück auf der Seite **Neues Projekt erstellen**, klicken Sie auf **Projekt erstellen**

    > **Hinweis**: Warten Sie, bis der Vorgang abgeschlossen ist. Dieser Vorgang dauert etwa zwei Minuten. Sollte der Vorgang fehlschlagen, navigieren Sie zu Ihrer DevOps-Organisation, löschen Sie das Projekt und versuchen Sie es erneut.

7. Klicken Sie auf der Seite **Neues Projekt erstellen** auf **Navigieren zu Projekt**.

#### Schritt 2: Erstellen von Azure-Ressourcen

In dieser Aufgabe werden Sie eine Azure-VM mit Windows Server 2016 zusammen mit SQL Express 2017, Chrome und Firefox bereitstellen.

1. Klicken Sie hier auf den Link **[Deploy to Azure](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2Falmvm%2Fmaster%2Flabs%2Fvstsextend%2Fselenium%2Farmtemplate%2Fazuredeploy.json)**. Dadurch werden Sie automatisch zur Seite **Custom deployment**im Azure-Portal weitergeleitet.
2. Wenn Sie dazu aufgefordert werden, melden Sie sich mit dem Benutzerkonto an, das die Eigentümer-Rolle in dem Azure-Abonnement hat, das Sie in dieser Übung verwenden werden, und das die Rolle des globalen Administrierenden in dem mit diesem Abonnement verbundenen Azure AD-Mandanten hat.
3. Wählen Sie auf der Seite **Benutzerdefinierte Bereitstellung**die Option **Vorlage bearbeiten**.
4. Suchen Sie auf der Seite **Vorlage bearbeiten** die Zeile `"https://raw.githubusercontent.com/microsoft/azuredevopslabs/master/labs/vstsextend/selenium/armtemplate/chrome_firefox_VSTSagent_IIS.ps1"`, ersetzen Sie sie durch `"https://raw.githubusercontent.com/MicrosoftLearning/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/master/Allfiles/Labs/11b/chrome_firefox_VSTSagent_IIS.ps1"`, und klicken Sie auf **Speichern**.
5. Geben Sie auf dem Blatt **Benutzerdefinierte Bereitstellung** die folgenden Einstellungen an:

    | Einstellung | Wert |
    | --- | --- |
    | Subscription | Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden. |
    | Resource group | Der Name einer neuen Ressourcengruppe: **az400m11l02-RG** |
    | Region | Der Name der Azure-Region, in der Sie in diesem Lab die Azure-Ressourcen einsetzen möchten |
    | Name der virtuellen Maschine | **az40011bvm** |

6. Klicken Sie auf **Überprüfen + erstellen** und dann auf **Erstellen**.

    > **Hinweis**: Warten Sie, bis der Vorgang abgeschlossen ist. Dieser Vorgang dauert etwa 15 Minuten.

### Übung 1: Implementieren von Selenium-Tests mithilfe eines selbst gehosteten Azure DevOps-Agents

In dieser Übung werden Sie Selenium-Tests mithilfe eines selbst gehosteten Azure DevOps-Agents implementieren.

#### Aufgabe 1: Konfigurieren eines selbst gehosteten Azure DevOps-Agent

In dieser Aufgabe konfigurieren Sie einen selbst gehosteten Agent unter Verwendung der VM, die Sie in der vorherigen Übung bereitgestellt haben. Selenium erfordert, dass der Agent im interaktiven Modus ausgeführt wird, um die Benutzeroberflächentests durchzuführen.

1. Suchen Sie in dem Webbrowser-Fenster, das das Azure-Portal anzeigt, nach **Virtuelle Computer** und wählen Sie auf der Seite **Virtuelle Computer** die Option **az40011bvm** aus.
2. Wählen Sie auf dem Blatt **az40011bvm** **Verbinden**, wählen Sie im Dropdown-Menü **RDP**, wählen Sie auf der Registerkarte **RDP** des Blattes **az40011bvm \| Verbinden** **RDP-Datei herunterladen** und öffnen Sie die heruntergeladene Datei.
3. Wenn Sie dazu aufgefordert werden, melden Sie sich mit den folgenden Anmeldeinformationen an:

    | Einstellung | Wert |
    | --- | --- |
    | Benutzername | **vmadmin** |
    | Kennwort | **P2ssw0rd@123** |

4. Öffnen Sie innerhalb der Remote Desktop-Sitzung zu **az40011bvm** ein Chrome-Webbrowser-Fenster, navigieren Sie zu **<https://dev.azure.com>** und melden Sie sich bei Ihrer Azure DevOps-Organisation an.
5. Klicken Sie in der unteren linken Ecke des Portals **Azure DevOps** auf **Organisationseinstellungen**.
6. Klicken Sie im vertikalen Menü auf der linken Seite im Abschnitt **Pipelines** auf **Agent-Pools**.
7. Klicken Sie im Fensterbereich **Agent-Pools** auf **Standard**.
8. Klicken Sie im Fensterbereich **Standard** auf **Neuer Agent**.
9. Vergewissern Sie sich im Feld **Abrufen des Agent**, dass die Registerkarte **Windows** und der Abschnitt **x64** ausgewählt sind, und klicken Sie dann auf **Herunterladen**.
10. Starten Sie den Datei-Explorer, erstellen Sie ein Verzeichnis **C:\\AzAgent** und entpacken Sie den Inhalt der heruntergeladenen Agent-Zip-Datei, die sich im Ordner **Downloads** befindet, in dieses Verzeichnis.
11. Klicken Sie innerhalb der Remotedesktopsitzung zu **az40011bvm** mit der rechten Maustaste auf das Menü **Start** und klicken Sie auf **Befehlszeile (Admin)**.
12. Führen Sie innerhalb des Fensters **Administrator: Eingabeaufforderung** den folgenden Befehl aus, um die Installation der Agent-Binärdateien zu starten:

    ```cmd
    cd C:\AzAgent
    Config.cmd
    ```

13. Geben sie im Fenster **Administrator: Eingabeaufforderung** bei der Aufforderung **Server-URL eingeben** **https://dev.azure.com/\<your-DevOps-organization-name\>** ein, wobei **\<your-DevOps-organization-name\>** für den Namen Ihrer Azure DevOps-Organisation steht, und drücken Sie die Taste **Eingabe**.
14. Drücken sie im Fenster **Administrator: Eingabeaufforderung** die **Eingabetaste**, wenn Sie aufgefordert werden, **den Authentifizierungstyp einzugeben (drücken Sie die Eingabetaste für PAT)**.
15. Wechseln Sie im Fenster **Administrator: Eingabeaufforderung**, wenn Sie zu **Persönliches Zugriffstoken eingeben** aufgefordert werden, zum Azure DevOps-Portal, schließen Sie den Bereich **Agent abrufen**, klicken Sie in der oberen rechten Ecke der Azure DevOps-Seite auf das Symbol **Benutzereinstellungen**, klicken Sie im Dropdown-Menü auf **Persönliche Zugriffstoken**, auf den Fensterbereich **Persönliche Zugriffstoken** und klicken Sie auf **+ Neues Token**.
16. Geben Sie im Fensterbereich **Erstellen eines neuen persönlichen Zugriffstokens** die folgenden Einstellungen an und klicken Sie auf **Erstellen** (lassen Sie alle anderen Werte auf den Standardwerten):

    | Einstellung | Wert |
    | --- | --- |
    | Name | **Übung: Einrichtung und Durchführung eines Labs für Funktionstests** |
    | Bereiche | **Benutzerdefiniert** |
    | Bereiche | Klicken Sie auf **Alle Bereiche anzeigen** (am unteren Rand des Fensters) |
    | Bereiche | **Agent-Pools** - **Lesen u. Verwalten** |

17. Kopieren Sie im Bereich **Erfolg** den Wert des persönlichen Zugriffstokens in die Zwischenablage.

    > **Anmerkung**: Stellen Sie sicher, dass Sie das Token kopieren. Sie können ihn nicht mehr abrufen, nachdem Sie diesen Bereich geschlossen haben.

18. Klicken Sie im Bereich **Erfolg** auf **Schließen**.
19. Wechseln Sie zurück zum Fenster **Administrator: Eingabeaufforderung**, fügen Sie den Inhalt der Zwischenablage ein und drücken Sie die **Eingabetaste**.
20. Drücken Sie im Fenster **Administrator: Eingabeaufforderung** bei der Aufforderung **Geben Sie den Agent-Pool ein (drücken Sie die Eingabetaste für die Standardeinstellung)** die **Eingabetaste**.
21. Drücken Sie im Fenster **Administrator: Eingabeaufforderung** bei der Aufforderung **Geben Sie den Namen des Agent ein (drücken Sie die Eingabetaste für az40011bvm)** die **Eingabetaste**.
22. Drücken Sie im Fenster **Administrator: Eingabeaufforderung** bei der Aufforderung **Geben Sie den Arbeitsordner ein (drücken Sie die Eingabetaste für _work)** die **Eingabetaste**.
23. Drücken Sie im Fenster **Administrator: Eingabeaufforderung** bei der Aufforderung **Agent als Dienst ausführen (J/N) (drücken Sie die Eingabetaste für N)** die **Eingabetaste**.
24. Drücken Sie im Fenster **Administrator: Eingabeaufforderung** bei der Aufforderung **Autologon konfigurieren und Agent beim Start ausführen (J/N) (drücken Sie die Eingabetaste für N)** die **Eingabetaste**.
25. Sobald der Agent registriert ist, geben Sie im Fenster **Administrator: Eingabeaufforderung** **run.cmd** ein und drücken die **Eingabetaste**, um den Agent zu starten.

    > **Hinweis**: Sie müssen auch das Dac Framework installieren, das von der Anwendung verwendet wird, die Sie später im Lab einsetzen werden.

26. Starten Sie innerhalb der Remotedesktop-Sitzung zu **az40011bvm** eine weitere Instanz des Webbrowsers, navigieren Sie zur [ Download-Seite für Microsoft SQL Server Data-Tier Application Framework (18.2)](https://www.microsoft.com/download/details.aspx?id=58207&WT.mc_id=rss_alldownloads_extensions) und klicken Sie auf **Herunterladen**.
27. Markieren sie auf der Seite **Wählen Sie den gewünschten Download** das Kontrollkästchen **EN\x64\DacFramework.msi** und klicken Sie auf **Weiter**. Dadurch wird der automatische Download der Datei **DacFramework.msi** ausgelöst.
28. Sobald der Download der Datei **DacFramework.msi** abgeschlossen ist, verwenden Sie sie, um die Installation des Microsoft SQL Server Data-Tier Application Framework mit den Standardeinstellungen auszuführen.

#### Aufgabe 2: Konfigurieren einer Freigabe-Pipeline

In dieser Aufgabe werden Sie die Freigabe-Pipeline konfigurieren.

> **Hinweis**: Die Azure-VM hat den Agent konfiguriert, um die Anwendungen bereitzustellen und Selenium-Testfälle auszuführen. Die Freigabedefinition verwendet **[Phasen](https://docs.microsoft.com/vsts/build-release/concepts/process/phases)** für die Bereitstellung auf Zielservern.

1. Klicken Sie innerhalb der Remotedesktop-Sitzung zu **az40011bvm** im Browserfenster, das das Portal **Azure DevOps** anzeigt, auf das Symbol **Azure DevOps** in der oberen linken Ecke.
2. Klicken Sie im Fensterbereich mit den Projekten Ihrer Organisation auf die Kachel, die das Projekt **Einrichten und Ausführen von Funktionstets** darstellt.
3. Wählen Sie im Fensterbereich **Einrichten und Ausführen von Funktionstests** im vertikalen Navigationsbereich **Pipelines**, klicken Sie im Abschnitt **Pipelines** auf **Freigaben** und klicken Sie dann im Fensterbereich **Selenium** auf **Bearbeiten**.
4. Klicken Sie im Fensterbereich **Alle Pipelines > Selenium** auf die Registerkarte **Aufgaben** und im Dropdown-Menü auf **Entwicklung**.
5. Überprüfen Sie innerhalb der Liste der Aufgaben der **Entwicklungs**-Phase die Bereitstellungsphasen **IIS-Bereitstellung**, **SQL-Bereitstellung** und **Selenium-Testausführung**.

   - **IIS-Bereitstellungsphase**: In dieser Phase wird die Anwendung mithilfe der folgenden Aufgaben auf der VM bereitgestellt:

     - **IIS Web App Manage**: Diese Aufgabe wird auf dem Zielrechner ausgeführt, auf dem wir den Agenten registriert haben. Es wird eine *Website* und ein *Anwendungspool* lokal mit dem Namen **PartsUnlimited** erstellt, der unter dem Port **82** läuft, [**http://localhost:82**](http://localhost:82)
     - **IIS Web App Deploy**: Diese Aufgabe stellt die Anwendung auf dem IIS-Server mit **Web Deploy** bereit.

   - **Bereitstellungsphase der Datenbank**: In dieser Phase verwenden wir die Aufgabe [**SQL Server Database Deploy**](https://github.com/Microsoft/vsts-tasks/blob/master/Tasks/SqlDacpacDeploymentOnMachineGroup/README.md), um die Datei [**dacpac**](https://docs.microsoft.com/sql/relational-databases/data-tier-applications/data-tier-applications) auf dem DB-Server bereitzustellen.

   - **Ausführung von Selenium-Tests**: Die Ausführung von **UI-Tests** als Teil des Release-Prozesses ermöglicht es uns, unerwartete Änderungen zu erkennen. Die Einrichtung automatisierter browserbasierter Tests steigert die Qualität Ihrer Anwendung, ohne dass Sie sie manuell durchführen müssen. In dieser Phase führen wir Selenium-Tests für die bereitgestellte Webanwendung aus. Die folgenden Aufgaben beschreiben die Verwendung von Selenium zum Testen der Websites in der Release-Pipeline.

     - **Visual Studio Test Platform Installer**: Die Aufgabe [Visual Studio Test Platform Installer](https://docs.microsoft.com/azure/devops/pipelines/tasks/tool/vstest-platform-tool-installer?view=vsts) bezieht die Microsoft-Testplattform von nuget.org oder einem angegebenen Feed und fügt sie dem Tools-Cache hinzu. Sie erfüllt die **vstest**-Anforderungen, sodass jede nachfolgende Visual Studio Test-Aufgabe in einer Build- oder Release-Pipeline ausgeführt werden kann, ohne dass eine vollständige Visual Studio-Installation auf dem Agentencomputer erforderlich ist.
     - **Ausführen von Selenium UI-Tests**: Diese [Aufgabe](https://github.com/Microsoft/azure-pipelines-tasks/blob/master/Tasks/VsTestV2/README.md) verwendet **vstest.console.exe**, um die Selenium-Testfälle auf den Agentenrechnern auszuführen.

6. Klicken Sie im Bereich **Alle Pipelines > Selenium** auf die Phase **IIS Deployment** und stellen Sie im Bereich **Agent job** sicher, dass der **Default** Agent pool ausgewählt ist.
7. Wiederholen Sie den vorherigen Schritt für die **SQL Deployment** und die **Selenium Testausführung** Phasen. Klicken Sie bei Bedarf auf **Speichern**, um die Änderungen zu speichern.

#### Aufgabe 3: Build und Freigabe auslösen

In dieser Aufgabe werden wir die **Build** auslösen, um Selenium C# Skripte zusammen mit der Webanwendung zu kompilieren. Die resultierenden Binärdateien werden auf den selbst gehosteten Agenten kopiert und die Selenium-Skripte werden als Teil der automatisierten **Freigabe** ausgeführt.

1. Klicken Sie innerhalb der Remote Desktop-Sitzung zu **az40011bvm** im Browserfenster, das das Portal **Azure DevOps** anzeigt, im vertikalen Navigationsbereich im Abschnitt **Pipelines** auf **Pipelines** und dann im Bereich **Pipelines** auf **Selenium**.
2. Klicken Sie im Bereich **Selenium** auf **Pipeline ausführen** und im Bereich **Pipeline ausführen** auf **Ausführen**.

    > **Hinweis**: Dieser Build wird die Testartefakte an Azure DevOps veröffentlichen, die in der Veröffentlichung verwendet werden.

    > **Anmerkung**: Sobald der Build erfolgreich ist, wird die Freigabe ausgelöst.

3. Klicken Sie im Bereich „Pipeline-Läufe“ im Abschnitt **Jobs** auf **Phase 1** und überwachen Sie den Build-Fortschritt bis zu seinem Abschluss.
4. Klicken Sie im Browserfenster, das das **Azure DevOps**-Portal anzeigt, im vertikalen Navigationsbereich im Abschnitt **Pipelines** auf **Releases**. Klicken Sie auf den Eintrag, der das Release darstellt, und klicken Sie im Bereich **Selenium > Release-1** auf **Dev**.
5. Überwachen Sie im Bereich **Selenium > Release-1 > Dev** die entsprechende Bereitstellung.
6. Sobald die **Selenium-Testausführungsphase** beginnt, überwachen Sie die Webbrowser-Tests.
7. Sobald die Freigabe abgeschlossen ist, klicken Sie im Bereich **Selenium > Release-1 > Dev** auf die Registerkarte **Tests**, um die Testergebnisse zu analysieren. Wählen Sie die gewünschten Filter aus dem Dropdown-Menü im Abschnitt **Ergebnis**, um die Tests und ihren Status anzuzeigen.

### Übung 2: Entfernen Sie die Azure-Laborressourcen

In dieser Übung entfernen Sie die in dieser Übung bereitgestellten Azure-Ressourcen, um unerwartete Gebühren zu beseitigen.

>**Hinweis**: Denken Sie daran, alle neu erstellten Azure-Ressourcen zu entfernen, die Sie nicht mehr verwenden. Durch das Entfernen nicht verwendeter Ressourcen wird sichergestellt, dass keine unerwarteten Kosten anfallen.

#### Übung 1: Entfernen der Azure Lab-Ressourcen

In dieser Aufgabe verwenden Sie Azure Cloud Shell, um die in dieser Übung bereitgestellten Azure-Ressourcen zu entfernen, um unnötige Gebühren zu vermeiden.

1. Öffnen Sie im Azure-Portal im **Cloud Shell**-Bereich die **Bash**-Sitzung.
2. Listen Sie alle Ressourcengruppen auf, die während der Labs in diesem Modul erstellt wurden, indem Sie den folgenden Befehl ausführen:

    ```sh
    az group list --query "[?starts_with(name,'az400m11l02-RG')].name" --output tsv
    ```

3. Löschen Sie alle Ressourcengruppen, die Sie während der praktischen Übungen in diesem Modul erstellt haben, indem Sie den folgenden Befehl ausführen:

    ```sh
    az group list --query "[?starts_with(name,'az400m11l02-RG')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**Hinweis**: Der Befehl wird (dem --nowait-Parameter entsprechend) asynchron ausgeführt. Dies bedeutet, dass Sie zwar einen weiteren Azure CLI-Befehl in derselben Bash-Sitzung direkt im Anschluss ausführen können, es jedoch einige Minuten dauert, bis die Ressourcengruppen tatsächlich entfernt werden.

## Überprüfung

In dieser Übung haben Sie gelernt, wie Sie Selenium-Testfälle auf einer C#-Webanwendung als Teil der Azure DevOps Release-Pipeline ausführen können.
