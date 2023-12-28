---
lab:
  title: 'Lab: Steuern von Bereitstellungen mithilfe von Releasegates'
  module: 'Module 04: Design and implement a release strategy'
---

# Lab: Steuern von Bereitstellungen mithilfe von Releasegates

## Lab-Handbuch für Kursteilnehmer

## Labanforderungen

- Für dieses Lab ist **Microsoft Edge** oder ein [von Azure DevOps unterstützter Browser](https://docs.microsoft.com/azure/devops/server/compatibility) erforderlich.

- **Einrichten einer Azure DevOps-Organisation**: Wenn Sie nicht bereits eine Azure DevOps-Organisation haben, die Sie für dieses Lab verwenden können, müssen Sie diese erstellen, indem Sie die unter [Erstellen einer Organisation oder Projektsammlung](https://docs.microsoft.com/azure/devops/organizations/accounts/create-organization) beschriebenen Anweisungen befolgen.

- Identifizieren Sie ein vorhandenes Azure-Abonnement, oder erstellen Sie ein neues Abonnement.

- Stellen Sie sicher, dass Sie über ein Microsoft- oder Microsoft Entra-Konto mit der Rolle „Besitzer“ im Azure-Abonnement und der Rolle „Globaler Administrator“ im Microsoft Entra-Mandanten verfügen, der dem Azure-Abonnement zugeordnet ist. Ausführliche Informationen finden Sie in den Artikeln zum [Auflisten von Azure-Rollenzuweisungen mithilfe des Azure-Portals](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-list-portal) und [Anzeigen und Zuweisen von Administratorrollen in Azure Active Directory](https://docs.microsoft.com/azure/active-directory/roles/manage-roles-portal).

## Übersicht über das Labor

In diesem Lab werden die Konfiguration der Bereitstellungsgates sowie die Steuerung der Ausführung von Azure Pipelines behandelt. Um die Implementierung zu veranschaulichen, konfigurieren Sie eine Releasedefinition mit zwei Umgebungen für eine Azure-Web-App. Sie führen nur dann eine Bereitstellung in der Canary-Umgebung durch, wenn keine Fehler für die App vorhanden sind, die eine Bereitstellung blockieren würden, und Sie markieren die Canary-Umgebung nur als abgeschlossen, wenn keine aktiven Warnungen im Azure Monitor-Feature Application Insights vorhanden sind.

Eine Releasepipeline gibt den End-to-End-Releaseprozess für eine Anwendung an, die in verschiedenen Umgebungen bereitgestellt werden soll. Die Bereitstellungen in den einzelnen Umgebungen erfolgen vollständig automatisiert über Aufträge und Tasks. Im Idealfall sollten neue Updates für die Anwendungen nicht gleichzeitig für alle Benutzer verfügbar gemacht werden. Es hat sich bewährt, bei Updates phasenweise vorzugehen: Ein Update wird zunächst nur für eine Teilmenge von Benutzern verfügbar gemacht, deren Nutzung wird überwacht, und dann wird das Update anderen Benutzern basierend auf der Erfahrung der ersten Benutzergruppe zur Verfügung gestellt.

Mit Genehmigungen und Gates haben Sie vollständige Kontrolle über den Beginn und Abschluss der Bereitstellungen in einem Release. Sie können warten, bis Benutzer Bereitstellungen manuell genehmigen oder ablehnen. Mithilfe von Releasegates können Sie Kriterien für die Anwendungsintegrität angeben, die erfüllt werden müssen, bevor das Release in die nächste Umgebung weitergegeben wird. Vor oder nach einer Umgebungsbereitstellung werden alle angegebenen Gates automatisch ausgewertet, bis sie den von Ihnen definierten Timeoutzeitraum erreichen oder überschreiten, wonach ein Fehler auftritt.

Gates können einer Umgebung in der Releasedefinition im Bereich der Bedingungen vor der Bereitstellung oder im Bereich der Bedingungen nach der Bereitstellung hinzugefügt werden. Den Umgebungsbedingungen können mehrere Gates hinzugefügt werden, um sicherzustellen, dass alle Eingaben für das Release erfolgreich sind.

Beispiel:

- Gates vor der Bereitstellung stellen sicher, dass im Arbeitselement- oder Problemverwaltungssysstem keine aktiven Probleme vorhanden sind, bevor ein Build in einer Umgebung bereitgestellt wird.
- Gates nach der Bereitstellung stellen sicher, dass im Überwachungs- oder Incidentverwaltungssystem der App keine Incidents vorhanden sind, bevor sie das Release in die nächste Umgebung weitergeben.

In jedem Konto sind standardmäßig vier Arten von Gates enthalten.

- Azure Function aufrufen: Hiermit wird die Ausführung einer Azure Function ausgelöst und sichergestellt, dass sie erfolgreich abgeschlossen wird.
- Azure Monitor-Warnungen abfragen: Hiermit werden die konfigurierten Azure Monitor-Warnungsregeln für aktive Warnungen beobachtet.
- REST-API aufrufen: Hiermit wird eine REST-API aufgerufen, der Prozess wird fortgesetzt, wenn eine Erfolgsantwort zurückgesendet wird.
- Arbeitselemente abfragen: Hiermit wird sichergestellt, dass die Anzahl der von einer Abfrage zurückgegebenen übereinstimmenden Arbeitselemente innerhalb eines bestimmten Schwellenwerts liegt.

## Ziele

In diesem Lab lernen Sie Folgendes:

- Konfigurieren Sie Releasepipelines.
- Konfigurieren Sie Releasegates.
- Testen Sie Releasegates.

## Geschätzte Zeit: 15 Minuten

## Anweisungen

### Übung 0: Konfigurieren der Voraussetzungen für das Lab.

> **Hinweis**: Wenn Sie dieses Projekt bereits in früheren Laboren erstellt haben, kann diese Übung übersprungen werden.

In dieser Übung richten Sie die Voraussetzungen für das Labor ein, das aus einem neuen Azure DevOps-Projekt mit einem Repository basierend auf dem [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb) besteht.

#### Aufgabe 1: (überspringen, falls bereits erledigt) Erstellen und Konfigurieren des Teamprojekts

In dieser Aufgabe erstellen Sie ein **eShopOnWeb** Azure DevOps-Projekt, das von mehreren Laboren verwendet werden soll.

1. Öffnen Sie auf Ihrem Laborcomputer in einem Browserfenster Ihre Azure DevOps-Organisation. Klicken auf „Neues Projekt“ Weisen Sie Ihrem Projekt den Namen **"eShopOnWeb** " zu, und lassen Sie die anderen Felder standardmäßig. Klicken Sie auf **Erstellen**.

    ![Erstellen eines Projekts](images/create-project.png)

#### Aufgabe 2: (überspringen, falls bereits erledigt) Importieren von eShopOnWeb Git Repository

Bei dieser Aufgabe importieren Sie das eShopOnWeb Git-Repository, das von mehreren Labs verwendet wird.

1. Öffnen Sie auf Ihrem Laborcomputer in einem Browserfenster Ihre Azure DevOps-Organisation und das zuvor erstellte **eShopOnWeb-Projekt** . Klicken Sie auf **Repos>Files** , **Import a Repository**. Wählen Sie **Importieren** aus. Fügen Sie im **Fenster "Git Repository** importieren" die folgende URL https://github.com/MicrosoftLearning/eShopOnWeb.git  ein, und klicken Sie auf " **Importieren**":

    ![Importieren eines Repositorys](images/import-repo.png)

2. Das Repository ist wie folgt organisiert:
    - Der Ordner „.ado“ enthält Azure DevOps-YAML-Pipelines.
    - Der Ordner „.devcontainer“ enthält ein Containersetup für die Entwicklung mithilfe von Containern (entweder lokal in VS Code oder über GitHub Codespaces).
    - **Azure-Ordner** enthält Bicep&ARM-Infrastruktur als Codevorlagen, die in einigen Lab-Szenarien verwendet werden.
    - **GITHUB-Ordnercontainer-YAML-GitHub-Workflowdefinitionen** .
    - Der Ordner „src“ enthält die .NET 6-Website, die in den Labszenarien verwendet wird.

#### Aufgabe 3: (überspringen, falls bereits geschehen) Konfigurieren der CI-Pipeline als Code mit YAML in Azure DevOps

In diesem Vorgang fügen Sie dem vorhandenen Projekt eine YAML-Builddefinition hinzu.

1. Navigieren Sie zurück zum **Bereich "Pipelines** " im **Pipelinehub** .
2. Klicken Sie im **Fenster "Erste Pipeline** erstellen" auf **"Pipeline erstellen"**.

    > **Hinweis**: Wir verwenden den Assistenten, um eine neue YAML-Pipelinedefinition basierend auf unserem Projekt zu erstellen.

3. Klicken Sie im **Bereich "Wo befindet sich Ihr Code?** " auf die **Option "Azure Repos Git (YAML)** ".
4. Klicken Sie im **Bereich "Repository** auswählen" auf **"eShopOnWeb**".
5. Wählen Sie auf dem Bildschirm **Pipeline konfigurieren** die Option **Vorhandene Azure Pipelines-YAML-Datei** aus.
6. Geben Sie im **Blatt "Auswählen einer vorhandenen YAML-Datei** " die folgenden Parameter an:
   - Branch: **main**
   - Pfad: **.ado/eshoponweb-ci.yml**
7. Klicken Sie auf **Konfigurieren**, um die Einstellungen zu speichern.
8. Klicken Sie auf dem **Bildschirm "Pipeline überprüfen** " auf **"Ausführen** ", um den Buildpipelineprozess zu starten.
9. Warten Sie auf den Abschluss der Buildpipeline. Ignorieren Sie alle Warnungen bezüglich des Quellcodes selbst, da sie für diese Übung nicht relevant sind.

    > **Hinweis**: Jede Aufgabe aus der YAML-Datei steht zur Überprüfung zur Verfügung, einschließlich aller Warnungen und Fehler.

10. Ihre Pipeline nimmt einen Namen auf der Grundlage des Projektnamens an. **Benennen** wir sie um, um die Pipeline besser zu identifizieren. Wechseln Sie zu **Pipelines>Pipelines** , und klicken Sie auf die kürzlich erstellte Pipeline. Klicken Sie auf die Auslassungspunkte und **die Option "Umbenennen/Verschieben** ". Nennen Sie es **eshoponweb-ci** , und klicken Sie auf **"Speichern"**.

### Übung 2: Erstellen der erforderlichen Azure-Ressourcen für die Releasepipeline

#### Aufgabe 1: Erstellen einer Azure-Web-App

In dieser Aufgabe erstellen Sie zwei Azure-Web-Apps, die die **DevTest** - und **Produktionsumgebungen** darstellen, in denen Sie die Anwendung über Azure-Pipelines bereitstellen.

1. Starten Sie auf Ihrem Labcomputer einen Webbrowser, navigieren Sie zum [**Azure-Portal**](https://portal.azure.com), und melden Sie sich an. Verwenden Sie hierzu die Anmeldeinformationen eines Benutzerkontos, das in dem Abonnement, das Sie in diesem Lab verwenden, über die Rolle „Besitzer“ und in dem Azure AD-Mandanten, der dem Azure-Abonnement zugeordnet ist, über die Rolle „Globaler Administrator“ verfügt.
2. Öffnen Sie im Azure-Portal den Bereich **Cloud Shell**, indem Sie auf das Symbolleistensymbol rechts neben dem Textfeld für die Suche klicken.
3. Wählen Sie bei Aufforderung zur Auswahl von **Bash** oder **PowerShell** die Option **Bash** aus.

   >**Hinweis**: Wenn Sie **Cloud Shell** zum ersten Mal starten und die Meldung **Für Sie wurde kein Speicher bereitgestellt** angezeigt wird, wählen Sie das in diesem Lab verwendete Abonnement aus, und klicken Sie dann auf **Speicher erstellen**.

4. Führen Sie in der **Bash-Eingabeaufforderung** im **Cloud Shell-Bereich** den folgenden Befehl aus, um eine Ressourcengruppe zu erstellen (ersetzen Sie den `<region>` Variablenplatzhalter durch den Namen der Azure-Region, die die beiden Azure-Web-Apps hosten soll, z. B. "westeurope" oder "centralus" oder eine andere verfügbare Region Ihrer Wahl):

    > **Hinweis**: Mögliche Speicherorte werden gefunden, indem Sie den folgenden Befehl ausführen, verwenden Sie den **Namen** unter `<region>` : `az account list-locations -o table`

    ```bash
    REGION='centralus'
    RESOURCEGROUPNAME='az400m04l09-RG'
    az group create -n $RESOURCEGROUPNAME -l $REGION
    ```

5. Wie erstelle ich einen Plan?

    ```bash
    SERVICEPLANNAME='az400m04l09-sp1'
    az appservice plan create -g $RESOURCEGROUPNAME -n $SERVICEPLANNAME --sku S1
    ```

6. Erstellen Sie zwei Web-Apps mit eindeutigen App-Namen.

     ```bash
     SUFFIX=$RANDOM$RANDOM
     az webapp create -g $RESOURCEGROUPNAME -p $SERVICEPLANNAME -n RGATES$SUFFIX-DevTest
     az webapp create -g $RESOURCEGROUPNAME -p $SERVICEPLANNAME -n RGATES$SUFFIX-Prod
     ```

    > **Hinweis: Notieren** Sie den Namen der DevTest-Web-App. Sie benötigen ihn später in diesem Lab.

7. Warten Sie, bis der Bereitstellungsprozess für Web App Services-Ressourcen abgeschlossen ist, und schließen Sie den **Cloud Shell-Bereich** .

#### Aufgabe 2: Erstellen einer Application Insights-Ressource

1. Verwenden Sie im Azure-Portal das Textfeld **Ressourcen, Dienste und Dokumente durchsuchen** oben auf der Seite, um nach **Application Insights** zu suchen. Wählen Sie in der Ergebnisliste dann **Application Insights** aus.
2. Klicken Sie im Blatt **Application Insights** auf **+ Erstellen**.
3. Geben Sie auf der Registerkarte **Grundlagen** des Blatts **Web-App erstellen** die folgenden Einstellungen an (übernehmen Sie für andere Einstellungen die Standardwerte):

    | Einstellung | Wert |
    | --- | --- |
    | Ressourcengruppe | **az400m04l09-RG** |
    | Name | der Name der DevTest-Web-App, die Sie in der vorherigen Aufgabe aufgezeichnet haben |
    | Region | Die gleiche Azure-Region, in der Sie die Ressourcen in der vorherigen Aufgabe dieser Übung bereitgestellt haben |
    | Ressourcenmodus | **Klassisch** |

    > **Hinweis**: Ignorieren Sie die Veraltetkeitsnachricht. Dies ist erforderlich, um Fehler der DevOps-Aufgabe "Fortlaufende Integration aktivieren" zu verhindern, die Sie später in dieser Übung verwenden werden.

4. Klicken Sie auf **Überprüfen + erstellen** und dann auf **Erstellen**.
5. Warten Sie auf den Abschluss des Bereitstellungsvorgangs.
6. Navigieren Sie im Azure-Portal zur Ressourcengruppe **az400m04l09-RG**, die Sie im vorherigen Vorgang erstellt haben.
7. Wählen Sie die soeben erstellte Web-App in der Liste der Ressourcen aus.
8. Klicken Sie auf der **DevTest-Web-App-Seite** im vertikalen Menü auf der linken Seite im **Abschnitt Einstellungen** auf **Application Insights**.
9. Wählen Sie im Bereich **Application Insights** die Option **Application Insights aktivieren** aus.
10. Klicken Sie im **Abschnitt "Ressource ändern**" auf die **Option "Vorhandene Ressource** auswählen", wählen Sie in der Liste der vorhandenen Ressourcen die neu erstellte Application Insight-Ressource aus, klicken Sie auf "Übernehmen", und klicken Sie **auf "Ja****", wenn Sie **zur Bestätigung aufgefordert werden.
11. Warten Sie, bis die Änderung wirksam wird.

    > **Hinweis**: Sie erstellen hier Warnungen, die Sie später in diesem Labor verwenden werden.
12. Wählen Sie in derselben **Einstellungen**** /  Application Insights-Menüoption** in der Web App die Option **"Application Insights-Daten** anzeigen" aus. Das Blatt Application Insights wird im Azure-Portal geöffnet.
13. Klicken Sie auf dem Ressourcenblatt "Application Insights" im **Abschnitt "Überwachung** " auf **"Warnungen** ", und klicken Sie dann auf **"> Warnungsregel erstellen"**.
14. Geben **Sie auf dem **Blatt "Signal** auswählen" im **Textfeld "Nach Signalname** suchen" "Anforderungen"** ein. Wählen Sie in der Ergebnisliste die Option "Fehlgeschlagene Anforderungen"** aus**.
15. Lassen Sie auf dem **Blatt "Warnungsregel** erstellen" im **Abschnitt "Bedingung** " den **Schwellenwert** auf **"Statisch**" festgelegt, und überprüfen Sie die anderen Standardeinstellungen wie folgt:
    - Aggregationstyp: Count (Anzahl)
    - Operator: Größer als
    - Einheit: Anzahl
16. Geben Sie **im **Textfeld "Schwellenwert"** "0"** ein, und klicken Sie auf **"Weiter:Aktionen**". Nehmen Sie keine Änderungen am **Blatt "Aktionen** "-Einstellungen vor, und definieren Sie die folgenden Parameter im **Abschnitt "Details** ":

    | Einstellung | Wert |
    | --- | --- |
    | severity | 2: Warnung |
    | Name der Warnungsregel | **RGATESDevTest_FailedRequests** |
    | Erweiterte Optionen: Automatisches Auflösen von Warnungen | Deaktiviert |

    > **Hinweis**: Metrische Warnungsregeln können bis zu 10 Minuten dauern, bis sie aktiviert werden.

    > **Hinweis**: Sie können mehrere Warnungsregeln für unterschiedliche Metriken erstellen, z. B. verfügbarkeit < 99 Prozent, Serverantwortzeit > 5 Sekunden oder Server exceptions > 0

17. Bestätigen Sie die Erstellung der Warnungsregel, indem Sie auf "Überprüfen+Erstellen **" klicken **und erneut bestätigen, indem Sie auf "Erstellen"** klicken**. Warten Sie, bis die Warnungsregel erfolgreich erstellt wurde.

### Übung 2: Konfigurieren der Releasepipeline

In dieser Übung konfigurieren Sie eine Releasepipeline.

#### Aufgabe 1: Einrichten von Freigabeaufgaben

In dieser Aufgabe richten Sie die Veröffentlichungsaufgaben als Teil der Releasepipeline ein.

1. Wählen Sie **im Azure DevOps-Portal im eShopOnWeb-Projekt** im vertikalen Navigationsbereich Pipelines** aus, und klicken Sie dann im **Abschnitt "Pipelines**" auf **"Releases"**.**
2. Klicken Sie auf **Neue Pipeline**.
3. Wählen Sie im **Fenster "Vorlage** auswählen****" **Azure-App Dienstbereitstellung** aus (Bereitstellen der Anwendung für Azure-App Dienst. Wählen Sie aus Web App on Windows, Linux, Containern, Funktions-Apps oder WebJobs.
4. Klicken Sie auf **Übernehmen**.
5. Aktualisieren Sie im angezeigten Fenster " **Phase** 1" den Standardnamen "Phase 1" auf **DevTest**. Schließen Sie das Popupfenster mithilfe der **Schaltfläche "X** ". Sie befinden sich jetzt im grafischen Editor der Releasepipeline mit der DevTest-Phase.
6. Benennen Sie oben auf der Seite die aktuelle Pipeline von **der New Release-Pipeline** in **eshoponweb-cd** um.
7. Zeigen Sie mit der Maus auf die DevTest-Phase, und klicken Sie auf die **Schaltfläche "Klonen** ", um die DevTest-Phase in eine zusätzliche Phase zu kopieren. Nennen Sie diese Phasenproduktion****.

    > **Hinweis**: Die Pipeline enthält jetzt zwei Stufen namens **DevTest** und **Production**.

8. Wählen Sie auf der **Registerkarte "Pipeline**" das **Rechteck "Artefakt** hinzufügen" aus, und wählen Sie das ****Feld "eshoponweb-ci**" im Feld "Quelle" (Buildpipeline)** aus. Klicken Sie auf **"Hinzufügen"** , um die Auswahl des Artefakts zu bestätigen.
9. Beachten Sie aus dem **Artefaktrechteck** den Auslöser** für die **kontinuierliche Bereitstellung (Blitz). Legen Sie dann **Continuous Deployment-Trigger** auf „Aktiviert“ fest. Klicken Sie auf **"Deaktiviert** ", um den Schalter umzuschalten und ihn zu aktivieren. Behalten Sie alle anderen Einstellungen standardmäßig bei, und schließen Sie den **Auslöserbereich** für die kontinuierliche Bereitstellung, indem Sie in der oberen rechten Ecke auf das **X-Zeichen** klicken.
10. Klicken Sie in der **Phase "DevTest-Umgebungen** " auf den **Auftrag, die Aufgabenbezeichnung 1** , und überprüfen Sie die Aufgaben in dieser Phase.

    > **Hinweis**: Die DevTest-Umgebung hat 1 Aufgabe, die das Artefaktpaket in Azure Web App veröffentlicht.

11. **Stellen Sie im Bereich "Alle Pipelines > eshoponweb-cd**" sicher, dass die **DevTest-Phase** ausgewählt ist. Wählen Sie in der Dropdownliste Abonnement Ihr Azure-Abonnement aus. Wenn Sie dazu aufgefordert werden, authentifizieren Sie sich mithilfe des Benutzerkontos mit der Rolle "Besitzer" im Azure-Abonnement.
12. Vergewissern Sie sich, dass der App-Typ auf "Web App unter Windows" festgelegt ist. Wählen Sie als Nächstes in der **Dropdownliste "App Service-Name** " den Namen der **DevTest-Web-App** aus.
13. Wählen Sie die Azure Cloud Services-Bereitstellungsaufgabe aus. Aktualisieren Sie im **Feld "Paket" oder "Ordner**" den Standardwert "$(System.DefaultWorkingDirectory)//\*\*\*.zip" auf "$(System.DefaultWorkingDirectory)/\*\*/Web.zip".

    > beachten Sie ein Ausrufezeichen neben der Registerkarte "Vorgänge". Dies wird erwartet, da wir die Einstellungen für die Produktionsstufe konfigurieren müssen.

14. **Navigieren Sie im Bereich "Alle Pipelines" > Bereich "eshoponweb-cd**" zur **Registerkarte "Pipeline**", und klicken Sie dieses Mal in der **Produktionsstufe** auf den **Auftrag 1, 1 Aufgabenbeschriftung**. Ähnlich wie in der DevTest-Phase zuvor schließen Sie die Pipelineeinstellungen ab. Wählen Sie auf der Registerkarte "Aufgaben" /Produktionsbereitstellungsprozess in der Dropdownliste des **Azure-Abonnements** das Azure-Abonnement aus, das Sie für die **DevTest-Umgebungsphase** verwendet haben, die unter **"Verfügbare Azure Service-Verbindungen**" angezeigt wird, da wir die Dienstverbindung bereits erstellt haben, bevor wir die Abonnementnutzung autorisieren.
15. Wählen Sie in der **Dropdownliste "App Service-Name** " den Namen der **Prod** Web App aus.
16. Wählen Sie die Azure Cloud Services-Bereitstellungsaufgabe aus. Aktualisieren Sie im **Feld "Paket" oder "Ordner**" den Standardwert "$(System.DefaultWorkingDirectory)//\*\*\*.zip" auf "$(System.DefaultWorkingDirectory)/\*\*/Web.zip".
17. Klicken Sie im **Bereich "Alle Pipelines > eshoponweb-cd** " auf " **Speichern** ", und klicken Sie im **Dialogfeld "Speichern** " auf **"OK**".

    Sie haben jetzt die Releasepipeline erfolgreich konfiguriert.

18. Klicken Sie im Browserfenster mit dem **eShopOnWeb-Projekt** im vertikalen Navigationsbereich im **Abschnitt "Pipelines** " auf **"Pipelines**".
19. Klicken Sie im **Bereich "Pipelines**" auf den Eintrag, der die Buildpipeline "eshoponweb-ci **" darstellt**, und klicken Sie dann auf "**Pipeline ausführen"**.
20. Übernehmen Sie im **Bereich "Pipeline** ausführen" die Standardeinstellungen, und klicken Sie auf **"Ausführen** ", um die Pipeline auszulösen. Warten Sie, bis die Pipelineausführung abgeschlossen ist.

    > **Hinweis**: Nachdem der Build erfolgreich war, wird die Version automatisch ausgelöst, und die Anwendung wird in beiden Umgebungen bereitgestellt. Überprüfen Sie die Releaseaktionen, nachdem die Buildpipeline erfolgreich abgeschlossen wurde.

21. Klicken Sie im vertikalen Navigationsbereich im **Abschnitt "Pipelines** " auf **"Releases** ", und klicken Sie im **Bereich "eshoponweb-cd** " auf den Eintrag, der die neueste Version darstellt.
22. Verfolgen Sie auf dem **Blatt eshoponweb-cd > Release-1** den Fortschritt der Version nach, und überprüfen Sie, ob die Bereitstellung für beide Web-Apps erfolgreich abgeschlossen wurde.
23. Wechseln Sie zur Azure-Portal Schnittstelle, navigieren Sie zur Ressourcengruppe **az400m04l09-RG**, klicken Sie in der Liste der Ressourcen auf die **DevTest-Web-App**, klicken Sie im Web-App-Blatt auf **"Durchsuchen**", und vergewissern Sie sich, dass die Webseite (E-Commerce-Website) erfolgreich in einer neuen Webbrowserregisterkarte geladen wird.
24. Wechseln Sie zurück zur Azure-Portal Schnittstelle, diesmal navigieren Sie zur Ressourcengruppe **az400m04l09-RG**, klicken Sie in der Liste der Ressourcen auf die **Produktionsweb-App**, klicken Sie im Web-App-Blatt auf **"Durchsuchen**", und vergewissern Sie sich, dass die Webseite erfolgreich auf einer neuen Webbrowserregisterkarte geladen wird.
25. Schließen Sie die Registerkarte des Webbrowsers, auf der die **EShopOnWeb-Website** angezeigt wird.

    > **Hinweis**: Jetzt haben Sie die Anwendung mit CI/CD konfiguriert. In der nächsten Übung werden wir Quality Gates als Teil einer erweiterten Releasepipeline einrichten.

### Übung 3: Konfigurieren von Releasegates

In dieser Übung richten Sie Quality Gates in der Releasepipeline ein.

#### Aufgabe 1: Konfigurieren von Gates vor der Bereitstellung für Genehmigungen

In dieser Aufgabe konfigurieren Sie Vorbereitstellungsgates.

1. Öffnen Sie im Webbrowserfenster, in dem das Azure DevOps-Projekt angezeigt wird, eine weitere Webbrowserregisterkarte, und navigieren Sie zum **Azure-Portal**. Klicken Sie im vertikalen Navigationsbereich im **Abschnitt "Pipelines** " auf **"Versionen"** , und klicken Sie im **Bereich "eshoponweb-cd** " auf " **Bearbeiten"**.
2. **Klicken Sie im Bereich "Alle Pipelines" > eshoponweb-cd** am linken Rand des Rechtecks, das die **DevTest Environment-Phase** darstellt, auf das ovale Shape, das die Bedingungen** vor der **Bereitstellung darstellt.
3. **Legen Sie im Bereich "Bedingungen** vor der Bereitstellung" den Schieberegler "Genehmigungen** vor der **Bereitstellung" auf **"Aktiviert**" fest, und geben Sie im **Textfeld "Genehmigende Personen**" Ihren Azure DevOps-Kontonamen ein, und wählen Sie ihn aus.

    > **Hinweis**: In einem realen Szenario sollte dies ein DevOps-Teamnamenalias anstelle Ihres eigenen Namens sein.

4. **Speichern Sie die Einstellungen vor der Genehmigung, und schließen Sie** das Popupfenster.
5. Klicken Sie auf "Freigeben erstellen **", und bestätigen Sie**, indem Sie im Popupfenster auf die **Schaltfläche "Erstellen**" klicken.
6. Beachten Sie die grüne Bestätigungsmeldung, die besagt, dass "Release-2" erstellt wurde. Klicken Sie auf den Link "Release-2", um zu den Details zu navigieren.
7. Beachten Sie, dass sich die **DevTest-Phase** im **Status "Ausstehende Genehmigung"** befindet. Klicken Sie auf die Schaltfläche **Speichern**. Dadurch wird die DevTest-Phase erneut deaktiviert.

#### Aufgabe 2: Konfigurieren von Gates nach der Bereitstellung für Azure Monitor

In dieser Aufgabe aktivieren Sie das Gate nach der Bereitstellung für die DevTest-Umgebung.

1. Zurück auf die **Pipelines > eshoponweb-cd** bereich, klicken Sie am rechten Rand des Rechtecks, das die **DevTest-Umgebungsstufe** darstellt, auf das ovale Shape, das die Bedingungen** nach der **Bereitstellung darstellt.
2. **Legen Sie im Bereich "Bedingungen** nach der Bereitstellung" den **Schieberegler "Gates**" auf **"Aktiviert**" fest, klicken Sie auf **"+Hinzufügen**", und klicken Sie im Popupmenü auf "**Azure Monitor-Benachrichtigungen** abfragen".
3. **Wählen Sie im Bereich "Bedingungen** nach der Bereitstellung" im **Abschnitt "Azure Monitor-Benachrichtigungen** abfragen" in der **Dropdownliste für Azure-Abonnements** den **Dienstverbindungseintrag** aus, der die Verbindung mit Ihrem Azure-Abonnement darstellt, und wählen Sie in der **Dropdownliste "Ressourcengruppe**" den **Eintrag "az400m04l09-RG**" aus.
4. Erweitern Sie im Bereich "Bedingungen** nach der **Bereitstellung" den **Abschnitt "Erweitert**", und konfigurieren Sie die folgenden Optionen:

   - Der Filtertyp.
   - Sev0, Sev1, Sev2, Sev3, Sev4
   - Zeitbereich : *Letzte Stunde*
   - Warnungsstatus: **Bestätigt, neu**
   - Monitorbedingung 0

5. **Erweitern Sie im Bereich "Bedingungen** nach der Bereitstellung" die **Auswertungsoptionen**, und konfigurieren Sie die folgenden Optionen:

   - Legen Sie den Wert der **Zeit zwischen erneuter Auswertung von Toren** auf **5 Minuten** fest.
   - Legen Sie den Wert des **Timeouts fest, nach dem Tore 8 Minuten** nicht** ausgeführt werden**.
   - Bei erfolgreichen Gates Genehmigungen anfordern

    > **Hinweis**: Das Samplingintervall und timeout arbeiten zusammen, damit die Gates ihre Funktionen in geeigneten Intervallen aufrufen und die Bereitstellung ablehnen, wenn sie nicht innerhalb des gleichen Samplingintervalls innerhalb des Timeoutzeitraums erfolgreich sind.

6. Schließen Sie den Bereich "Bedingungen** nach der **Bereitstellung", indem Sie in der oberen rechten Ecke auf das **X-Zeichen** klicken.
7. Klicken Sie wieder im **Bereich "eshoponweb-cd**" auf "**Speichern", und klicken Sie im **Dialogfeld "Speichern****" auf **"OK**".

### Übung 4: Testen von Releasegates

In dieser Übung testen Sie die Freigabetore, indem Sie die Anwendung aktualisieren, wodurch eine Bereitstellung ausgelöst wird.

#### Aufgabe 1: Aktualisieren und Bereitstellen der Anwendung nach dem Hinzufügen von Releasegates

In dieser Aufgabe generieren Sie zunächst einige Warnungen für die DevTest Web App, gefolgt von der Nachverfolgung des Veröffentlichungsprozesses mit aktivierten Freigabegaten.

1. Navigieren Sie im Azure-Portal zu der **zuvor bereitgestellten DevTest Web App-Ressource** .
2. Beachten Sie im Bereich "Übersicht" das **URL-Feld** mit dem Link der Webanwendung. Klicken Sie auf diesen Link, der Sie an die EShopOnWeb-Webanwendung im Browser weiterleitet.
3. Um eine **fehlgeschlagene Anforderung** zu simulieren, fügen Sie **der URL "/discount** " hinzu, was zu einer Fehlermeldung führt, da diese Seite nicht vorhanden ist. Aktualisieren Sie diese Seite mehrmals, um mehrere Ereignisse zu generieren.
4. Geben Sie **im Azure-Portal im Feld "Ressourcen, Dienste und Dokumente durchsuchen" Application Insights** ein, und wählen Sie die **in der vorherigen Übung erstellte DevTest-AppInsights-Ressource** aus. Navigieren Sie als Nächstes zu **"Warnungen"**.
5. Es sollte mindestens **1** neue Warnung in der Liste der Ergebnisse vorhanden sein, wobei der **Schweregrad 2** "Warnungen **" eingibt**, um den Warnungsdienst von Azure Monitor zu öffnen.
6. Beachten Sie, dass mindestens **1** Failed_Alert mit **Schweregrad 2 vorhanden sein sollte – Warnung** , die in der Liste angezeigt wird. Dieser Trigger wurde ausgelöst, wenn Sie die url-Adresse der nicht vorhandenen Website in der vorherigen Übung überprüft haben.

    > **Hinweis:** Wenn noch keine Warnung angezeigt wird, warten Sie ein paar Minuten.

7. Kehren Sie zum Azure DevOps-Portal zurück, öffnen Sie das **EShopOnWeb-Projekt** . Navigieren Sie zu **Pipelines**, wählen Sie **"Releases"** aus, und wählen Sie die **eshoponweb-cd** aus.
8. Klicken Sie auf die Schaltfläche „Create a new release“ (Neues Release erstellen).
9. Warten Sie, bis die Releasepipeline gestartet wird, und **genehmigen Sie** die DevTest-Phase-Veröffentlichungsaktion.
10. Warten Sie, bis die DevTest-Versionsphase erfolgreich abgeschlossen wurde. Beachten Sie, wie die Gates nach der **Bereitstellung** zu einem **Bewertungsgate-Status** wechseln.  Klicken Sie auf das **Symbol "Bewertungsgate"** .
11. Beachten Sie für die **Azure Monitor-Abfragebenachrichtigungen einen anfänglichen fehlgeschlagenen** Zustand.
12. Lassen Sie die Releasepipeline in einem ausstehenden Zustand für die nächsten 5 Minuten. Beachten Sie nach ablauf der 5 Minuten, dass die 2. Auswertung erneut fehlschlägt.
13. Dies wird erwartet, da für die DevTest Web App eine Application Insights-Warnung ausgelöst wird.

    > **Hinweis**: Da eine warnung ausgelöst wird, die von der Ausnahme ausgelöst wird, **schlägt das Abfrage-Azure Monitor-Gate** fehl. Dadurch wird wiederum die Bereitstellung in der **Produktionsumgebung** verhindert.

14. Warten Sie ein paar Minuten, und überprüfen Sie den Status der Release Gates erneut. Innerhalb weniger Minuten nach der Überprüfung der ersten Release Gates, und da die anfängliche Application Insight Alert mit der Aktion "Ausgelöst" ausgelöst wurde, sollte es zu einem erfolgreichen Release Gate führen, was die Bereitstellung der Produktionsversionsphase zulässt.

    > **Hinweis:** Wenn Ihr Tor fehlschlägt, schließen Sie die Warnung.

### Übung 4: Entfernen der Azure Lab-Ressourcen

In dieser Übung entfernen Sie die in dieser Übung bereitgestellten Azure-Ressourcen, um unerwartete Gebühren zu beseitigen.

>**Hinweis**: Denken Sie daran, alle neu erstellten Azure-Ressourcen zu entfernen, die Sie nicht mehr verwenden. Durch das Entfernen nicht verwendeter Ressourcen wird sichergestellt, dass keine unerwarteten Kosten anfallen.

#### Übung 4: Entfernen der Azure Lab-Ressourcen

In dieser Aufgabe verwenden Sie Azure Cloud Shell, um die in dieser Übung bereitgestellten Azure-Ressourcen zu entfernen, um unnötige Gebühren zu vermeiden.

1. Öffnen Sie im Azure-Portal im **Cloud Shell**-Bereich die **Bash**-Sitzung.
1. Listen Sie alle Ressourcengruppen auf, die während der Labs in diesem Modul erstellt wurden, indem Sie den folgenden Befehl ausführen:

    ```sh
    az group list --query "[?starts_with(name,'az400m04l09-RG')].name" --output tsv
    ```

1. Löschen Sie alle Ressourcengruppen, die Sie während der praktischen Übungen in diesem Modul erstellt haben, indem Sie den folgenden Befehl ausführen:

    ```sh
    az group list --query "[?starts_with(name,'az400m04l09-RG')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**Hinweis**: Der Befehl wird (dem --nowait-Parameter entsprechend) asynchron ausgeführt. Dies bedeutet, dass Sie zwar einen weiteren Azure CLI-Befehl in derselben Bash-Sitzung direkt im Anschluss ausführen können, es jedoch einige Minuten dauert, bis die Ressourcengruppen tatsächlich entfernt werden.

## Überprüfung

In dieser Übung haben Sie Releasepipelinen konfiguriert und anschließend Freigabetore konfiguriert und getestet.
