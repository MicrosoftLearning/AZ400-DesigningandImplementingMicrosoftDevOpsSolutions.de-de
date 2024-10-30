---
lab:
  title: Agiles Planen und Portfoliomanagement mit Azure Boards
  module: 'Module 01: Implement development for enterprise DevOps'
---

# Agiles Planen und Portfoliomanagement mit Azure Boards

## Labanforderungen

- Für dieses Lab ist **Microsoft Edge** oder ein von [Azure DevOps unterstützter Browser](https://docs.microsoft.com/azure/devops/server/compatibility?view=azure-devops#web-portal-supported-browsers) erforderlich.

- **Einrichten einer Azure DevOps-Organisation**: Wenn Sie nicht bereits eine Azure DevOps-Organisation haben, die Sie für dieses Lab verwenden können, müssen Sie diese erstellen, indem Sie die unter [Erstellen einer Organisation oder Projektsammlung](https://docs.microsoft.com/azure/devops/organizations/accounts/create-organization?view=azure-devops) beschriebenen Anweisungen befolgen.

## Übersicht über das Labor

In diesem Lab erfahren Sie mehr über die von Azure Boards bereitgestellten Tools und Prozesse zur Agile-Planung und Portfolioverwaltung, und wie sie Ihnen helfen können, die Arbeit in ihrem gesamten Team schnell zu planen, zu verwalten und zu verfolgen. Sie erkunden den Product Backlog, den Sprint-Backlog und die Task Boards, die den Workflow während einer Iteration nachverfolgen können. Wir sehen uns auch die erweiterten Tools für größere Teams und Organisationen in diesem Release an.

## Ziele

In diesem Lab lernen Sie Folgendes:

- Verwalten von Teams, Bereichen und Iterationen.
- Verwalten von Arbeitselementen
- Verwalten von Sprints und Kapazität.
- Anpassen von Kanban-Boards.
- Definieren von Dashboards.
- Anpassen des Teamprozesses.

## Geschätzte Zeit: 60 Minuten

## Anweisungen

### Übung 0: (überspringen, falls bereits erledigt) Konfigurieren Sie die Lab-Voraussetzungen

In dieser Übung richten Sie die Voraussetzungen für das Lab ein, das aus einem neuen Azure DevOps-Projekt mit einem Repository basierend auf dem [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb) besteht.

#### Aufgabe 1: (überspringen, wenn fertig) Erstellen und Konfigurieren des Teamprojekts

In dieser Aufgabe erstellen Sie ein **eShopOnWeb** Azure DevOps-Projekt, das von mehreren Labs verwendet werden soll.

1. Öffnen Sie auf Ihrem Lab-Computer in einem Browserfenster Ihre Azure DevOps-Organisation. Klicken Sie auf **Neues Projekt**. Geben Sie dem Projekt den Namen **eShopOnWeb**. Definieren Sie **Privat** als Sichtbarkeit.
1. Klicken Sie auf **Erweitert** und geben Sie **Scrum** als **Arbeitselementprozess** an.
 Klicken Sie auf **Erstellen**.

    ![Screenshot des Panels „Neues Projekt erstellen“](images/create-project.png)

### Übung 1: Verwalten eines Agile-Projekts

In dieser Übung verwenden Sie Azure Boards, um eine Reihe gängiger agiler Planungs- und Portfolioverwaltungsaufgaben auszuführen, einschließlich der Verwaltung von Teams, Bereichen, Iterationen, Arbeitselemente, Sprints und Kapazitäten. Sie tun dies durch Anpassen von Kanban-Boards, Definieren von Dashboards und Anpassen von Teamprozessen.

#### Aufgabe 1: Verwalten von Teams, Bereichen und Iterationen.

In dieser Aufgabe erstellen Sie ein neues Team und konfigurieren dessen Bereich und Iterationen.

Jedes neue Projekt ist mit einem Standardteam konfiguriert, das dem Projektnamen entspricht. Sie haben die Möglichkeit, zusätzliche Teams zu erstellen. Jedem Team kann Zugriff auf eine Suite von Agile-Tools und Teamressourcen gewährt werden. Die Möglichkeit, mehrere Teams zu erstellen, bietet Ihnen die Flexibilität, das richtige Gleichgewicht zwischen Autonomie und Zusammenarbeit im gesamten Unternehmen auszuwählen.

1. Starten Sie auf Ihrem Lab-Computer einen Webbrowser und navigieren Sie zum Azure DevOps-Portal unter `https://aex.dev.azure.com`

   > **Hinweis**: Wenn Sie dazu aufgefordert werden, melden Sie sich mit dem Microsoft-Konto an, das mit Ihrem Azure DevOps-Abonnement verknüpft ist.

1. Öffnen Sie das **eShopOnWeb**-Projekt in Ihrer Azure DevOps-Organisation.

    > **Hinweis**: Alternativ können Sie die Projektseite direkt aufrufen, indem Sie zur URL <https://dev.azure.com/YOUR-AZURE-DEVOPS-ORGANIZATION/PROJECT-NAME> navigieren, wobei der Platzhalter YOUR-AZURE-DEVOPS-ORGANIZATION für Ihren Kontonamen und der Platzhalter PROJECT-NAME für den Namen des Projekts steht.

1. Klicken Sie auf das Zahnradsymbol mit der Bezeichnung **Projekteinstellungen** in der unteren linken Ecke der Seite, um die Seite **Projekteinstellungen** zu öffnen.

    ![Screenshot der Einstellungsseite von Azure DevOps.](images/m1/project_settings_v1.png)

1. Wählen Sie im Abschnitt **Allgemein** die Registerkarte **Teams** aus. Es gibt bereits ein Standardteam in diesem Projekt, das **eShopOnWeb**-Team, aber Sie erstellen für dieses Lab ein neues Team. Klicken Sie auf **Neues Team**.

    ![Screenshot des Fensters mit den Projekteinstellungen von Teams.](images/m1/new_team_v1.png)

1. Geben Sie im Bereich **Neues Team erstellen** im Textfeld **Teamname** **`EShop-Web`** ein, übernehmen Sie die Standardwerte für die anderen Einstellungen und klicken Sie auf **Erstellen**.

    ![Screenshot der Seite „Projekteinstellungen“ des Teams.](images/m1/eshopweb-team_v1.png)

1. Wählen Sie in der Liste der **Teams** das neu erstellte Team aus, um dessen Details anzuzeigen.

    > **Hinweis**: Standardmäßig verfügt das neue Team nur über Sie als Mitglied. Sie können diese Ansicht verwenden, um solche Funktionen wie Teammitgliedschaft, Benachrichtigungen und Dashboards zu verwalten.

1. Klicken Sie oben auf der **EShop-Web**-Seite auf den Link **Iterationen und Bereichspfade**, um mit der Definition des Zeitplans und des Umfangs des Teams zu beginnen.

    ![Screenshot der Option „Iterationen und Bereichspfade“](images/m1/EShop-WEB-iterationsareas_v1.png)

1. Wählen Sie oben im Bereich **Boards** die Registerkarte **Iterationen** aus, und klicken Sie dann auf **+Iteration(en) auswählen**.

    ![Screenshot der Konfigurationsseite für Iterationen.](images/m1/EShop-WEB-select_iteration_v1.png)

1. Wählen Sie **eShopOnWeb\Sprint 1** aus und klicken Sie auf **Speichern und schließen**. Beachten Sie, dass dieser erste Sprint in der Liste der Iterationen angezeigt wird, aber die Datumsangaben noch nicht festgelegt sind.
1. Wählen Sie **Sprint 1** aus, und klicken Sie auf die **Auslassungspunkte (...)**. Wählen Sie im Kontextmenü  **Bearbeiten** aus.

     ![Screenshot der Einstellungen auf der Registerkarte „Iterationen bearbeiten“.](images/m1/EShop-WEB-edit_iteration_v1.png)

    > **Hinweis**: Geben Sie das „Startdatum“ als ersten Arbeitstag der letzten Woche an, und rechnen Sie 3 volle Arbeitswochen für jeden Sprint. Wenn der 6  März beispielsweise der erste Arbeitstag des Sprints ist, geht dieser bis zum 24. März. Sprint 2 beginnt am 27  März, das ist 3 Wochen ab dem 6. März.

1. Wiederholen Sie den vorherigen Schritt, um **Sprint 2** und **Sprint 3** hinzuzufügen. Man könnte also sagen, dass wir derzeit in der 2. Woche des ersten Sprints sind.

    ![Screenshot der Sprint-Konfiguration unter der Registerkarte „Iterationen“.](images/m1/EShop-WEB-3sprints_v1.png)

1. Während Sie sich noch im Bereich **Projekteinstellungen/Boards/Teamkonfiguration** befinden, wählen Sie oben im Bereich die Registerkarte **Bereiche** aus. Dort werden Sie einen automatisch generierten Bereich mit dem Namen finden, der dem Namen des Teams entspricht.

    ![Screenshot der Einstellungsseite für Bereiche.](images/m1/EShop-WEB-areas_v1.png)

1. Klicken Sie neben dem Eintrag für den **Standardbereich** auf das Auslassungszeichen (...) und wählen Sie in der Dropdownliste **Unterbereiche einschließen** aus.

    ![Screenshot der Option „Unterbereiche einbeziehen“ unter der Registerkarte „Bereiche“.](images/m1/EShop-WEB-sub_areas_v1.png)

    > **Hinweis**: In der Standardeinstellung aller Teams werden Unterbereichspfade ausgeschlossen. Wir werden sie so ändern, dass sie Unterbereiche umfasst, damit das Team Einblick in alle Arbeitselemente aller Teams erhält. Optional kann sich das Managementteam auch dafür entscheiden, keine Unterbereiche einzuschließen, wodurch Arbeitselemente automatisch aus der Ansicht entfernt werden, sobald sie einem der Teams zugewiesen sind.

#### Aufgabe 2: Verwalten von Arbeitselementen

In dieser Aufgabe durchlaufen Sie allgemeine Verwaltungsaufgaben für Arbeitselemente.

Arbeitsaufgaben spielen in Azure DevOps eine herausragende Rolle. Ob es sich nun um die Beschreibung der zu erledigenden Arbeit, um Hindernisse für die Freigabe, um Testdefinitionen oder um andere wichtige Punkte handelt: Arbeitselemente sind das Herzstück moderner Projekte. In dieser Aufgabe konzentrieren Sie sich auf die Verwendung verschiedener Arbeitsaufgaben, um den Plans zum Erweitern der eShopOnWeb-Website um einem Produktschulungsabschnitt zu erweitern. Es kann zwar etwas schwierig erscheinen, einen so umfangreichen Teil des Unternehmensangebots zu entwickeln, aber mit Azure DevOps und dem Scrum-Prozess ist dies sehr gut zu bewältigen.

> **Hinweis**: Diese Aufgabe soll eine Reihe von Möglichkeiten aufzeigen, wie Sie verschiedene Arten von Arbeitselementen erstellen können, und die Vielfalt der auf der Plattform verfügbaren Features demonstrieren. Daher sollten diese Schritte nicht als verbindliche Anleitung für das Projektmanagement angesehen werden. Die Features sollen so flexibel sein, dass sie sich an Ihre Prozessanforderungen anpassen lassen, also experimentieren Sie ruhig damit.

1. Klicken Sie auf den Projektnamen in der oberen linken Ecke des Azure DevOps-Portals, um zur Projekt-Startseite zurückzukehren.

1. Wählen Sie im vertikalen Navigationsbereich des Azure DevOps-Portals das Symbol **Boards** und dann **Arbeitselemente** aus.

    > **Hinweis**:Es gibt viele Möglichkeiten, Arbeitselemente in Azure DevOps zu erstellen, und wir werden einige davon untersuchen. Manchmal ist es so einfach wie das direkte Abrufen eines Elements von einem Dashboard.

1. Klicken Sie im Fenster **Arbeitselemente** auf **+ Neues Arbeitselement > Epic**.

    ![Screenshot der Erstellung des Arbeitselements.](images/m1/EShop-WEB-create_epic_v1.png)

1. Geben Sie in das Textfeld **Titel eingeben** Folgendes ein: **`Product training`**.
1. Wählen Sie in der oberen linken Ecke den Eintrag **Keine Person ausgewählt** aus und wählen Sie in der Dropdown-Liste Ihr Benutzerkonto aus, um sich selbst das neue Arbeitselement zuzuweisen. Wenn Ihr Name nicht beginnt, beginnen Sie mit der Eingabe Ihres Namens, und klicken Sie auf **Suchen**.
1. Wählen Sie neben dem Eintrag **Bereich** den Eintrag **eShopOnWeb** und dann in der Dropdownliste **EShop-WEB** aus. Dadurch wird der **Bereich** auf **eShopOnWeb\EShop-WEB** festgelegt.
1. Wählen Sie neben dem Eintrag **Iteration** den Eintrag **eShopOnWeb** und in der Dropdownliste **Sprint 2** aus. Dadurch wird die **Iteration** auf **eShopOnWeb\Sprint 2** festgelegt.
1. Klicken Sie zum Abschließen der Änderungen auf **Speichern**. **Schließen Sie es nicht**.

    ![Screenshot der Details des Arbeitselements, einschließlich Titel, Bereich und Iterationsoptionen.](images/m1/EShop-WEB-epic_details_v1.png)

    > **Hinweis**: Normalerweise würden Sie so viele Informationen wie möglich ausfüllen, aber für die Zwecke dieses Labs ist dies ausreichend.

    > **Hinweis**: Das Formular Arbeitselemente enthält alle relevanten Einstellungen für Arbeitselemente. Dazu gehören Details darüber, wem sie zugewiesen ist, ihr Status in Bezug auf viele Parameter und alle zugehörigen Informationen und der Verarbeitungsverlauf seit ihrer Erstellung. Einer der wichtigsten Bereiche dabei sind **zugehörige Aufgaben**. Wir werden eine der Möglichkeiten erkunden, um diesem Epic ein Feature hinzuzufügen.

1. Wählen Sie im Abschnitt **Verwandte Aufgaben** unten rechts den Eintrag **Link hinzufügen** aus, und wählen Sie in der Dropdownliste **Neues Element** aus.
1. Wählen Sie im Bereich **Link hinzufügen** in der Dropdownliste **Linktyp** die Option **Untergeordnetes Element** aus. Wählen Sie anschließend in der Dropdown-Liste **Arbeitselementtyp** die Option **Funktion** aus und geben Sie in das Textfeld **Titel** **`Training dashboard`** ein.

    ![Screenshot der Erstellung des Links zum Arbeitselement.](images/m1/EShop-WEB-create_child_feature.png)

1. Klicken Sie auf **Link hinzufügen**, um das untergeordnete Element zu speichern.

    ![Screenshot des Arbeitsbereichs für das Arbeitselement.](images/m1/EShop-WEB-epic_with_linked_item_v1.png)

    > **Hinweis:** Beachten Sie im Bereich **Schulungsdashboard**, dass die Zuordnung, der **Bereich** und die **Iteration** bereits auf die gleichen Werte wie das Epic festgelegt sind, auf dem das Feature basiert. Darüber hinaus wird das Feature automatisch mit dem übergeordneten Element verknüpft, aus dem es erstellt wurde.

1. Klicken Sie im Bereich **Trainingsdashboard** (Neues Feature) auf **Speichern und Schließen**.

1. Wählen Sie im vertikalen Navigationsbereich des Azure DevOps-Portals in der Liste der **Boards**-Elemente die Option **Boards** aus.
1. Wählen Sie im Bereich **Boards** den Eintrag **EShop-WEB Boards** aus. Dadurch wird das Board für dieses bestimmte Team geöffnet.

    ![Screenshot der EShop-WEB-Board-Auswahl.](images/m1/EShop-WEB-_boards_v1.png)

1. Wählen Sie im Bereich **Boards** in der oberen rechten Ecke den Eintrag **Backlog Items** aus, und wählen Sie in der Dropdownliste **Features** aus.

    > **Hinweis**: Dies erleichtert das Hinzufügen von Aufgaben und anderen Arbeitselementen zu den Features.

1. Zeigen Sie mit dem Mauszeiger auf das Rechteck, das das Feature **Schulungsdashboard** darstellt. Dadurch wird das Auslassungszeichen (...) in der oberen rechten Ecke angezeigt.
1. Klicken Sie auf das Symbol mit den Auslassungspunkten (...) und wählen Sie in der Dropdownliste **Product Backlog Item hinzufügen** aus.

    ![Screenshot der Option „Produktrückstand hinzufügen“ in der Dashboard-Funktion „Schulung“.](images/m1/EShop-WEB-add_pb_v1.png)

1. Geben Sie im Textfeld des neuen Produktrückstandspostens **`As a customer, I want to view new tutorials`** ein und drücken Sie die Taste **Eingabe**, um den Eintrag zu speichern.

    > **Hinweis**: Dadurch wird ein neues Product Backlog Item (PBI) erstellt, das ein untergeordnetes Element des Features ist und dessen Bereich und Iteration teilt.

1. Wiederholen Sie den vorherigen Schritt, um zwei weitere PBIs hinzuzufügen, die es den Kundinnen und Kunden ermöglichen, ihre kürzlich angesehenen Tutorials anzuzeigen und neue Tutorials mit den Namen **`As a customer, I want to see tutorials I recently viewed`** und **`As a customer, I want to request new tutorials`** anzufordern.

    ![Screenshot des hinzugefügten Produktrückstands.](images/m1/EShop-WEB-pbis_v1.png)

1. Wählen Sie im Bereich **Boards** in der oberen rechten Ecke den Eintrag **Features** aus, und wählen Sie in der Dropdownliste **Backlog Items**aus.

     ![Screenshot der Ansicht der Rückstands-Einträge.](images/m1/EShop-WEB-backlog_v1.png)

    > **Hinweis**: Backlog Items haben einen Status, der definiert, wo sie relativ zur Fertigstellung stehen. Obwohl Sie die Arbeitsaufgabe mithilfe des Formulars öffnen und bearbeiten können, ist es einfacher, Karten auf das Board zu ziehen.

1. Ziehen Sie auf der Registerkarte **Board** im Bereich **EShop-WEB** das erste Arbeitselement namens **Als Kundschaft möchte ich neue Tutorials ansehen** von Status **Neu** auf **Genehmigt**.

    ![Screenshot des Boards mit Arbeitselementen.](images/m1/EShop-WEB-new2ap_v1.png)

    > **Hinweis**: Sie können die Karten für Arbeitsaufgabe auch vergrößern, um bequem alle bearbeitbaren Details zu erreichen.

1. Zeigen Sie mit dem Mauszeiger auf das Rechteck der Arbeitsaufgabe, die Sie in **Genehmigt** verschoben haben. Dadurch wird das nach unten zeigende Caretsymbol angezeigt.
1. Klicken Sie auf das nach unten weisende Caretzeichen, um die Arbeitselementkarte zu erweitern, ersetzen Sie den Eintrag **Nicht zugewiesen** durch Ihren Namen, und wählen Sie dann Ihr Konto aus, um sich das verschobene PBI selbst zuzuweisen.
1. Ziehen Sie auf der Registerkarte **Board** im Bereich **EShop-WEB** das zweite Arbeitselement namens **Als Kundschaft möchte ich meine kürzlich angesehenenTutorials sehen** von Status **Neu** auf **Ausgeliefert**.
1. Ziehen Sie auf der Registerkarte **Board** im Bereich **EShop-WEB** das dritte Arbeitselement namens **Als Kundschaft möchte ich neue Tutorials anfordern** von Status **Neu** auf **Fertig**.

    ![Screenshot des Boards mit Arbeitselementen, die aus vorherigen Schritten in die angegebenen Spalten verschoben wurden.](images/m1/EShop-WEB-board_pbis_v1.png)

    > **Hinweis**: Das Taskboard ist ein Blick in das Backlog. Sie können auch die tabellarische Ansicht verwenden.

1. Klicken Sie auf der Registerkarte **Board** oben im Bereich **EShop-WEB** auf **Als Backlog anzeigen**, um das tabellarische Formular anzuzeigen.

    ![Screenshot des EShop-WEB-Rückstands.](images/m1/EShop-WEB-view_backlog_v1.png)

    > **Hinweis**: Sie können das Pluszeichen direkt unter dem Registerkartennamen **Backlog** im Bereich **EShop-WEB** verwenden, um Aufgaben unter diesen Arbeitsaufgaben verschachtelt anzuzeigen.

    > **Hinweis**: Sie können das zweite Pluszeichen direkt links vom ersten Backlogelement verwenden, um eine neue Aufgabe hinzuzufügen.

1. Klicken Sie auf der Registerkarte **Backlog** im Bereich **EShop-WEB** in der oberen linken Ecke auf das Pluszeichen neben dem ersten Arbeitselement. Dadurch wird der Bereich **NEUE AUFGABE** angezeigt.

    ![Screenshot der Option „Aufgabe erstellen“ in der Backlog-Liste.](images/m1/new_task_v1.png)

1. Geben Sie oben im Feld **NEUE AUFGABE** im Textfeld **Titel eingeben** **`Add page for most recent tutorials`** ein.
1. Geben Sie im Bereich **NEUE AUFGABE** im Textfeld **Verbleibende Arbeit****5** ein.
1. Wählen Sie im Bereich **NEUE AUFGABE** in der Dropdownliste **Aktivität** die Option **Entwicklung** aus.
1. Klicken Sie im Bereich **NEUE AUFGABE** auf **Speichern und Schließen**.

    ![Screenshot der Erstellung eines neuen Arbeitselements.](images/m1/EShop-WEB-save_task_v1.png)

1. Wiederholen Sie die letzten fünf Schritte, um eine weitere Aufgabe mit dem Namen **`Optimize data query for most recent tutorials`** hinzuzufügen. Legen Sie  **Verbleibende Arbeit** auf **3** fest und **Aktivität** auf **Design** fest. Klicken Sie nach Abschluss auf **Speichern und Schließen**.

#### Aufgabe 3: Verwalten von Sprints und Kapazität.

In dieser Aufgabe durchlaufen Sie allgemeine Sprint- und Kapazitätsverwaltungsaufgaben.

Teams erstellen das Sprint-Backlog während der Besprechung zur Sprintplanung, die in der Regel am ersten Tag des Sprints stattfindet. Jeder Sprint entspricht einem Zeitintervall, was Ihr Team bei der Arbeit mit Agile-Prozessen und -Tools unterstützt. Während der Planungsbesprechung arbeiten Produktverantwortliche mit Ihrem Team zusammen, um die Storys oder Backlog Items zu identifizieren, die im Sprint abgeschlossen werden sollen.

Planungsbesprechungen bestehen in der Regel aus zwei Teilen. Im ersten Teil identifizieren das Team und der*die Produktbesitzer*in die Backlog Items, für die das Team auf der Grundlage der Erfahrungen mit früheren Sprints davon ausgeht, dass sie im Sprint abgeschlossen werden können. Diese Elemente werden dem Sprintbacklog hinzugefügt. Im zweiten Teil legt Ihr Team fest, wie die einzelnen Elemente entwickelt und getestet werden. Anschließend definieren und schätzen sie die Aufgaben, die zum Abschließen der einzelnen Elemente erforderlich sind. Schließlich verpflichtet sich das Team, einige oder alle Elemente basierend auf diesen Schätzungen zu implementieren.

Nach Fertigstellung des Sprintplans sollten Ihre Sprint-Backlogs alle Informationen enthalten, die Ihr Team benötigt, um die Arbeit innerhalb der vorgesehenen Zeit ohne Zeitdruck am Ende erfolgreich abzuschließen. Bevor Sie den Sprint planen, sollten Sie den Backlog erstellt, priorisiert und geschätzt und die Sprints definiert haben.

1. Wählen Sie im vertikalen Navigationsbereich des Azure DevOps-Portals das ** Boards**-Symbol aus, und wählen Sie in der Liste der **Boards**-Elemente **Sprints** aus.
1. Wählen Sie auf der Registerkarte **Taskboard** der Ansicht **Sprints** auf der Symbolleiste auf der rechten Seite das Symbol **Ansichtsoptionen** (direkt links neben dem Trichtersymbol) aus, und wählen Sie in der Dropdownliste **Optionen anzeigen** den Eintrag **Arbeitsdetails** aus. Wählen Sie **Sprint 2** als Filter aus.

    ![Screenshot des EShop-WEB-Boards mit den Arbeitsdetails.](images/m1/EShop-WEB-work_details_v1.png)

    > **Hinweis**: Der aktuelle Sprint hat einen ziemlich begrenzten Umfang. Es gibt zwei Aufgaben in der **To-Do**-Phase. An diesem Punkt wurde keine Aufgabe zugewiesen. Beide zeigen einen numerischen Wert rechts neben dem Eintrag **Nicht zugewiesen** an, der die neu berechnete Arbeitsschätzung darstellt.

1. Beachten Sie in der Spalte **ToDo** das Aufgabenelement **Seite für die neuesten Tutorials hinzufügen**, klicken Sie auf den Eintrag **Nicht zugewiesen**, und wählen Sie in der Liste der Benutzerkonten Ihr Konto aus, um sich die Aufgabe selbst zuzuweisen.

1. Wählen Sie die Registerkarte **Kapazität** der Ansicht **Sprint** aus.

    ![Screenshot des Boards mit Arbeitselementen und Kapazitätsstunden.](images/m1/EShop-WEB-capacity_v1.png)

    > **Hinweis**: Mit dieser Ansicht können Sie definieren, welche Aktivitäten und Kapazitätsstufe Benutzer*innen übernehmen können.

1. Stellen Sie in der Ansicht **Sprints** auf der Registerkarte **Kapazität** für Ihr Benutzerkonto das Feld **Aktivität** auf **Entwicklung** und geben Sie im Textfeld **Kapazität pro Tag** **1** ein. Klicken Sie anschließend auf **Speichern**.

    > **Hinweis**: Dies stellt 1 Stunde Entwicklungsarbeit pro Tag dar. Beachten Sie, dass Sie zusätzliche Aktivitäten pro Benutzer*in hinzufügen können, wenn diese mehr tun als nur Entwicklung.

    ![Screenshot der Kapazitätskonfiguration.](images/m1/EShop-WEB-capacity-setdevelopment_v1.png)

    > **Hinweis:** Nehmen wir an, Sie werden auch Urlaub nehmen. Dieser sollte der Kapazitätsansicht ebenfalls hinzugefügt werden.

1. Klicken Sie auf der Registerkarte **Kapazität** der Ansicht **Sprints** direkt neben dem Eintrag, der Ihr Benutzerkonto darstellt, in der Spalte **Freie Tage** auf den Eintrag **0 Tage**. Dadurch wird ein Bereich angezeigt, in dem Sie Ihre Tage einstellen können.
1. Verwenden Sie im angezeigten Bereich die Kalenderansicht, um Ihren Urlaub auf fünf Arbeitstage während des aktuellen Sprints (innerhalb der nächsten drei Wochen) festzulegen, und klicken Sie nach Abschluss auf **OK**.

    ![Screenshot der Konfiguration für den freien Tag.](images/m1/EShop-WEB-days_off_v1.png)

1. Zurück auf der Registerkarte **Kapazität** der Ansicht **Sprints** klicken Sie auf **Speichern**.
1. Wählen Sie die Registerkarte **Taskboard** der Ansicht **Sprints** aus.

    ![Screenshot des Abschnitts mit den Arbeitsdetails.](images/m1/EShop-WEB-work_details_window_v1.png)

    > **Hinweis**: Beachten Sie, dass der Bereich **Arbeitsdetails** aktualisiert wurde, um Ihren verfügbaren Zeitrahmen widerzuspiegeln. Die tatsächliche Zahl, die im Bereich **Arbeitsdetails** angezeigt wird, kann variieren, ihre Gesamt-Sprintkapazität entspricht jedoch der Anzahl der Arbeitstage, die bis zum Ende des Sprints verbleiben, da Sie 1 Stunde pro Tag zugewiesen haben. Notieren Sie sich diesen Wert, da Sie ihn in den kommenden Schritten verwenden werden.

    > **Hinweis**: Ein praktisches Feature der Boards besteht darin, dass Sie wichtige Daten ganz einfach inline aktualisieren können. Es empfiehlt sich, die Schätzung **Verbleibende Arbeit** regelmäßig zu aktualisieren, um die für jede Aufgabe erwartete Zeit widerzuspiegeln. Angenommen, Sie haben die Arbeit für die Seite **Seite für aktuellste Tutorials hinzufügen** überprüft und festgestellt, dass sie tatsächlich länger dauert als ursprünglich erwartet.

1. Legen Sie auf der Registerkarte **Taskboard** der Ansicht **Sprints** im rechteckigen Feld, das  **Seite für aktuellste Tutorials hinzufügen** darstellt, die geschätzte Anzahl von Stunden auf **14** fest, sodass Sie ihre Gesamtkapazität für diesen Sprint, die Sie im vorherigen Schritt identifiziert haben, anpassen.

    ![Screenshot des Boards und der zugewiesenen Zeitkapazität des Teams.](images/m1/EShop-WEB-over_capacity_v1.png)

    > **Hinweis**: Dadurch wird die **Entwicklung** und Ihre persönlichen Kapazitäten automatisch auf ihr Maximum erweitert. Da sie groß genug sind, um die zugewiesenen Aufgaben abzudecken, bleiben sie grün. Die Gesamtkapazität des **Teams** wird jedoch aufgrund der zusätzlichen 3 Stunden überschritten, die für die Aufgabe **Datenabfrage für die aktuellsten Tutorials optimieren** erforderlich sind.

    > **Hinweis**: Eine Möglichkeit, dieses Kapazitätsproblem zu beheben, besteht darin, die Aufgabe in eine zukünftige Iteration zu verschieben. Dafür gibt es mehrere Möglichkeiten. Sie können die Aufgabe beispielsweise hier öffnen und im Bereich mit den Aufgabendetails bearbeiten. Ein weiterer Ansatz wäre die Verwendung der **Backlog**-Ansicht, die eine Inlinemenüoption zum Verschieben bereitstellt. Zu diesem Zeitpunkt sollten Sie die Aufgabe jedoch noch nicht verschieben.

1. Wählen Sie auf der Registerkarte **Taskboard** in der Ansicht **Sprints** in der Symbolleiste auf der rechten Seite das Symbol **Ansichtsoptionen** (direkt links neben dem Trichter-Symbol) und wählen Sie in der Dropdown-Liste **Ansichtsoptionen** den Eintrag **Zugewiesen an** aus.

    > **Hinweis**: Dadurch wird Ihre Ansicht so angepasst, dass Sie den Fortschritt von Aufgaben nach Person statt nach Backlog-Element überprüfen können.

    > **Hinweis**: Es gibt auch eine Vielzahl von Anpassungen.

1. Klicken Sie auf das Zahnradsymbol **Teameinstellungen konfigurieren** (direkt rechts neben dem Trichtersymbol).
1. Wählen Sie im Bereich **Einstellungen** die Registerkarte **Stile** aus, klicken Sie auf **+ Stilregel hinzufügen**, geben Sie unter der Bezeichnung **Regelname** in das Textfeld **Name** den Text **`Development`** ein und wählen Sie in der Dropdown-Liste **Farbe** das grüne Rechteck aus.

    > **Hinweis**: Dadurch werden alle Karten grün gefärbt, wenn sie die im Abschnitt **Regelkriterien** direkt darunter festgelegten Regelkriterien erfüllen.

1. Wählen Sie im Abschnitt **Regelkriterien** in der Dropdown-Liste **Feld** den Eintrag **Aktivität**, in der Dropdown-Liste **Operator** den Eintrag **=** und in der Dropdown-Liste **Wert** den Eintrag **Entwicklung**.

    ![Screenshot der Einstellungen für den Stil des Boards.](images/m1/EShop-WEB-styles_v2.jpg)

1. Klicken Sie auf **Speichern**, um die Einstellungen zu speichern und zu schließen.

    ![Screenshot der Aufgabenstile im Board des Teams.](images/m1/EShop-WEB-sprint-green_v1.png)

    > **Hinweis**: Dadurch werden alle Karten, die Aktivitäten vom Typ **Entwicklung** zugeordnet sind, auf grün eingestellt.

1. Gehen Sie zurück zum Bereich **Einstellungen**, wählen Sie die Registerkarte **Allgemein** aus und konfigurieren Sie im Abschnitt **Rückstände** die Navigationsebenen.

    > **Hinweis**: Epics sind standardmäßig nicht enthalten, aber Sie können das ändern.

1. Wählen Sie im Bereich **Einstellungen** die Registerkarte **Allgemein** und geben Sie unter dem Abschnitt **Arbeitstage** die Arbeitstage an, an denen das Team arbeitet.

    > **Hinweis**: Dies gilt für Kapazitäts- und Burndown-Berechnungen.

1. In den **Einstellungen**, wählen Sie die Registerkarte **Allgemein**, und unter dem Abschnitt **Bearbeitung von Fehlern** können Sie festlegen, wie Sie Fehler in Backlogs und Boards verwalten.

    > **Hinweis**: Einträge auf dieser Registerkarte ermöglichen es Ihnen, anzugeben, wie Fehler auf dem Board dargestellt werden.

1. Klicken Sie im Bereich **Einstellungen** auf **Speichern**, um die Stilregel zu speichern und zu schließen.

    > **Hinweis**: Die mit  **Entwicklung** verbundene Aufgabe ist jetzt grün und sehr einfach zu identifizieren.

#### Aufgabe 4: Anpassen von Kanban-Boards.

In dieser Aufgabe durchlaufen Sie den Prozess der Anpassung von Kanban-Boards.

Um die Fähigkeit eines Teams zu verbessern, dauerhaft hochwertige Software bereitzustellen, setzt Kanban vor allem auf zwei Verfahren. Die erste Visualisierung des Workflows erfordert, dass Sie die Workflowphasen Ihres Teams zuordnen und ein Kanban-Board so konfigurieren, dass es übereinstimmt. Das zweite Verfahren, die Begrenzung der derzeit durchgeführten Arbeiten, erfordert die Festlegung von WIP-Grenzwerten (Work in Progress). Anschließend können Sie den Fortschritt auf Ihrem Kanban-Board nachverfolgen und die wichtigsten Metriken überwachen, um die Vorlauf- oder Zykluszeiten zu verringern. Ihr Kanban-Board verwandelt Ihr Backlog in eine interaktive „Hinweistafel“, die den Arbeitsablauf veranschaulicht. Während die Arbeit von der Idee bis zur Fertigstellung voranschreitet, aktualisieren Sie die Elemente auf dem Board. Jede Spalte stellt eine Arbeitsphase dar, und jede Karte repräsentiert eine User Story (blaue Karten) oder einen Fehler (rote Karten) in dieser Arbeitsphase. Jedes Team entwickelt jedoch im Laufe der Zeit seinen eigenen Prozess. Daher ist die Möglichkeit, das Kanban-Board an die Arbeitsweise Ihres Teams anzupassen, entscheidend für die erfolgreiche Bereitstellung

1. Wählen Sie im vertikalen Navigationsbereich des Azure DevOps-Portals in der Liste der **Boards**-Elemente den Eintrag **Boards** aus.
1. Klicken Sie im Bereich **Boards** auf das Zahnradsymbol **Boardeinstellungen konfigurieren** (direkt rechts neben dem Trichtersymbol).

    > **Hinweis**: Das Team legt den Schwerpunkt auf die Arbeit mit Daten. Daher wird allen Aufgaben, die mit dem Zugriff auf oder der Speicherung von Daten verbunden sind, besondere Aufmerksamkeit gewidmet.

1. Wählen Sie im Bereich **Einstellungen** die Registerkarte **Tag-Farben** aus, klicken Sie auf **+ Tag-Farbe hinzufügen**, geben Sie im Textfeld **Tag** **`data`** ein und wählen Sie das gelbe Rechteck aus.

    ![Screenshot der Tag-Einstellungen.](images/m1/EShop-WEB-tag_color_v1.png)

    > **Hinweis**: Wenn ein Backlog-Element oder ein Fehler mit **Daten** markiert ist, wird dieses Tag hervorgehoben.

1. Wählen Sie die Registerkarte **Anmerkungen** aus.

    > **Hinweis**: Sie können angeben, welche **Anmerkungen** in Karten enthalten sein sollen, damit sie einfacher zu lesen und zu navigieren sind. Wenn eine Anmerkung aktiviert ist, können auf die untergeordneten Arbeitselemente dieses Typs problemlos zugegriffen werden, indem sie auf die Visualisierung in den einzelnen Karte klicken.

1. Wählen Sie die Registerkarte **Tests** aus.

    > **Hinweis**: Auf der Registerkarte **Tests** können Sie konfigurieren, wie Tests auf den Karten angezeigt werden und sich verhalten sollen.

1. Klicken Sie auf **Speichern**, um die Einstellungen zu speichern und zu schließen.
1. Öffnen Sie auf der Registerkarte **Board** des Bereichs **EShop-WEB** das Arbeitselement, welches das Backlog Item **Als Kunde möchte ich neue Tutorials anzeigen** darstellt.
1. Klicken Sie in der detaillierten Elementansicht oben im Bereich rechts neben dem Eintrag **0 Kommentare** auf **Tag hinzufügen**.
1. Geben Sie in das daraufhin angezeigte Textfeld **`data`** ein und drücken Sie die Taste **Enter**.
1. Wiederholen Sie den vorherigen Schritt, um das **`ux`** Tag hinzuzufügen.
1. Speichern Sie diese Bearbeitungen, indem Sie auf **Speichern und Schließen** klicken.

    ![Screenshot der beiden neuen Tags, die auf der Karte sichtbar sind.](images/m1/EShop-WEB-tags_v1.png)

    > **Hinweis**: Die beiden Tags sind jetzt auf der Karte sichtbar, wobei das Tag **Daten** entsprechend der Konfiguration gelb hervorgehoben ist.

1. Klicken Sie im Bereich **Boards** auf das Zahnradsymbol **Boardeinstellungen konfigurieren** (direkt rechts neben dem Trichtersymbol).
1. Wählen Sie im Bereich **Einstellungen** die Registerkarte **Spalten** aus.

    > **Hinweis**: In diesem Abschnitt können Sie dem Workflow neue Stufen hinzufügen.

1. Klicken Sie auf **+ Spalte hinzufügen**, geben Sie unter der Beschriftung **Spaltenname** in das Textfeld **Name** **`QA Approved`** und in das Textfeld **WIP-Grenze** **1** ein.

    > **Hinweis**: Der Grenzwert 1 für In Bearbeitung gibt an, dass sich jeweils nur ein Arbeitselement in diesem Status befinden sollte. Sie würden dies normalerweise höher festlegen, aber es gibt nur zwei Arbeitselemente, um das Feature zu veranschaulichen.

    ![Screenshot der WIP-Einstellungen.](images/m1/EShop-WEB-qa_column_v1.png)

1. Beachten Sie die Auslassungspunkte neben der von Ihnen erstellten Spalte **QA genehmigt**. Wählen Sie zweimal **Nach rechts verschieben** aus, sodass die Spalte „QA genehmigt“ zwischen **Committet** und **Fertig** positioniert wird.
1. Klicken Sie auf **Speichern**, um die Einstellungen zu speichern und zu schließen.

1. Im **Boards-Portal** ist jetzt die Spalte **QA genehmigt** in der Kanban-Boardansicht sichtbar.
1. Ziehen Sie das Arbeitselement **Als Kundschaft möchte ich meine kürzlich angesehenenTutorials sehen** vom Status **Geliefert** in den Status **QA Genehmigt**.
1. Ziehen Sie das Arbeitselement **Als Kundschaft möchte ich neue Tutorials ansehen** aus dem Status **Genehmigt** in den Status **QA Genehmigt**.

    ![Screenshot des Boards, das seine WIP-Grenze überschreitet.](images/m1/EShop-WEB-wip_limit_v1.png)

    > **Hinweis**: Der Status überschreitet jetzt den **WIP**-Grenzwert und wird als Warnung  rot gefärbt.

1. Verschieben Sie das Backlog Item **Als Kundschaft möchte ich meine kürzlich angesehenenTutorials sehen** zurück in **Geliefert**.
1. Klicken Sie im Bereich **Boards** auf das Zahnradsymbol **Boardeinstellungen konfigurieren** (direkt rechts neben dem Trichtersymbol).
1. Kehren Sie im Bereich **Einstellungen** zur Registerkarte **Spalten** zurück, und wählen Sie die Registerkarte **QA Genehmigt** aus.

    > **Hinweis**:Es gibt häufig eine Verzögerung zwischen dem Verschieben der Arbeit in eine Spalte und dem tatsächlichen Arbeitsbeginn. Um dieser Verzögerung zu begegnen und den tatsächlichen Status der laufenden Arbeit anzuzeigen, können Sie geteilte Spalten aktivieren. Nach der Teilung enthält jede Spalte zwei Unterspalten: **In Bearbeitung** und **Fertig**. Mit geteilten Spalten kann Ihr Team ein Pullmodell implementieren. Ohne geteilte Spalten pushen Teams ihre Arbeit, um zu signalisieren, dass sie ihre Arbeitsphase abgeschlossen haben. Das Pushen in die nächste Phase bedeutet jedoch nicht zwangsläufig, dass ein Teammitglied sofort mit der Arbeit an diesem Element beginnt.

1. Aktivieren Sie auf der Registerkarte **QA Genehmigt** das Kontrollkästchen **Spalte teilen in In Bearbeitung und Fertig**, um zwei separate Spalten zu erstellen.

    > **Hinweis**: Wenn Ihr Team den Status der Arbeit aktualisiert, während sie von einer Phase zur nächsten fortschreitet, hilft es ihnen, sich darüber zu einigen, was **Fertig** bedeutet. Indem Sie die Kriterien für die **Definition von Fertig** für jede Kanban-Spalte angeben, können Sie die wesentlichen Aufgaben teilen, die abgeschlossen werden müssen, bevor Sie ein Element in eine nachgelagerte Phase verschieben.

1. Geben Sie auf der Registerkarte **QA Approved** unten im Feld **Definition of done** den Text **`Passes \*\*all\*\* tests`** ein.
1. Klicken Sie auf **Speichern**, um die Einstellungen zu speichern und zu schließen.

    ![Screenshot der geteilten Spalte für Einstellungen und Definition der fertigen Konfiguration.](images/m1/dd_v1.png)

    > **Hinweis**: Der Status **QA Approved** enthält jetzt die Spalten **In Bearbeitung** und **Fertig**. Sie können auch auf das Informationssymbol (mit Buchstaben **i** in einem Kreis) neben der Spaltenüberschrift klicken, um die **Definition von Fertig** zu lesen. Möglicherweise müssen Sie den Browser aktualisieren, um Änderungen anzuzeigen.

    ![Screenshot der geteilten Spalten für die Spalte „QA Approved“ (Qualitätssicherung genehmigt).](images/m1/EShop-WEB-qa_2columns_v1.png)

1. Klicken Sie im Bereich **Boards** auf das Zahnradsymbol **Boardeinstellungen konfigurieren** (direkt rechts neben dem Trichtersymbol).

    > **Hinweis**: Ihr Kanban-Board unterstützt Sie mit dem Verschieben der Arbeit von Neu in Richtung Fertig beim Visualisieren des Workflows. Wenn Sie **Verantowrtlichkeitsbereiche** hinzufügen, können Sie auch den Status von Arbeiten visualisieren, die verschiedene Servicelevelklassen unterstützen. Sie können eine Swimlane erstellen, die jede andere Dimension darstellt, die Ihre Nachverfolgungsanforderungen unterstützt.

1. Wählen Sie im Bereich **Einstellungen** die Registerkarte **Verantwortlichkeitsbereiche** aus.
1. Klicken Sie auf der Registerkarte **Swimlanes** auf **+ Swimlane hinzufügen**, direkt unter der Beschriftung **Swimlane-Name**. Geben Sie im Textfeld **Name** **`Expedite`** ein.
1. Wählen Sie in der Dropdown-Liste **Farbe** das **grüne** Rechteck aus.
1. Klicken Sie auf **Speichern**, um die Einstellungen zu speichern und zu schließen.

    ![Screenshot der Swimlane-Funktion zur Beschleunigung der Erstellung.](images/m1/EShop-WEB-swimlane_v1.png)

1. Zurück auf der Registerkarte **Board** im Bereich **Boards** ziehen Sie die Aufgabe aus **Geliefert** in die Phase **QA Genehmigt \|In Bearbeitung** des Verantwortlichkeitsbereichs **Beschleunigen**, sodass erkennbar ist, dass sie Priorität hat, sobald QA-Kapazität verfügbar wird.

    > **Hinweis:** Möglicherweise müssen Sie den Browser aktualisieren, um den Verantwortlichkeitsbereich sichtbar zu machen.

In dieser Übung haben Sie gelernt, wie Sie das Kanban-Board verwenden können, um den Arbeitsfluss auf leicht verständliche Weise zu visualisieren und den Fortschritt zu verfolgen. Sie haben auch gelernt, wie Sie das Board so konfigurieren, dass es den Prozess Ihres Teams unterstützt, und wie Sie das Limit für laufende Arbeiten (WIP) festlegen, um sicherzustellen, dass das Team nicht mit Arbeit überlastet wird.

### Übung 2: Definieren von Dashboards

#### Aufgabe 1: Erstellen und Anpassen von Dashboards

In dieser Aufgabe werden Sie schrittweise durch den Prozess der Erstellung von Dashboards und deren Kernkomponenten geführt.

Dashboards ermöglichen es den Teams, den Status zu visualisieren und den Fortschritt im gesamten Projekt zu überwachen. Auf einen Blick können Sie fundierte Entscheidungen treffen, ohne sich in andere Bereiche Ihrer Team-Projektseite einarbeiten zu müssen. Die Übersichtsseite bietet Zugriff auf ein Standard-Team-Dashboard, das Sie durch Hinzufügen, Entfernen oder Umordnen der Kacheln anpassen können. Jede Kachel entspricht einem Widget, das Zugriff auf eine oder mehrere Funktionen bietet.

1. Wählen Sie im vertikalen Navigationsbereich des Azure DevOps-Portals das Symbol **Übersicht** und in der Liste der Elemente **Übersicht** die Option **Dashboards**.
1. Wählen Sie die **Übersicht** für das **eShopOnWeb-Team** aus und überprüfen Sie das vorhandene Dashboard.

    ![Screenshot der Überblickseite der Dashboards.](images/m1/EShop-WEB-dashboard_v1.png)

1. Wählen Sie im Bereich **Dashboards** in der oberen rechten Ecke **+ Neues Dashboard** aus.

1. Geben Sie im Bereich **Dashboard erstellen** im Textfeld **Name** **`Product training`** ein, wählen Sie in der Dropdown-Liste **Team** das Team **EShop-WEB** aus und klicken Sie auf **Erstellen**.

    ![Screenshot des „Dashboard erstellen“ Panels.](images/m1/EShop-WEB-create_dash_v1.png)

1. Klicken Sie im neuen Dashboard-Bereich auf **Widget hinzufügen**.
1. Geben Sie im Bereich **Widget hinzufügen** in das Textfeld **Widget suchen** **`sprint`** ein, um vorhandene Widgets zu finden, die sich auf Sprints konzentrieren. Wählen Sie in der Ergebnisliste **Sprintübersicht** und klicken Sie auf **Hinzufügen**.
1. Klicken Sie in dem Rechteck, das das neu hinzugefügte Widget darstellt, auf das Zahnradsymbol **Einstellungen** und sehen Sie sich den Bereich **Konfiguration** an.

    > **Hinweis**: Die Anpassungsebene variiert je nach Widget.

1. Klicken Sie im Bereich **Konfiguration** auf **Schließen**, ohne irgendwelche Änderungen vorzunehmen.
1. Zurück im Bereich **Widget hinzufügen** geben Sie im Textfeld **Suchen** erneut **`sprint`** ein, um vorhandene Widgets zu finden, die sich auf Sprints konzentrieren. Wählen Sie in der Ergebnisliste **Sprintkapazität** und klicken Sie auf **Hinzufügen**.
    > **Hinweis**: Wenn im Widget „Kapazität für die Verwendung des Sprint-Kapazitäts-Widgets festlegen“ angezeigt wird, können Sie den Link **Kapazität festlegen** auswählen, um die Kapazität festzulegen. Setzen Sie die Aktivität auf Entwicklung und die Kapazität auf 1. Klicken Sie auf **Speichern** und kehren Sie zum Dashboard zurück.
1. Klicken Sie in der Ansicht **Dashboard** am oberen Rand des Bereichs auf **Bearbeitung abgeschlossen**.

    ![Screenshot des Dashboards mit zwei neuen Widgets.](images/m1/EShop-WEB-finished_dashboard_v1.png)

    > **Hinweis**: Sie können jetzt zwei wichtige Aspekte Ihres aktuellen Sprints in Ihrem benutzerdefinierten Dashboard überprüfen.

    > **Hinweis**: Eine weitere Möglichkeit, Dashboards anzupassen, besteht darin, Diagramme auf der Grundlage von Arbeitselement-Abfragen zu erstellen, die Sie an ein Dashboard weitergeben können.

1. Wählen Sie im vertikalen Navigationsbereich des Azure DevOps-Portals das Symbol **Boards** und wählen Sie in der Liste der **Boards**-Elemente **Abfragen**.
1. Klicken Sie im Bereich **Abfragen** auf **+ Neue Abfrage**.
1. Wählen Sie auf der Registerkarte **Editor** des Fensters **Abfragen > Meine Abfragen** in der Dropdown-Liste **Wert** der Zeile **Arbeitselementtyp** die Option **Aufgabe**.
1. Klicken Sie auf **Neue Klausel hinzufügen**, wählen Sie in der Spalte **Feld** die Option **Bereichspfad** aus und wählen Sie in der entsprechenden Dropdown-Liste **Wert** die Option **eShopOnWeb\\EShop-WEB** aus.
1. Klicken Sie auf **Speichern**.

    ![Screenshot der neuen Abfrage im Abfrage-Editor.](images/m1/EShop-WEB-query_v1.png)

1. Geben Sie in das Textfeld **Name eingeben** den Text **`Web tasks`** ein, wählen Sie in der Dropdown-Liste **Ordner** die Option **Shared Queries** aus und klicken Sie auf **OK**.
1. Klicken Sie im Bereich **Abfragen > Gemeinsame Abfragen** auf **Web-Aufgaben**, um die Abfrage zu öffnen.
1. Wählen Sie die Registerkarte **Diagramme** aus und klicken Sie auf **Neues Diagramm**.
1. Geben Sie im Bereich **Diagramm konfigurieren** im Textfeld **Name** **`Web tasks - By assignment`** ein, wählen Sie in der Dropdown-Liste **Gruppieren nach** die Option **Zugewiesen an** aus und klicken Sie auf **Diagramm speichern**, um die Änderungen zu speichern.

    ![Screenshot des neuen Kreisdiagramms für Web-Aufgaben.](images/m1/EShop-WEB-chart_v1.png)

    > **Hinweis**: Sie können dieses Diagramm jetzt zu einem Dashboard hinzufügen.

1. Kehren Sie zum Abschnitt **Dashboards** im Menü **Übersicht** zurück. Wählen Sie im Abschnitt **EShop-Web** das zuvor von Ihnen verwendete Dashboard **Produkttraining** aus, um es zu öffnen.

1. Klicken Sie im oberen Menü auf **Bearbeiten**. Suchen Sie in der Liste **Widget hinzufügen** nach **`Chart`** und wählen Sie **Darstellen von Arbeitselementen** aus. Klicken Sie auf **Hinzufügen**, um dieses Widget dem EShop-Web-Dashboard hinzuzufügen.

1. Klicken Sie innerhalb von **Diagramm für Arbeitselemente** auf **Konfigurieren** (Zahnrad), um die Widgeteinstellungen zu öffnen.

1. Akzeptieren Sie den Titel so wie er ist. Wählen Sie unter **Abfrage** die Option **Freigegebene Abfragen/ Webaufgaben** aus. Behalten Sie **Kreis** für den Diagrammtyp bei. Wählen Sie unter **Gruppieren nach** die Option **Zugewiesen zu** aus. Behalten Sie die Standardwerte Aggregation (Anzahl) und Sortierung (Wert/Aufsteigend) bei.

1. Bestätigen Sie die Konfiguration, indem Sie auf **Speichern** klicken.

1. Beachten Sie, dass das Abfrageergebnis-Kreisdiagramm im Dashboard angezeigt wird. **Speichern** Sie die Änderungen, indem Sie oben auf die Schaltfläche **Bearbeitung beendet** drücken.

## Überprüfung

In diesem Lab haben Sie Azure Boards verwendet, um eine Reihe gängiger Aufgaben im Bereich Agile Planning And Portfolio Management durchzuführen, darunter die Verwaltung von Teams, Bereichen, Iterationen, Arbeitselementen, Sprints und Kapazitäten, die Anpassung von Kanban-Boards und die Definition von Dashboards.
