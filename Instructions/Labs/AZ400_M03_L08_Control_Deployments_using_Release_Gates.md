---
lab:
  title: Steuern von Bereitstellungen mithilfe von Releasegates
  module: 'Module 03: Design and implement a release strategy'
---

# Steuern von Bereitstellungen mithilfe von Releasegates

## Labanforderungen

- Für dieses Lab ist **Microsoft Edge** oder ein von [Azure DevOps unterstützter Browser](https://docs.microsoft.com/azure/devops/server/compatibility) erforderlich.

- **Einrichten einer Azure DevOps-Organisation**: Wenn Sie nicht bereits eine Azure DevOps-Organisation haben, die Sie für dieses Lab verwenden können, müssen Sie diese erstellen, indem Sie die unter [Erstellen einer Organisation oder Projektsammlung](https://docs.microsoft.com/azure/devops/organizations/accounts/create-organization) beschriebenen Anweisungen befolgen.

- Identifizieren Sie ein vorhandenes Azure-Abonnement, oder erstellen Sie ein neues Abonnement.

- Stellen Sie sicher, dass Sie über ein Microsoft- oder Microsoft Entra-Konto mit der Rolle „Besitzer“ im Azure-Abonnement und der Rolle „Globaler Administrator“ im Microsoft Entra-Mandanten verfügen, der dem Azure-Abonnement zugeordnet ist. Ausführliche Informationen finden Sie in den Artikeln zum [Auflisten von Azure-Rollenzuweisungen mithilfe des Azure-Portals](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-list-portal) und [Anzeigen und Zuweisen von Administratorrollen in Azure Active Directory](https://docs.microsoft.com/azure/active-directory/roles/manage-roles-portal).

## Übersicht über das Labor

In diesem Lab werden die Konfiguration der Bereitstellungsgates sowie die Steuerung der Ausführung von Azure Pipelines behandelt. Um die Implementierung zu veranschaulichen, konfigurieren Sie eine Releasedefinition mit zwei Umgebungen für eine Azure-Web-App. Sie stellen die DevTest-Umgebung nur dann bereit, wenn es keine blockierenden Fehler für die Anwendung gibt, und markieren die DevTest-Umgebung nur dann als abgeschlossen, wenn es keine aktiven Alarme in Application Insights von Azure Monitor gibt.

Eine Releasepipeline gibt den End-to-End-Releaseprozess für eine Anwendung an, die in verschiedenen Umgebungen bereitgestellt werden soll. Die Bereitstellungen in den einzelnen Umgebungen erfolgen vollständig automatisiert über Aufträge und Tasks. Im Idealfall sollten neue Updates für die Anwendungen nicht gleichzeitig für alle Benutzer verfügbar gemacht werden. Es hat sich bewährt, bei Updates phasenweise vorzugehen: Ein Update wird zunächst nur für eine Teilmenge von Benutzern verfügbar gemacht, deren Nutzung wird überwacht, und dann wird das Update anderen Benutzern basierend auf der Erfahrung der ersten Benutzergruppe zur Verfügung gestellt.

Mit Genehmigungen und Gates haben Sie vollständige Kontrolle über den Beginn und Abschluss der Bereitstellungen in einem Release. Sie können warten, bis Benutzer Bereitstellungen manuell genehmigen oder ablehnen. Mithilfe von Releasegates können Sie Kriterien für die Anwendungsintegrität angeben, die erfüllt werden müssen, bevor das Release in die nächste Umgebung weitergegeben wird. Vor oder nach einer Umgebungsbereitstellung werden alle angegebenen Gates automatisch ausgewertet, bis sie den von Ihnen definierten Timeoutzeitraum erreichen oder überschreiten, wonach ein Fehler auftritt.

Gates können einer Umgebung in der Releasedefinition im Bereich der Bedingungen vor der Bereitstellung oder im Bereich der Bedingungen nach der Bereitstellung hinzugefügt werden. Den Umgebungsbedingungen können mehrere Gates hinzugefügt werden, um sicherzustellen, dass alle Eingaben für das Release erfolgreich sind.

Beispiel:

- Gates vor der Bereitstellung stellen sicher, dass im Arbeitselement- oder Problemverwaltungssysstem keine aktiven Probleme vorhanden sind, bevor ein Build in einer Umgebung bereitgestellt wird.
- Gates nach der Bereitstellung stellen sicher, dass im Überwachungs- oder Incidentverwaltungssystem der App keine Incidents vorhanden sind, bevor sie das Release in die nächste Umgebung weitergeben.

In jedem Konto sind standardmäßig 4 Arten von Gates enthalten.

- Azure Function aufrufen: Hiermit wird die Ausführung einer Azure Function ausgelöst und sichergestellt, dass sie erfolgreich abgeschlossen wird.
- Azure Monitor-Warnungen abfragen: Hiermit werden die konfigurierten Azure Monitor-Warnungsregeln für aktive Warnungen beobachtet.
- REST-API aufrufen: Hiermit wird eine REST-API aufgerufen, der Prozess wird fortgesetzt, wenn eine Erfolgsantwort zurückgesendet wird.
- Arbeitselemente abfragen: Hiermit wird sichergestellt, dass die Anzahl der von einer Abfrage zurückgegebenen übereinstimmenden Arbeitselemente innerhalb eines bestimmten Schwellenwerts liegt.

## Ziele

In diesem Lab lernen Sie Folgendes:

- Konfigurieren Sie Releasepipelines.
- Konfigurieren Sie Releasegates.
- Testen Sie Releasegates.

## Geschätzte Dauer: 75 Minuten

## Anweisungen

### Übung 0: Konfigurieren der Voraussetzungen für das Lab

In dieser Übung richten Sie die Voraussetzungen für das Lab ein.

#### Aufgabe 1: (überspringen, wenn fertig) Erstellen und Konfigurieren des Teamprojekts

In dieser Aufgabe erstellen Sie ein **eShopOnWeb** Azure DevOps-Projekt, das von mehreren Labs verwendet werden soll.

1. Öffnen Sie auf Ihrem Lab-Computer in einem Browserfenster Ihre Azure DevOps-Organisation. Klicken Sie auf **Neues Projekt**. Weisen Sie Ihrem Projekt den Namen **eShopOnWeb** zu, und lassen Sie die anderen Felder auf den Standardwerten. Klicken Sie auf **Erstellen**.

   ![Screenshot des Bereichs „Neues Projekt erstellen“.](images/create-project.png)

#### Aufgabe 2: (überspringen, wenn erledigt) Importieren von eShopOnWeb Git Repository

Bei dieser Aufgabe importieren Sie das eShopOnWeb Git-Repository, das von mehreren Labs verwendet wird.

1. Öffnen Sie auf Ihrem Lab-Computer in einem Browserfenster Ihre Azure DevOps-Organisation und das zuvor erstellte **eShopOnWeb**-Projekt. Klicken Sie auf **Repos > Dateien** , **Importiere ein Repository**. Klicken Sie auf **Importieren**. Fügen Sie im Fenster **Git Repository importieren** die folgende URL <https://github.com/MicrosoftLearning/eShopOnWeb.git> ein, und klicken Sie auf **Importieren**:

   ![Screenshot: Fenster „Repository importieren“](images/import-repo.png)

1. Das Repository ist wie folgt organisiert:
   - Der Ordner **.ado** enthält Azure DevOps-YAML-Pipelines.
   - Der Ordner **.devcontainer** enthält ein Containersetup für die Entwicklung mithilfe von Containern (entweder lokal in VS Code oder über GitHub Codespaces).
   - Der Ordner **infra** enthält eine Bicep&ARM-Infrastruktur als Codevorlagen, die in einigen Labszenarien verwendet werden.
   - Der Ordner **.github** enthält YAML GitHub-Workflow-Definitionen.
   - Der Ordner **src** enthält die .NET 8-Website, die in den Labszenarios verwendet wird.

1. Wechseln Sie zu **Repos > Branches**.
1. Bewegen Sie den Mauszeiger auf den **Main**-Branch und klicken Sie dann rechts neben der Spalte auf die Auslassungspunkte.
1. Klicken Sie auf **Als Mainbranch festlegen**.

#### Aufgabe 3: Konfigurieren einer CI-Pipeline-as-Code mit YAML in Azure DevOps

In dieser Aufgabe fügen Sie dem vorhandenen Projekt eine YAML-Builddefinition hinzu.

1. Navigieren Sie zurück zum Bereich **Pipelines** im **Pipelinehub**.
1. Klicken Sie im Fenster **Erste Pipeline erstellen** auf **Pipeline erstellen**.

   > **Hinweis**: Wir verwenden den Assistenten, um eine neue YAML-Pipelinedefinition basierend auf unserem Projekt zu erstellen.

1. Klicken Sie im Bereich **Wo befindet sich Ihr Code?** auf die Option **Azure Repos Git (YAML)**.
1. Klicken Sie im Bereich **Ein Repository auswählen** auf **eShopOnWeb**.
1. Scrollen Sie auf dem Bildschirm **Pipeline konfigurieren** nach unten und wählen Sie die Option **Vorhandene Azure Pipelines-YAML-Datei** aus.
1. Geben Sie im Blatt **Auswählen einer vorhandenen YAML-Datei** die folgenden Parameter an:
   - Branch: **main**
   - Pfad: **.ado/eshoponweb-ci.yml**
1. Klicken Sie auf **Weiter**, um die Einstellungen zu speichern.
1. Klicken Sie auf dem Bildschirm **Pipeline-YML überprüfen** auf **Ausführen**, um den Buildpipelineprozess zu starten.
1. Warten Sie auf die erfolgreiche Fertigstellung der Buildpipeline. Ignorieren Sie alle Warnungen bezüglich des Quellcodes selbst, da sie für diese Übung nicht relevant sind.

   > **Hinweis**: Jede Aufgabe aus der YAML-Datei steht zur Überprüfung zur Verfügung, einschließlich aller Warnungen und Fehler.

1. Ihre Pipeline bekommt einen Namen basierend auf dem Projektnamen. Wir **benennen sie um**, damit wir die Pipeline besser identifizieren können. Wechseln Sie zu **Pipelines > Pipelines**, und klicken Sie auf die kürzlich erstellte Pipeline. Klicken Sie auf die Auslassungspunkte und die Option **Umbenennen/Verschieben**. Benennen Sie sie**`eshoponweb-ci`**, und klicken Sie auf **Speichern**.

### Übung 1: Die erforderlichen Azure-Ressourcen für die Releasepipeline erstellen

#### Aufgabe 1: Zwei Azure-Webanwendungen erstellen

In dieser Aufgabe erstellen Sie zwei Azure-Webanwendungen, die die Umgebungen **DevTest** und **Produktion** repräsentieren, in denen Sie die Anwendung über Azure Pipelines bereitstellen.

1. Starten Sie auf Ihrem Labcomputer einen Webbrowser, navigieren Sie zum [**Azure-Portal**](https://portal.azure.com), und melden Sie sich an. Verwenden Sie hierzu die Anmeldeinformationen eines Benutzerkontos, das in dem Abonnement, das Sie in diesem Lab verwenden, und das in dem in dem Microsoft Entra-Mandanten, der dem Abonnement zugeordnet ist, über die Rolle „Globaler Administrator“ verfügt.
1. Klicken Sie im Azure-Portal auf das Symbol **Cloud Shell**, das sich direkt rechts neben dem Textfeld für die Suche im oberen Bereich der Seite befindet.
1. Wählen Sie bei Aufforderung zur Auswahl von **Bash** oder **PowerShell** die Option **Bash** aus.

   > **Hinweis**: Wenn Sie **Cloud Shell** zum ersten Mal starten und die Meldung **Für Sie wurde kein Speicher bereitgestellt** angezeigt wird, wählen Sie das in diesem Lab verwendete Abonnement aus, und klicken Sie dann auf **Speicher erstellen**.

1. Führen Sie an der **Bash**-Eingabeaufforderung im Bereich **Cloud Shell** den folgenden Befehl aus, um eine Ressourcengruppe zu erstellen (ersetzen Sie den `<region>`-Variablenplatzhalter durch den Namen der Azure-Region, in der die beiden Azure-Webanwendungen gehostet werden, z. B. "westeurope" oder "centralus", oder eine andere verfügbare Region Ihrer Wahl):

   > **Hinweis**: Mögliche Speicherorte können durch Ausführen des folgenden Befehls gefunden werden, verwenden Sie den **Namen** für `<region>` : `az account list-locations -o table`

   ```bash
   REGION='centralus'
   RESOURCEGROUPNAME='az400m03l08-RG'
   az group create -n $RESOURCEGROUPNAME -l $REGION
   ```

1. So erstellen Sie einen App-Serviceplan

   ```bash
   SERVICEPLANNAME='az400m03l08-sp1'
   az appservice plan create -g $RESOURCEGROUPNAME -n $SERVICEPLANNAME --sku S1
   ```

1. Erstellen Sie zwei Webanwendungen mit eindeutigen App-Namen.

   ```bash
   SUFFIX=$RANDOM$RANDOM
   az webapp create -g $RESOURCEGROUPNAME -p $SERVICEPLANNAME -n RGATES$SUFFIX-DevTest
   az webapp create -g $RESOURCEGROUPNAME -p $SERVICEPLANNAME -n RGATES$SUFFIX-Prod
   ```

   > **Hinweis**: Notieren Sie den Namen der DevTest-Webanwendung. Sie benötigen diese später in diesem Lab.

1. Warten Sie, bis der Prozess der Bereitstellung von Webanwendungs-Service-Ressourcen abgeschlossen ist, und schließen Sie das Fenster **Cloud Shell**.

#### Aufgabe 2: Eine Application Insights-Ressource erstellen

1. Verwenden Sie im Azure-Portal das Textfeld **Ressourcen, Dienste und Dokumente suchen** oben auf der Seite, um nach **Application Insights** zu suchen, und wählen Sie in der Ergebnisliste **Application Insights** aus.
1. Klicken Sie im Blatt **Application Insights** auf **+ Erstellen**.
1. Geben Sie auf dem Blatt **Application Insights** auf der Registerkarte **Grundlagen** die folgenden Einstellungen an (belassen Sie die anderen mit ihren Standardwerten):

   | Einstellung        | Wert                                                                                 |
   | -------------- | ------------------------------------------------------------------------------------- |
   | Resource group | **az400m03l08-RG**                                                                    |
   | Name           | der Name der DevTest-Web-App, die Sie in der vorherigen Aufgabe aufgezeichnet haben                     |
   | Region         | die gleiche Azure-Region, in der Sie die Web-Apps in der vorherigen Aufgabe bereitgestellt haben |

1. Klicken Sie auf **Überprüfen + erstellen** und dann auf **Erstellen**.
1. Warten Sie auf den Abschluss des Bereitstellungsvorgangs.
1. Navigieren Sie im Azure-Portal zu der Ressourcengruppe **az400m03l08-RG**, die Sie in der vorherigen Aufgabe erstellt haben.
1. Klicken Sie in der Liste der Ressourcen auf die Web-App **DevTest**.
1. Klicken Sie auf der Web-App-Seite **DevTest** im vertikalen Menü links im Abschnitt **Überwachen** auf **Application Insights**.
1. Klicken Sie im Blatt **Application Insights** auf **Application Insights aktivieren**.
1. Klicken Sie im Abschnitt **Ressource ändern** auf die Option **Vorhandene Ressource auswählen**, wählen Sie in der Liste der vorhandenen Ressourcen die neu erstellte Application Insight-Ressource aus, klicken Sie auf **Anwenden** und, wenn Sie zur Bestätigung aufgefordert werden, auf **Ja**.
1. Warten Sie, bis die Änderung wirksam wird.

   > **Hinweis**: Sie werden hier Monitor-Warnungen erstellen, die Sie im späteren Teil dieses Übung verwenden werden.

1. Wählen Sie im Menü **Einstellungen** / **Anwendungsübersichten** in der Web-App die Option **Anwendungsübersichten-Daten anzeigen**. Das Blatt Application Insights wird im Azure-Portal geöffnet.
1. Klicken Sie auf dem Blatt "Application Insights Resource" im Abschnitt **Überwachung** auf **Warnungen** und dann auf **Erstellen einer >Alarmregel**.
1. Klicken Sie auf dem Blatt **Warnungsregel erstellen** im Abschnitt **Bedingung** auf **Alle Signale anzeigen** und geben Sie **Abfragen** ein. Wählen Sie in der Ergebnisliste **Fehlgeschlagene Abfragen**.
1. Lassen Sie auf der Seite **Erstellen Sie eine Alarmregel** im Abschnitt **Bedingung** den **Schwellenwert** auf **Statisch** gesetzt und bestätigen Sie die anderen Standardeinstellungen wie folgt:

   - Aggregationstyp: Count
   - Operator: Größer als
   - Einheit: Anzahl

1. Geben Sie im Textfeld **Schwellenwert** **0** ein und klicken Sie auf **Weiter:Aktionen**. Nehmen Sie keine Änderungen auf der Einstellungsseite **Aktionen** vor und definieren Sie die folgenden Parameter im Abschnitt **Details**:

   | Einstellung                                        | Wert                            |
   | ---------------------------------------------- | -------------------------------- |
   | Severity                                       | **2- Warnung**                   |
   | Name der Warnungsregel                                | **RGATESDevTest_FailedRequests** |
   | Erweiterte Optionen: Automatisches Auflösen von Warnungen | **deaktiviert**                      |

   > **Hinweis**: Metrikwarnungsregeln können bis zu 10 Minuten dauern, bis sie aktiviert werden.

   > **Hinweis**: Sie können mehrere Warnungsregeln für unterschiedliche Metriken erstellen, z. B. Verfügbarkeit < 99 Prozent, Serverantwortzeit > 5 Sekunden oder Serverausnahmen > 0

1. Bestätigen Sie die Erstellung der Warnungsregel, indem Sie auf **Überprüfen+Erstellen** klicken und erneut bestätigen, indem Sie auf **Erstellen** klicken. Warten Sie, bis die Warnungsregel erfolgreich erstellt wurde.

### Übung 2: Konfigurieren der Releasepipeline

In dieser Übung konfigurieren Sie eine Releasepipeline.

#### Aufgabe 1: Einrichten von Releaseaufgaben

In dieser Aufgabe richten Sie die Releaseaufgaben als Teil der Releasepipeline ein.

1. Wählen Sie im Azure DevOps-Portal im **eShopOnWeb**-Projekt im vertikalen Navigationsbereich **Pipelines** aus, und klicken Sie dann im Abschnitt **Pipelines** auf **Releases**.
1. Klicken Sie auf **Neue Pipeline**.
1. **Wählen** Sie im Fenster **Vorlage auswählen** den Eintrag **Azure-Appdienstbereitstellung** (Bereitstellen Ihrer Anwendung in Azure App Service. Wählen Sie Web App unter Windows, Linux, Container, Function Apps oder WebJobs) aus der Liste der **Empfohlenen** Vorlagen.
1. Klicken Sie auf **Anwenden**.
1. Aktualisieren Sie im Fenster **Phase** den Standard-Phasenname „Phase 1“ auf **DevTest**. Schließen Sie das Popupfenster mithilfe der Schaltfläche **X**. Sie befinden sich jetzt im grafischen Editor der Releasepipeline, und die DevTest-Phase wird angezeigt.
1. Benennen Sie oben auf der Seite die aktuelle Pipeline von **Neue Releasepipeline** in **eshoponweb-cd** um.
1. Bewegen Sie den Mauszeiger über die DevTest-Phase, und klicken Sie auf die Schaltfläche **Klonen**, um die DevTest-Phase in eine zusätzliche Phase zu kopieren. Nennen Sie diese Phase **Produktion**.

   > **Hinweis**: Die Pipeline umfasst jetzt zwei Phasen mit den Namen **DevTest** und **Produktion**.

1. Markieren Sie auf der Registerkarte **Pipeline** das Rechteck **Artefakt hinzufügen**, und wählen Sie **eshoponweb-ci** im Feld **Quelle (Buildpipeline)**. Klicken Sie auf **Hinzufügen**, um die Auswahl des Artefakts zu bestätigen.
1. Achten Sie im Rechteck **Artefakte** auf den **Continuous Deployment-Trigger** (Blitz). Klicken Sie darauf, um die Einstellungen für den **Continuous Deployment-Trigger** zu öffnen. Klicken Sie auf **Deaktiviert**, um die Funktion umzuschalten und zu aktivieren. Belassen Sie alle anderen Einstellungen bei den Standardeinstellungen, und schließen Sie das Fenster **Continuous Deployment-Trigger**, indem Sie oben rechts auf die Markierung **x** klicken.
1. Klicken Sie in der Phase **DevTest-Umgebungen** auf die Bezeichnung **1 Auftrag, 1 Aufgabe**, und überprüfen Sie die Aufgaben in dieser Phase.

   > **Hinweis**: Die DevTest-Umgebung hat 1 Aufgabe, die das Artefaktpaket in Azure Web App veröffentlicht.

1. Stellen Sie im Bereich **Alle Pipelines > eshoponweb-cd** sicher, dass die Phase **DevTest** ausgewählt ist. Wählen Sie in der Dropdownliste **Azure-Abonnement** Ihr Azure-Abonnement aus, und klicken Sie auf **Autorisieren**. Wenn Sie dazu aufgefordert werden, authentifizieren Sie sich mit dem Benutzerkonto, das im Azure-Abonnement mit der Rolle „Besitzer“ eingerichtet wurde.
1. Stellen Sie sicher, dass der App-Typ auf „Web-App unter Windows“ eingestellt ist. Wählen Sie anschließend in der Dropdownliste **App Service-Name** den Namen der Web App **DevTest**.
1. Wählen Sie den Task **Azure App Service bereitstellen**. Aktualisieren Sie im Feld **Paket oder Ordner** den Standardwert von „$(System.DefaultWorkingDirectory)/\*\*/\*.zip“ auf „$(System.DefaultWorkingDirectory)/\*\*/Web.zip“.

   > **Hinweis**: Beachten Sie das Ausrufezeichen neben der Registerkarte „Aufgaben“. Dies ist beabsichtigt, da die Einstellungen für die Produktionsphase konfiguriert werden müssen.

1. Öffnen Sie den Bereich **Anwendungs- und Konfigurationseinstellungen**, und geben Sie `-UseOnlyInMemoryDatabase true -ASPNETCORE_ENVIRONMENT Development` in das Feld **App-Einstellungen** ein.

1. Navigieren Sie im Bereich **Alle Pipelines > eshoponweb-cd** zur Registerkarte **Pipeline** und klicken Sie diesmal innerhalb der Phase **Produktion** auf die Bezeichnung **1 Auftrag, 1 Aufgabe**. Vervollständigen Sie die Pipeline-Einstellungen, genau wie in der DevTest-Phase zuvor. Wählen Sie auf der Registerkarte „Aufgaben / Produktionsbereitstellungsprozess“ in der Dropdown-Liste **Azure-Abonnement** das Azure-Abonnement aus, das Sie für die **DevTest-Umgebung** verwendet haben und das unter **Verfügbare Azure-Dienstverbindungen** angezeigt wird, da die Dienstverbindung bereits zuvor bei der Autorisierung der Abonnementnutzung erstellt wurde.
1. Wählen Sie in der Dropdownliste **App Service-Name** den Namen der **Prod** Web App aus.
1. Wählen Sie den Task **Azure App Service bereitstellen**. Aktualisieren Sie im Feld **Paket oder Ordner** den Standardwert von „$(System.DefaultWorkingDirectory)/\*\*/\*.zip“ auf „$(System.DefaultWorkingDirectory)/\*\*/Web.zip“.
1. Öffnen Sie den Bereich **Anwendungs- und Konfigurationseinstellungen**, und geben Sie `-UseOnlyInMemoryDatabase true -ASPNETCORE_ENVIRONMENT Development` in das Feld **App-Einstellungen** ein.
1. Klicken Sie im Bereich **Alle Pipelines > eshoponweb-cd** auf **Speichern** und im Dialogfeld **Speichern** auf **OK**.

   Sie haben jetzt die Release-Pipeline erfolgreich konfiguriert.

1. Klicken Sie in dem Browserfenster, in dem das Projekt **eShopOnWeb** angezeigt wird, in der vertikalen Navigationsleiste im Abschnitt **Pipelines** auf **Pipelines**.
1. Klicken Sie im Bereich **Pipelines** auf den Eintrag, der die **eshoponweb-ci** Build-Pipeline darstellt und klicken Sie dann auf **Pipeline ausführen**.
1. Bestätigen Sie im Bereich **Pipeline ausführen** die Standardeinstellungen und klicken Sie auf **Ausführen**, um die Pipeline zu starten. **Warten Sie, bis die Build-Pipeline abgeschlossen ist**.

   > **Hinweis**: Nach erfolgreichem Build wird das Release automatisch angestoßen und die Anwendung wird in beiden Umgebungen bereitgestellt. Bestätigen Sie die Release-Aktionen, nachdem die Build-Pipeline erfolgreich abgeschlossen wurde.

1. Klicken Sie in der vertikalen Navigationsleiste im Abschnitt **Pipelines** auf **Releases** und im Abschnitt **eshoponweb-cd** auf den Eintrag, der der neuesten Version entspricht.
1. Überwachen Sie auf dem **eshoponweb-cd > Release-1**-Blatt den Fortschritt des Releases und vergewissern Sie sich, dass die Bereitstellung für beide Web-Apps erfolgreich abgeschlossen wurde.
1. Wechseln Sie zur Azure-Portal-Schnittstelle, navigieren Sie zur Ressourcengruppe **az400m03l08-RG**, klicken Sie in der Ressourcenliste auf die Web-App **DevTest**, klicken Sie auf dem Web-App-Blatt auf **Durchsuchen** und überprüfen Sie, ob die Webseite (E-Commerce-Website) erfolgreich in einer neuen Webbrowserregisterkarte geladen wird.
1. Wechseln Sie zurück zur Azure-Portal-Schnittstelle und navigieren Sie diesmal zur Ressourcengruppe **az400m03l08-RG**. Klicken Sie in der Ressourcenliste auf die Web-App **Produktion**, klicken Sie im Web-App-Blatt auf **Durchsuchen** und überprüfen Sie, ob die Webseite erfolgreich in einer neuen Webbrowserregisterkarte geladen wird.
1. Schließen Sie die Webbrowserregisterkarte, die die **EShopOnWeb**-Website anzeigt.

   > **Hinweis**: Sie haben nun die Anwendung mit CI/CD konfiguriert. In der nächsten Übung richten Sie Quality Gates als Teil einer erweiterten Release-Pipeline ein.

### Übung 3: Konfigurieren von Release-Gates

In dieser Übung richten Sie Quality Gates in der Release-Pipeline ein.

#### Aufgabe 1: Konfigurieren von Gates vor der Bereitstellung für Genehmigungen

In dieser Aufgabe konfigurieren Sie die Gates vor der Bereitstellung.

1. Wechseln Sie zu dem Webbrowser-Fenster, in dem das Azure DevOps-Portal angezeigt wird, und öffnen Sie das **eShopOnWeb**-Projekt. Klicken Sie im vertikalen Navigationsbereich unter **Pipelines** auf **Releases** und dann im Bereich **eshoponweb-cd** auf **Bearbeiten**.
1. Klicken Sie im Bereich **Alle Pipelines > eshoponweb-cd** an den linken Rand des Rechtecks, das für die Phase **DevTest-Umgebung** steht, auf die ovale Form, die die **Bedingungen vor der Bereitstellung** darstellt.
1. Setzen Sie im Bereich **Bedingungen vor der Bereitstellung** den Schieberegler **Genehmigungen vor der Bereitstellung** auf **Aktiviert**, geben Sie in das Textfeld **Genehmigende Personen** Ihren Azure DevOps-Kontonamen ein, und wählen Sie ihn aus.

   > **Hinweis**: In einem realen Szenario sollte dies ein DevOps-Teamnamenalias anstelle Ihres eigenen Namens sein.

1. **Speichern** Sie die Einstellungen vor der Genehmigung, und schließen Sie das Popupfenster.
1. Klicken Sie auf **Release erstellen**, und bestätigen Sie den Vorgang, indem Sie im Popupfenster auf die Schaltfläche **Erstellen** klicken.
1. Achten Sie auf die grüne Bestätigungsmeldung, die besagt, dass „Release-2“ erstellt wurde. Klicken Sie auf den Link „Release-2“, um zu den Details zu navigieren.
1. Die **DevTest**-Phase befindet sich im Status **Genehmigung ausstehend**. Klicken Sie auf die Schaltfläche **Genehmigen**. Dadurch wird die DevTest-Phase erneut ausgelöst.

#### Aufgabe 2: Konfigurieren von Gates nach der Bereitstellung für Azure Monitor

In dieser Aufgabe aktivieren Sie das Gate nach der Bereitstellung für die DevTest-Umgebung.

1. Klicken Sie im Bereich **Alle Pipelines > eshoponweb-cd** am rechten Rand des Rechtecks, das für die **DevTest-Umgebung** steht, auf die ovale Form, die die **Bedingungen nach der Bereitstellung** darstellt.
1. Stellen Sie im Bereich **Bedingungen nach der Bereitstellung** den Schieberegler **Gates** auf **Aktiviert**, klicken Sie auf **+ Hinzufügen** und dann im Popupmenü auf **Azure Monitor-Warnungen abfragen**.
1. Wählen Sie im Bereich **Bedingungen nach der Bereitstellung** im Abschnitt **Azure Monitor-Warnungen abfragen** in der Dropdownliste **Azure-Abonnement** den Eintrag **Serviceverbindung** aus, der für die Verbindung zu Ihrem Azure-Abonnement steht. Wählen Sie dann in der Dropdownliste **Ressourcengruppe** den Eintrag **az400m03l08-RG** aus.
1. Erweitern Sie im Bereich **Bedingungen nach der Bereitstellung** den Abschnitt **Erweitert**, und konfigurieren Sie die folgenden Optionen:

   - Filtertyp: **Kein Filter**
   - Schweregrad: **Sev0, Sev1, Sev2, Sev3, Sev4**
   - Zeitbereich: **Letzte Stunde**
   - Warnungsstatus: **Bestätigt, Neu**
   - Monitorzustand: **Ausgelöst**

1. Erweitern Sie im Bereich **Bedingungen nach der Bereitstellung** die **Auswertungsoptionen**, und konfigurieren Sie die folgenden Optionen:

   - Legen Sie den Wert der **Zeit zwischen erneuter Auswertung von Gates** auf **5 Minuten** fest.
   - Legen Sie den Wert von **Timeout, nachdem ein Gatefehler ausgegeben wird** auf **8 Minuten** fest.
   - Wählen Sie die Option **Bei erfolgreichen Gates Genehmigungen anfordern**.

   > **Hinweis**: Samplingintervall und Timeout arbeiten zusammen, damit die Gates ihre Funktionen in geeigneten Intervallen aufrufen und die Bereitstellung ablehnen, wenn sie nicht während des gleichen Samplingintervalls innerhalb des Timeoutzeitraums erfolgreich sind.

1. Schließen Sie den Bereich **Bedingungen nach der Bereitstellung**, indem Sie oben rechts auf die Markierung **x** klicken.
1. Klicken Sie dann im Bereich **eshoponweb-cd** auf **Speichern** und im Dialogfeld **Speichern** auf **OK**.

### Übung 4: Testen von Releasegates

In dieser Übung testen Sie die Releasegates, indem Sie die Anwendung aktualisieren, wodurch eine Bereitstellung ausgelöst wird

#### Aufgabe 1: Aktualisieren und Bereitstellen der Anwendung nach dem Hinzufügen von Releasegates

In dieser Aufgabe generieren Sie zunächst einige Warnungen für die DevTest-Web-App, gefolgt von der Nachverfolgung des Release-Prozesses mit aktivierten Release-Gates.

1. Navigieren Sie im Azure-Portal zu der zuvor bereitgestellten Ressource **DevTest-Web-App**.
1. Beachten Sie im Übersichtsfenster das Feld **URL**, das den Hyperlink der Webanwendung anzeigt. Klicken Sie auf diesen Link, der Sie an die eShopOnWeb-Webanwendung im Browser weiterleitet.
1. Um eine **Fehlgeschlagene Anfrage** zu simulieren, fügen Sie **/discount** in die URL ein, was zu einer Fehlermeldung führt, da diese Seite nicht existiert. Aktualisieren Sie diese Seite mehrmals, um mehrere Ereignisse zu generieren.
1. Geben Sie im Azure Portal im Feld „Ressourcen, Dienste und Dokumente durchsuchen“ **`Application Insights`** ein und wählen Sie die in der vorherigen Übung erstellte Ressource **DevTest-AppInsights** aus. Navigieren Sie als Nächstes zu **Warnungen**.
1. Es sollte mindestens **1** neue Warnung in der Ergebnisliste sein, mit einem **Schweregrad 2** geben Sie **`Alerts`** ein, um den Warnungsdienst von Azure Monitor zu öffnen.
1. Beachten Sie, dass mindestens **1** Failed_Alert mit **Schweregrad 2: Warnung** in der Liste auftauchen sollte. Dies wurde ausgelöst, als Sie in der vorherigen Übung die nicht existierende URL-Adresse der Website überprüft haben.

   > **Hinweis:** Wenn noch keine Warnung angezeigt wird, warten Sie ein paar Minuten.

1. Kehren Sie zum Azure DevOps-Portal zurück, öffnen Sie das **eShopOnWeb**-Projekt. Navigieren Sie zu **Pipelines**, wählen Sie **Releases** und wählen Sie die **eshoponweb-cd**.
1. Klicken Sie auf die Schaltfläche **Release erstellen**.
1. Warten Sie auf den Start der Release-Pipeline und **genehmigen** Sie die DevTest-Phase-Release-Aktion.
1. Warten Sie, bis die DevTest-Releasephase erfolgreich abgeschlossen ist. Beachten Sie, dass **Gates nach der Bereitstellung** in einen **Auswertungs-Gates**-Status wechselt. Klicken Sie auf das Symbol **Auswertungs-Gates**.
1. Beachten Sie bei **Azure Monitor-Warnungen abfragen** den anfänglichen Fehlerstatus.
1. Lassen Sie die Releasepipeline für die nächsten 5 Minuten im Status „Ausstehend“. Beachten Sie nach Ablauf der 5 Minuten, dass die zweite Auswertung erneut fehlschlägt.
1. Dies ist zu erwarten, da eine Application Insights-Warnung für die DevTest Web App ausgelöst wurde.

   > **Hinweis**: Da durch die Ausnahme eine Warnung ausgelöst wird, schlägt das Gate **Azure Monitor abfragen** fehl. Dies wiederum verhindert die Bereitstellung in der **Produktionsumgebung**.

1. Warten Sie ein paar Minuten und überprüfen Sie den Status der Releasegates erneut. Wenige Minuten nach der Überprüfung der ersten Releasegates und nachdem die erste Application Insight-Warnung mit der Aktion „Ausgelöst“ ausgelöst wurde, sollte ein erfolgreiches Releasegate vorliegen, das die Bereitstellung der Produktionsrelease-Phase ermöglicht.

   > **Hinweis:** Wenn Ihr Gate fehlschlägt, schließen Sie die Warnung.

   > [!IMPORTANT]
   > Denken Sie daran, die im Azure-Portal erstellten Ressourcen zu löschen, um unnötige Gebühren zu vermeiden.

## Überprüfung

In diesem Lab haben Sie Releasepipelines konfiguriert und anschließend Releasegates konfiguriert und getestet.
