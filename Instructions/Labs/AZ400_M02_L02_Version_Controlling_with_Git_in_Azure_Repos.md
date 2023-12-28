---
lab:
  title: Versionskontrolle mit Git in Azure Repos
  module: 'Module 02: Work with Azure Repos and GitHub'
---

# Versionskontrolle mit Git in Azure Repos

## Lab-Handbuch für Kursteilnehmer

## Labanforderungen

- Für dieses Lab ist **Microsoft Edge** oder ein [von Azure DevOps unterstützter Browser](https://docs.microsoft.com/azure/devops/server/compatibility) erforderlich.

- **Einrichten einer Azure DevOps-Organisation**: Wenn Sie nicht bereits eine Azure DevOps-Organisation haben, die Sie für dieses Lab verwenden können, müssen Sie diese erstellen, indem Sie die unter [Erstellen einer Organisation oder Projektsammlung](https://docs.microsoft.com/azure/devops/organizations/accounts/create-organization) beschriebenen Anweisungen befolgen.

- Falls Sie kein Git 2.29.2 oder höher installiert haben, starten Sie einen Webbrowser, navigieren zur Downloadseite von Git für Windows und führen die Installation aus.
- Wenn Sie Visual Studio Code noch nicht installiert haben, navigieren Sie im Webbrowserfenster zur [Downloadseite von Visual Studio Code](https://code.visualstudio.com/), laden ihn herunter und führen die Installation aus.
- Wenn Sie die Visual Studio C#-Erweiterung noch nicht installiert haben, navigieren Sie im Webbrowserfenster zur [Installationsseite der C#-Erweiterung](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp), und installieren Sie sie.

## Übersicht über das Labor

Azure DevOps unterstützt zwei Arten von Versionskontrolle: Git und Team Foundation-Versionskontrolle (TFVC). Hier finden Sie eine kurze Übersicht über die beiden Versionskontrollsysteme:

- **Team Foundation-Versionskontrolle (TFVC)**: TFVC ist ein zentralisiertes Versionskontrollsystem. In der Regel verfügen Teammitglieder auf ihren Entwicklungscomputern nur über eine Version jeder Datei. Daten zur Versionsgeschichte einer Datei werden nur auf dem Server gespeichert. Verzweigungen sind pfadbasiert und werden auf dem Server erstellt.

- **Git**: Git ist ein verteiltes Versionskontrollsystem. Git-Repositorys können lokal (auf dem Computer eines Entwicklers) vorhanden sein. Jeder Entwickler verfügt auf seinem Entwicklungscomputer über eine Kopie des Quellrepositorys. Entwickler können für alle Änderungen auf ihrem Entwicklungscomputer Commits ausführen und Versionskontrollvorgänge wie den Aufruf von Versionsgeschichten oder Vergleichen ohne Netzwerkverbindung vornehmen.

Git ist der Standardanbieter für Versionskontrolle für neue Projekte. Sie sollten Git für die Versionskontrolle in Ihren Projekten verwenden, es sei denn, Sie benötigen zentrale Versionskontrollfeatures in TFVC.

In diesem La lernen Sie, ein lokales Git-Repository einzurichten, das problemlos mit einem zentralen Git-Repository in Azure DevOps synchronisiert werden kann. Darüber hinaus erfahren Sie mehr über Git-Branching- und -Mergingunterstützung. Sie verwenden Visual Studio Code, aber die gleichen Prozesse gelten für die Verwendung eines beliebigen Git-kompatiblen Clients.

## Ziele

In diesem Lab lernen Sie Folgendes:

- Klonen eines vorhandenen Repositorys.
- Speichern von Arbeit mit Commits.
- Überprüfen des Verlaufs der Änderungen.
- Arbeiten mit Branches mithilfe von Visual Studio Code.

## Geschätzte Zeit: 60 Minuten

## Anweisungen

### Übung 0: Konfigurieren der Voraussetzungen für das Lab.

In dieser Übung richten Sie die Voraussetzungen für das Labor ein, das aus einem neuen Azure DevOps-Projekt mit einem Repository basierend auf dem [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb) besteht.

#### Aufgabe 1: (überspringen, wenn fertig) Erstellen und Konfigurieren des Teamprojekts

In dieser Aufgabe erstellen Sie ein **eShopOnWeb** Azure DevOps-Projekt, das von mehreren Laboren verwendet werden soll.

1. Öffnen Sie auf Ihrem Laborcomputer in einem Browserfenster Ihre Azure DevOps-Organisation. Klicken auf „Neues Projekt“ Geben Sie Ihrem Projekt den Namen **"eShopOnWeb** ", und wählen Sie **"Scrum** " in der **Dropdownliste "Arbeitsaufgabe"** aus. Klicken Sie auf **Erstellen**.

    ![Erstellen eines Projekts](images/create-project.png)

#### Aufgabe 2: (überspringen, wenn erledigt) Importieren von eShopOnWeb Git Repository

Bei dieser Aufgabe importieren Sie das eShopOnWeb Git-Repository, das von mehreren Labs verwendet wird.

1. Öffnen Sie auf Ihrem Laborcomputer in einem Browserfenster Ihre Azure DevOps-Organisation und das zuvor erstellte **eShopOnWeb-Projekt** . Klicken Sie auf **Repos>Files** , **Import**. Fügen Sie im **Fenster "Git Repository** importieren" die folgende URL https://github.com/MicrosoftLearning/eShopOnWeb.git  ein, und klicken Sie auf " **Importieren**":

    ![Importieren eines Repositorys](images/import-repo.png)

2. Das Repository ist wie folgt organisiert:
    - Der Ordner „.ado“ enthält Azure DevOps-YAML-Pipelines.
    - Der Ordner „.devcontainer“ enthält ein Containersetup für die Entwicklung mithilfe von Containern (entweder lokal in VS Code oder über GitHub Codespaces).
    - **Azure-Ordner** enthält Bicep&ARM-Infrastruktur als Codevorlagen, die in einigen Lab-Szenarien verwendet werden.
    - **GITHUB-Ordnercontainer-YAML-GitHub-Workflowdefinitionen** .
    - Der Ordner „src“ enthält die .NET 6-Website, die in den Labszenarien verwendet wird.

#### Aufgabe 3: Konfigurieren von Git und Visual Studio Code

In dieser Aufgabe installieren und konfigurieren Sie Git und Visual Studio Code, einschließlich der Konfiguration der Git-Anmeldeinformationshilfe zum sicheren Speichern der Git-Anmeldeinformationen, die für die Kommunikation mit Azure DevOps verwendet werden. Wenn Sie diese Voraussetzungen bereits implementiert haben, können Sie direkt mit der nächsten Aufgabe fortfahren.

1. Starten Sie auf dem Lab-Computer Visual Studio Code.
2. Wählen Sie in der Visual Studio Code-Schnittstelle im Menü Standard die Option **Terminal New Terminal** aus, um den **** TERMINAL-Bereich \| zu öffnen.
3. Stellen Sie sicher, dass das aktuelle Terminal PowerShell** ausführt**, indem Sie überprüfen, ob in der Dropdownliste in der oberen rechten Ecke des **TERMINALbereichs** "1: powershell" angezeigt wird **.**

    > **Hinweis**: Um die aktuelle Terminalshell in PowerShell** zu **ändern, klicken Sie auf die Dropdownliste in der oberen rechten Ecke des **TERMINAL-Bereichs**, und klicken Sie auf **"Standardshell** auswählen". Wählen Sie oben im Visual Studio Code-Fenster Die bevorzugte Terminalshell **windows PowerShell** aus, und klicken Sie auf das Pluszeichen auf der rechten Seite der Dropdownliste, um ein neues Terminal mit der ausgewählten Standardshell zu öffnen.

4. Führen Sie im **TERMINALbereich** den folgenden Befehl aus, um das Hilfsprogramm für Anmeldeinformationen zu konfigurieren.

    ```git
    git config --global credential.helper wincred
    ```

5. Führen Sie im **TERMINALbereich** die folgenden Befehle aus, um einen Benutzernamen und eine E-Mail für Git Commits zu konfigurieren (ersetzen Sie die Platzhalter in geschweiften Klammern durch Ihren bevorzugten Benutzernamen und ihre E-Mail, wodurch die symbole < und > entfernt werden):

    ```git
    git config --global user.name "<John Doe>"
    git config --global user.email <johndoe@example.com>
    ```

### Übung 1: Klonen eines vorhandenen Repositorys.

In dieser Übung verwenden Sie Visual Studio Code, um das Git-Repository zu klonen, das Sie als Teil der vorherigen Übung bereitgestellt haben.

#### Klonen eines vorhandenen Repositorys

In dieser Aufgabe durchlaufen Sie den Prozess des Klonens eines Git-Repositorys mithilfe von Visual Studio Code.

1. Wechseln Sie zum Webbrowser, in dem Ihre Azure DevOps-Organisation mit dem eShopOnWeb-Projekt** angezeigt wird, das **Sie in der vorherigen Übung generiert haben.
2. Wählen Sie im vertikalen Navigationsbereich des Azure DevOps-Portals das **Symbol "Repos** " aus.

3. Klicken Sie in der oberen rechten Ecke des **eShopOnWeb-Repositorybereichs** auf Klonen****.

    ![Git-Repository klonen](images/clone-repo.png)

    > **Hinweis**: Das Abrufen einer lokalen Kopie eines Git-Repositorys wird als Klonen* bezeichnet*. Jedes Standard stream-Entwicklungstool unterstützt dies und kann eine Verbindung mit Azure Repos herstellen, um die neueste Quelle abzurufen, mit der sie arbeiten kann.

4. Klicken Sie im **Bereich "Repository klonen** " mit ausgewählter **HTTPS-Befehlszeilenoption** auf die **Schaltfläche "In Zwischenablage** kopieren" neben der Url des Repositoryklons.

    > **Hinweis**: Sie können diese URL mit jedem gitkompatiblen Tool verwenden, um eine Kopie der Codebasis abzurufen.

5. Schließen Sie den **Bereich "Klon-Repository** ".
6. Wechseln Sie auf dem Lab-Computer zu Visual Studio Code.
7. Klicken Sie auf die **Menüüberschrift "Ansicht** ", und klicken Sie im Dropdownmenü auf **"Befehlspalette"**.

    > **Hinweis**: Die Befehlspalette bietet eine einfache und bequeme Möglichkeit, auf eine Vielzahl von Aufgaben zuzugreifen, einschließlich derer, die als Drittanbietererweiterungen implementiert wurden. Alternativ können Sie auch die Tastenkombination STRG+UMSCHALT+P verwenden.

8. Führen Sie an der Eingabeaufforderung der Befehlspalette den **Befehl "Git: Klonen"** aus.

    ![VS Code-Befehlspalette](images/vscode-command.png)

    > **Hinweis**: Um alle relevanten Befehle anzuzeigen, können Sie mit der Eingabe **von Git** beginnen.

9. Fügen Sie in der Url " **Repository bereitstellen" ein, oder wählen Sie ein Textfeld für die Repositoryquelle** aus, fügen Sie die url des Repositoryklons ein, die Sie zuvor in dieser Aufgabe kopiert haben, und drücken Sie die **EINGABETASTE** .
10. **Navigieren Sie im Dialogfeld "Ordner** auswählen" zum Laufwerk "C:", erstellen Sie einen neuen Ordner namens **"Git**", wählen Sie ihn aus, und klicken Sie dann auf **"Repositoryspeicherort** auswählen".
11. Melden Sie sich an Ihrem Azure-Konto an, wenn die Aufforderung angezeigt wird.
12. Klicken Sie nach Abschluss des Klonvorgangs im Visual Studio Code auf " **Öffnen** ", um das geklonte Repository zu öffnen.

    > **Hinweis**: Sie können Warnungen ignorieren, die Bei Problemen beim Laden des Projekts auftreten können. Die Lösung befindet sich möglicherweise nicht im Zustand, der für einen Build geeignet ist, aber wir konzentrieren uns auf die Arbeit mit Git, sodass das Erstellen des Projekts nicht erforderlich ist.

### Übung 2: Speichern von Arbeit mit Commits.

In dieser Übung durchlaufen Sie mehrere Szenarien, in denen Visual Studio Code verwendet wird, um Änderungen zu stufen und zu übernehmen.

Wenn Sie Änderungen an Ihren Dateien vornehmen, zeichnet Git die Änderungen im lokalen Repository auf. Sie können die Änderungen auswählen, die Sie übernehmen möchten, indem Sie sie bereitstellen. Commits werden immer gegen Ihr lokales Git-Repository vorgenommen, daher müssen Sie sich keine Gedanken darüber machen, dass der Commit perfekt ist oder bereit ist, mit anderen zu teilen. Sie können weitere Commits vornehmen, während Sie weiterhin arbeiten und die Änderungen an andere Personen übertragen, wenn sie bereit sind, freigegeben zu werden.

Git Commits besteht aus den folgenden Komponenten:

- Committe die Änderungen an die -Datei. Git behält den Inhalt aller Dateiänderungen in Ihrem Repository in den Commits bei. Dadurch bleibt es schnell und ermöglicht intelligente Zusammenführungen.
- Ein Verweis auf übergeordnete Commits. Git verwaltet Ihren Codeverlauf mithilfe dieser Verweise.
- Eine Nachricht, die einen Commit beschreibt. Sie geben git diese Nachricht an, wenn Sie den Commit erstellen. Es ist ratsam, diese Nachricht beschreibend zu halten, aber zu dem Punkt.

#### Aufgabe 1: Übernehmen von Änderungen

In dieser Aufgabe verwenden Sie Visual Studio Code, um Änderungen zu übernehmen.

1. Wählen Sie im Visual Studio Code-Fenster oben auf der vertikalen Symbolleiste die **Registerkarte EXPLORER** aus, navigieren Sie zur **Datei "/eShopOnWeb/src/Web/Program.cs** ", und wählen Sie sie aus. Dadurch wird der Inhalt automatisch im Detailbereich angezeigt.
2. Fügen Sie die folgenden Zeilen nach dem Kommentar hinzu:

    ```csharp
    // My first change
    ```

    > **Hinweis**: Es spielt keine Rolle, was der Kommentar ist, da das Ziel darin besteht, nur eine Änderung vorzunehmen.

3. Drücken Sie **STRG+S** , um die Änderung zu speichern.
4. Wählen Sie im Visual Studio Code-Fenster die **Registerkarte "QUELLCODEVERWALTUNG** " aus, um zu überprüfen, ob Git die neueste Änderung an der Datei erkannt hat, die sich im lokalen Klon des Git-Repositorys befindet.
5. Wenn die **Registerkarte "QUELLCODEVERWALTUNG** " am oberen Rand des Bereichs im Textfeld ausgewählt ist, geben Sie **"Mein Commit"** als Commitnachricht ein, und drücken **Sie STRG+EINGABETASTE** , um ihn lokal zu übernehmen.

    ![Erster Commit](images/first-commit.png)

6. Wenn Sie dazu aufgefordert werden sollen, ihre Änderungen automatisch zu stufen und direkt zu übernehmen, klicken Sie auf **"Immer"**.

    > **Hinweis**: Wir werden das Staging** später im Labor besprechen**.

7. Notieren Sie sich in der unteren linken Ecke des Visual Studio Code-Fensters rechts neben der beschriftung Standard **** das **Symbol "Änderungen** synchronisieren" eines Kreises mit zwei vertikalen Pfeilen, die in die entgegengesetzte Richtung zeigen, und die Zahl **1** neben dem Pfeil, der nach oben zeigt. Klicken Sie auf das Symbol, und klicken Sie, wenn Sie dazu aufgefordert werden, auf **"OK**", um Commits an den Ursprung/Standard** zu übertragen und von **dieser zu ziehen.

#### Aufgabe 2: Überprüfen von Commits

In dieser Aufgabe verwenden Sie das Azure DevOps-Portal, um Commits zu überprüfen.

1. Wechseln Sie zum Webbrowserfenster, in dem das Azure-Portal angezeigt wird.
2. Wählen Sie im vertikalen Navigationsbereich des Azure DevOps-Portals im **Abschnitt "Repos** " die Option **"Commits**" aus.
3. Vergewissern Sie sich, dass Ihr Commit oben in der Liste angezeigt wird.

    ![ADO-Repository-Commits](images/ado-commit.png)

#### Aufgabe 3: Phasenänderungen

In dieser Aufgabe untersuchen Sie die Verwendung von Stagingänderungen mithilfe von Visual Studio Code. Mit Stagingänderungen können Sie bestimmte Dateien selektiv zu einem Commit hinzufügen, während sie die in anderen Dateien vorgenommenen Änderungen übergeben.

1. Wechseln Sie zurück zum **Visual Studio Code**-Fenster.
2. Aktualisieren Sie die open **Program.cs-Klasse** , indem Sie den ersten Kommentar mit der folgenden Datei ändern und die Datei speichern.

    ```csharp
        //My second change
    ```

3. Wechseln Sie im Visual Studio Code-Fenster zurück zur **Registerkarte EXPLORER** , navigieren Sie zur **Datei "/eShopOnWeb/src/Web/Constants.cs** ", und wählen Sie sie aus. Dadurch wird der Inhalt automatisch im Detailbereich angezeigt.
4. Fügen Sie der **Datei "Constants.cs** " einen Kommentar in der ersten Zeile hinzu, und speichern Sie die Datei.

    ```csharp
    // My third change
    ```

5. Wechseln Sie im Visual Studio Code-Fenster zur **Registerkarte "QUELLCODEVERWALTUNG** ", zeigen Sie mit dem Mauszeiger auf den **Eintrag "Program.cs** ", und klicken Sie auf das Pluszeichen auf der rechten Seite dieses Eintrags.

    > **Hinweis**: Dadurch wird die Änderung nur in der **Datei "Program.cs** " ausgeführt, soweit sie ohne **Constants.cs** für commit vorbereitet wird.

6. Wenn die **Registerkarte "QUELLCODEVERWALTUNG** " am oberen Rand des Bereichs ausgewählt ist, geben **Sie im Textfeld "Hinzugefügte Kommentare** " als Commitnachricht ein.

    ![Bereitgestellte Änderungen](images/staged-changes.png)

7. Klicken Sie oben auf der **Registerkarte "QUELLCODEVERWALTUNG**" auf das Symbol mit den Auslassungspunkten, wählen Sie **im Dropdownmenü "Commit ausführen"** aus, und wählen Sie im Kaskadierungsmenü die Option "Commit stufend **" aus**.
8. Klicken Sie in der unteren linken Ecke des Visual Studio Code-Fensters auf die **Schaltfläche "Änderungen synchronisieren**", um die zugesicherten Änderungen mit dem Server zu synchronisieren. Wenn Sie dazu aufgefordert werden, klicken Sie auf **"OK**", um Commits an den Ursprung/Standard** zu übertragen und daraus **zu ziehen.

    > **Hinweis**: Da nur die mehrstufige Änderung übernommen wurde, steht die andere Änderung noch aus, um synchronisiert zu werden.

### Übung 3: Überprüfen des Verlaufs.

In dieser Übung verwenden Sie das Azure DevOps-Portal, um den Verlauf von Commits zu überprüfen.

Git verwendet die übergeordneten Referenzinformationen, die in jedem Commit gespeichert sind, um einen vollständigen Verlauf Ihrer Entwicklung zu verwalten. Sie können diesen Commit-Verlauf ganz einfach überprüfen, um herauszufinden, wann Dateiänderungen vorgenommen wurden, und Unterschiede zwischen Versionen Ihres Codes mithilfe des Terminals oder von einer der vielen verfügbaren Visual Studio Code-Erweiterungen ermitteln. Sie können Daten auch über das Azure-Portal anzeigen.

Die Verwendung der **Funktion "Verzweigungen und Zusammenführungen** " von Git funktioniert über Pullanforderungen, sodass der Commitverlauf Ihrer Entwicklung nicht unbedingt eine gerade, chronologische Linie bildet. Wenn Sie den Verlauf zum Vergleichen von Versionen verwenden, denken Sie in Bezug auf Dateiänderungen zwischen zwei Commits anstelle von Dateiänderungen zwischen zwei Zeitpunkten nach. Eine kürzlich vorgenommene Änderung an einer Datei in der Standard-Verzweigung stammt möglicherweise aus einem Commit, der vor zwei Wochen in einer Featurebranch erstellt wurde, die gestern zusammengeführt wurde.

#### Aufgabe 1: Vergleichen von Dateien

In dieser Aufgabe durchlaufen Sie den Commitverlauf mithilfe des Azure DevOps-Portals.

1. Wählen Sie auf der **Registerkarte "QUELLCODEVERWALTUNG**" des Visual Studio Code-Fensters "Constants.cs **" aus, das die nicht mehrstufige Version der Datei **darstellt.

    ![Rollenvergleich](images/file-comparison.png)

    > **Hinweis**: Eine Vergleichsansicht wird geöffnet, damit Sie die vorgenommenen Änderungen problemlos finden können. In diesem Fall ist es nur der einzige Kommentar.

2. Wechseln Sie zum Webbrowserfenster, in dem der **Bereich "Commits** " des **Azure DevOps-Portals** angezeigt wird, um die Quellzweige zu überprüfen und zusammenzuführen. Diese bieten eine bequeme Möglichkeit, zu visualisieren, wann und wie Änderungen an der Quelle vorgenommen wurden.
3. Scrollen Sie nach unten zum **Eintrag "Mein Commit"** (vor gedrückt), und zeigen Sie mit dem Mauszeiger darauf, um das Auslassungszeichen auf der rechten Seite anzuzeigen.
4. Klicken Sie auf die Auslassungspunkte, wählen Sie **im Dropdownmenü "Dateien** durchsuchen" aus, und überprüfen Sie die Ergebnisse.

    ![Commit durchsuchen](images/commit-browse.png)

    > **Hinweis**: Diese Ansicht stellt den Status der Quelle dar, die dem Commit entspricht, sodass Sie die einzelnen Quelldateien überprüfen und herunterladen können.

### Übung 4: Arbeiten mit Branches.

In dieser Übung durchlaufen Sie Szenarien, die die Verzweigungsverwaltung mithilfe von Visual Studio Code und dem Azure DevOps-Portal umfassen.

Sie können in Ihrem Azure DevOps Git-Repository über die **Ansicht "Branches** " von **Azure Repos** im Azure DevOps-Portal verwalten. Passen Sie die Ansicht an, um die Branches nachzuverfolgen, die Ihnen am wichtigsten sind, damit Sie bei den von Ihrem Team vorgenommenen Änderungen auf dem Laufenden bleiben.

Das Übernehmen von Änderungen an einer Verzweigung wirkt sich nicht auf andere Verzweigungen aus, und Sie können Zweigniederlassungen für andere freigeben, ohne die Änderungen in das Standard Projekt zusammenführen zu müssen. Sie können neue Branches erstellen, um Änderungen für ein Feature oder eine Fehlerkorrektur von Ihrem Mainbranch und anderen Arbeiten zu isolieren. Aufgrund der Einfachheit von Branches ist ein schneller und einfacher Wechsel zwischen Branches möglich. Git erstellt nicht mehrere Kopien Ihres Quellcodes, wenn Sie mit Branches arbeiten. Es verwendet die in Commits enthaltenen Verlaufsinformationen, um die Dateien in einem Branch neu zu erstellen, wenn Sie beginnen, daran zu arbeiten. Ihr Git-Workflow sollte Branches zum Verwalten von Features und Fehlerkorrekturen erstellen und verwenden. Der gesamte Rest des Git-Workflows, z. B. das Teilen von Code und das Überprüfen von Code mit Pull Requests, wird über Branches erledigt. Die Isolierung von Arbeiten in Branches erleichtert die Änderung, was Sie gerade bearbeiten, indem Sie Ihren aktuellen Branch ändern.

#### Erstellen Sie einen neuen lokalen Branch in Ihrem geklonten Repository.

In dieser Aufgabe erstellen Sie eine Verzweigung mithilfe von Visual Studio Code.

1. Wechseln Sie auf dem Lab-Computer zu Visual Studio Code.
2. Klicken Sie mit ausgewählter **Registerkarte "QUELLCODEVERWALTUNG**" in der unteren linken Ecke des Visual Studio Code-Fensters auf **Standard**.
3. Wählen Sie im Popupfenster + **Neue Verzweigung erstellen aus...** aus.

    ![Verzweigung erstellen](images/create-branch.png)

4. Geben Sie **im **Textfeld "Verzweigungsname**" dev** ein, um die neue Verzweigung anzugeben, und drücken Sie die **EINGABETASTE**.
5. Wählen Sie im **Textfeld "Referenzverzweigung auswählen" aus, um die Verzweigung** "dev" zu erstellen, Standard** als Referenzverzweigung aus**.

    > **Hinweis**: Zu diesem Zeitpunkt werden Sie automatisch in den **Dev** Branch umgestellt.

#### Aufgabe 2: Löschen einer Verzweigung

In dieser Aufgabe verwenden Sie Visual Studio Code, um mit einer Verzweigung zu arbeiten, die in der vorherigen Aufgabe erstellt wurde.

Git verfolgt, an welchem Branch Sie arbeiten, und stellt sicher, dass Ihre Dateien beim -Vorgang für einen Branch mit dem letzten Commit im Branch übereinstimmen. Branches bieten die Möglichkeit, gleichzeitig mit mehreren Versionen des Quellcodes im gleichen lokalen Git-Repository zu arbeiten. Sie können Visual Studio Code verwenden, um Verzweigungen zu veröffentlichen, auszuchecken und zu löschen.

1. Klicken Sie im **Visual Studio Code-Fenster** mit ausgewählter **Registerkarte "QUELLCODEVERWALTUNG** " in der unteren linken Ecke des Visual Studio Code-Fensters auf das **Symbol "Änderungen** veröffentlichen" (direkt rechts neben der **Dev-Bezeichnung** , die Ihre neu erstellte Verzweigung darstellt).
2. Wechseln Sie zum Webbrowserfenster, in dem der **Bereich "Commits**" des **Azure DevOps-Portals** angezeigt wird, und wählen Sie "Verzweigungen"** aus**.
3. Überprüfen Sie auf der **Registerkarte "Mine**" im **Bereich "Verzweigungen**", ob die Liste der Verzweigungen Dev** enthält**.
4. Zeigen Sie mit dem Mauszeiger auf den **Dev** Branch-Eintrag, um das Auslassungszeichen auf der rechten Seite anzuzeigen.
5. Klicken Sie auf die Auslassungspunkte, wählen Sie **im Popupmenü "Verzweigung löschen"** aus, und klicken Sie, wenn Sie zur Bestätigung aufgefordert werden, auf **"Löschen"**.

    ![Branch löschen](images/delete-branch.png)

6. Wechseln Sie zurück zum **Visual Studio Code-Fenster** , und klicken Sie mit ausgewählter **Registerkarte "QUELLCODEVERWALTUNG** " in der unteren linken Ecke des Visual Studio Code-Fensters auf den **Dev-Eintrag** . Dadurch werden die vorhandenen Verzweigungen im oberen Teil des Visual Studio Code-Fensters angezeigt.
7. Stellen Sie sicher, dass jetzt zwei **Dev** Branches aufgelistet sind.

    > **Hinweis**: Die lokale Verzweigung (**Dev**) wird aufgelistet, da sie von der Löschung der Verzweigung im Remoterepository nicht betroffen ist. Der Server (**Origin/Dev**) wird aufgelistet, da er nicht gekürzt wurde.

8. Wählen Sie in der Liste **Standardbranch** den auszucheckenden Branch aus.
9. Drücken Sie STRG+UMSCHALT+P, um die Befehlspalette zu öffnen.
10. Beginnen Sie an der Eingabeaufforderung der Befehlspalette mit der **Eingabe von **Git: "Löschen**" und wählen Sie **"Git: Verzweigung** löschen" aus, wenn sie sichtbar** wird.
11. Wählen Sie den **Dev-Eintrag** in der Liste der zu löschenden Verzweigungen aus.
12. Klicken Sie in der unteren linken Ecke des Visual Studio Code-Fensters erneut auf den **Standard** Eintrag. Dadurch werden die vorhandenen Verzweigungen im oberen Teil des Visual Studio Code-Fensters angezeigt.
13. Stellen Sie sicher, dass der lokale **Dev** Branch nicht mehr in der Liste angezeigt wird, aber der Remoteursprung **/-dev** ist noch vorhanden.
14. Drücken Sie STRG+UMSCHALT+P, um die Befehlspalette zu öffnen.
15. Beginnen Sie an der Eingabeaufforderung der Befehlspalette mit der **Eingabe von **Git: Fetch** and select **Git: Fetch (Prune),** wenn sie sichtbar** wird.

    > **Hinweis**: Mit diesem Befehl werden die Ursprungsverzweigungen in der lokalen Momentaufnahme aktualisiert und gelöscht, die nicht mehr vorhanden sind.

    > **Hinweis**: Sie können genau überprüfen, was diese Aufgaben ausführen, indem Sie im unteren rechten Teil des Visual Studio Code-Fensters das **Ausgabefenster** auswählen. Wenn die Git-Protokolle in der Ausgabekonsole nicht angezeigt werden, stellen Sie sicher, dass Sie Git** als Quelle auswählen**.

16. Klicken Sie in der unteren linken Ecke des Visual Studio Code-Fensters erneut auf den **Standard** Eintrag.
17. Stellen Sie sicher, dass der **Origin/Dev** Branch nicht mehr in der Liste der Verzweigungen angezeigt wird.

#### Aufgabe 3: Wiederherstellen einer Verzweigung

In dieser Aufgabe verwenden Sie das Azure DevOps-Portal, um die Verzweigung wiederherzustellen, die Sie in der vorherigen Aufgabe gelöscht haben.

1. Wechseln Sie zum Webbrowser, der die **Registerkarte "Mine** " im **Bereich "Verzweigungen** " im Azure DevOps-Portal anzeigt.
2. Wählen Sie auf der **Registerkarte "Meine** " im **Bereich "Verzweigungen** " die **Registerkarte "Alle** " aus.
3. Geben **Sie auf der **Registerkarte "Alle**" des **Bereichs "Verzweigungen**" im **Textfeld "Verzweigungsname** suchen" "Dev"** ein.
4. Überprüfen Sie den **Abschnitt "Gelöschte Verzweigungen** ", der den Eintrag enthält, der die neu gelöschte Verzweigung darstellt.
5. Zeigen Sie im **Abschnitt "Gelöschte Verzweigungen** " mit dem Mauszeiger auf den **Dev** Branch-Eintrag, um das Auslassungszeichen auf der rechten Seite anzuzeigen.
6. Klicken Sie im Popupmenü auf die Auslassungspunkte, und wählen Sie **"Verzweigung wiederherstellen"** aus.

    ![Verzweigung wiederherstellen](images/restore-branch.png)

    > **Hinweis**: Sie können diese Funktion verwenden, um eine gelöschte Verzweigung wiederherzustellen, solange Sie den genauen Namen kennen.

#### Aufgabe 4: Verzweigungsrichtlinien

In dieser Aufgabe verwenden Sie das Azure DevOps-Portal, um dem Standard Branch Richtlinien hinzuzufügen und nur Änderungen mithilfe von Pullanforderungen zuzulassen, die den definierten Richtlinien entsprechen. Sie möchten sicherstellen, dass Änderungen in einer Verzweigung überprüft werden, bevor sie zusammengeführt werden.

Der Einfachheit halber arbeiten wir direkt im Webbrowser-Repository-Editor (direkt im Ursprung), anstatt den lokalen Klon im VS-Code zu verwenden (empfohlen für reale Szenarien).

1. Wechseln Sie zum Webbrowser, der die **Registerkarte "Mine** " im **Bereich "Filialen** " im Azure DevOps-Portal anzeigt.
2. Zeigen Sie auf der **Registerkarte "Mine**" im **Bereich "Verzweigungen**" mit dem Mauszeiger auf den **Standard** Verzweigungseintrag, um das Auslassungszeichen auf der rechten Seite anzuzeigen.
3. Klicken Sie auf die Auslassungspunkte, und wählen Sie **im Popupmenü "Verzweigungsrichtlinien**" aus.

    ![Branchrichtlinien](images/branch-policies.png)

4. Aktivieren Sie auf der **Registerkarte Standard** der Repositoryeinstellungen die Option für **die Mindestanzahl der Prüfer** erforderlich. Fügen Sie 1** Prüfer hinzu, und aktivieren Sie **das Kontrollkästchen **"Anforderer zulassen", um ihre eigenen Änderungen** zu genehmigen(da Sie der einzige Benutzer in Ihrem Projekt für das Labor sind)
5. Aktivieren Sie auf der **Registerkarte Standard** der Repositoryeinstellungen die Option "**Auf verknüpfte Arbeitsaufgaben** überprüfen" und lassen Sie sie mit **der Option "Erforderlich**" belassen.

    ![Richtlinieneinstellungen](images/policy-settings.png)

#### Aufgabe 5: Testen der Verzweigungsrichtlinie

In dieser Aufgabe verwenden Sie das Azure DevOps-Portal, um die Richtlinie zu testen und Ihre erste Pullanforderung zu erstellen.

1. Stellen Sie im vertikalen Navigationsbereich des Azure DevOps-Portals im **Repos>Files** sicher, dass die **Standard** Verzweigung ausgewählt ist (Dropdown oben gezeigter Inhalt).
2. Um sicherzustellen, dass Richtlinien funktionieren, versuchen Sie, eine Änderung vorzunehmen und für die **Standard** Branch zu übernehmen, navigieren Sie zur **Datei "/eShopOnWeb/src/Web/Program.cs**", und wählen Sie sie aus. Dadurch wird der Inhalt automatisch im Detailbereich angezeigt.
3. Fügen Sie die folgenden Zeilen nach dem Kommentar hinzu:

    ```csharp
    // Testing main branch policy
    ```

4. Klicken Sie auf **Commit > Commit**. Es wird eine Warnung angezeigt: Änderungen an der Standard Verzweigung können nur mithilfe einer Pullanforderung ausgeführt werden.

    ![Richtlinie verweigert Commit](images/policy-denied.png)

5. Klicken Sie auf **'Abbrechen'** , um den Commit zu überspringen.

#### Arbeiten mit Pull Requests

In dieser Aufgabe verwenden Sie das Azure DevOps-Portal, um eine Pullanforderung zu erstellen, indem Sie den **Dev** Branch verwenden, um eine Änderung in der geschützten **Standard** Verzweigung zusammenzuführen. Eine Azure DevOps-Arbeitsaufgabe, die mit den Änderungen verknüpft ist, um ausstehende Arbeit mit Codeaktivitäten nachverfolgen zu können.

1. Wählen Sie im vertikalen Navigationsbereich des Azure DevOps-Portals im **Abschnitt "Boards** " die Option **"Arbeitsaufgaben"** aus.
2. Klicken Sie auf **+ Neue Arbeitsaufgabe > Produktrückstandselement**. Schreiben Sie **im Titelfeld "Testen meiner ersten PR"**, und klicken Sie auf "Speichern"****.
3. Wechseln Sie nun zum vertikalen Navigationsbereich des Azure DevOps-Portals im **Repository>Files**, um sicherzustellen, dass der **Dev** Branch ausgewählt ist.
4. Navigieren Sie zur **Datei "/eShopOnWeb/src/Web/Program.cs** ", und nehmen Sie die folgende Änderung in der ersten Zeile vor:

    ```csharp
    // Testing my first PR
    ```

5. Klicken Sie auf **Commit > Commit** (Standard-Commit-Nachricht verlassen). Diesmal funktioniert der Commit, **hat Dev** Branch keine Richtlinien.
6. Eine Nachricht wird eingeblendt und schlägt vor, eine Pullanforderung zu erstellen (während Sie **branch** jetzt im Vergleich zu **Standard** im Voraus sind). Klicke auf **Pull Request erstellen**.

    ![Erstellen einer Pullanforderung](images/create-pr.png)

7. Lassen Sie auf der **Registerkarte "Neue Pullanforderung**" die Standardeinstellungen, und klicken Sie auf "Erstellen"****.
8. Die Pullanforderung zeigt einige fehlgeschlagene/ausstehende Anforderungen an, basierend auf den Richtlinien, die auf unsere Ziel-Standard **** Verzweigung angewendet wurden.
    - Vorgeschlagene Änderungen sollten eine Arbeitsaufgabe verknüpft haben
    - Mindestens 1 Benutzer sollten die Änderungen überprüfen und genehmigen.

9. Klicken Sie auf der rechten Seite auf die Schaltfläche neben **"****+Arbeitselemente".** Verknüpfen Sie die zuvor erstellte Arbeitsaufgabe mit der Pullanforderung, indem Sie darauf klicken. Es wird eins der Anforderungen angezeigt, die den Status "Änderungen" ändern.

    ![Verknüpfen von Arbeitsaufgaben](images/link-wit.png)

10. Öffnen Sie als Nächstes die **Registerkarte "Dateien** ", um die vorgeschlagenen Änderungen zu überprüfen. In einer vollständigeren Pullanforderung können Sie Dateien einzeln (als überprüft markiert) überprüfen und Kommentare für Zeilen öffnen, die möglicherweise nicht klar sind (wenn Sie mit der Maus auf die Zeilennummer zeigen, haben Sie die Möglichkeit, einen Kommentar zu posten).
11. Wechseln Sie zurück zur Registerkarte "Übersicht **", und klicken Sie oben mit der **rechten Maustaste auf "**Genehmigen"**. Alle Anforderungen werden auf Grün geändert. Jetzt können Sie auf **"Abgeschlossen"** klicken.
12. Auf der **Registerkarte "Vollständige Pullanforderung** " werden mehrere Optionen vor Abschluss des Seriendrucks angezeigt:
    - **Seriendrucktyp**: 4 Seriendrucktypen werden angeboten, Sie können sie [hier](https://learn.microsoft.com/azure/devops/repos/git/complete-pull-requests?view=azure-devops&tabs=browser#complete-a-pull-request) überprüfen oder die angegebenen Animationen beobachten. Zusammenführen (kein schneller Vorlauf).
    - **Optionen nach Abschluss des Vorgangs**:
        - Überprüfen Sie **die zugeordnete Arbeitsaufgabe abgeschlossen...**. Es wird zugeordneter PBI in den **Zustand "Fertig"** verschoben.

13. Klicken Sie auf **"Seriendruck abschließen".**

#### Aufgabe 7: Anwenden von Tags

Das Produktteam hat beschlossen, dass die aktuelle Version der Website als v1.1.0-Beta veröffentlicht werden soll.

1. Wählen Sie im vertikalen Navigationsbereich des Azure DevOps-Portals im **Abschnitt "Repos**" die Option "Kategorien **" aus**.
2. Klicken Sie im **Bereich "Kategorien** " auf " **Neues Tag"**.
3. Geben **Sie im Feld "Name" im Feld "Name **" in der **** Dropdown Standard ****** liste "Basierend auf** der Dropdownliste**" in das **Textfeld **"Name**" v1.1.0** ein**, und klicken Sie auf "**Erstellen**".

    > **Hinweis**: Sie haben das Repository jetzt in dieser Version markiert (der neueste Commit wird mit dem Tag verknüpft). Sie können Commits aus verschiedenen Gründen markieren, und Azure DevOps bietet die Flexibilität, sie zu bearbeiten und zu löschen sowie deren Berechtigungen zu verwalten.

## Überprüfung

In dieser Übung haben Sie das Azure DevOps-Portal zum Verwalten von Verzweigungen und Repositorys verwendet.
