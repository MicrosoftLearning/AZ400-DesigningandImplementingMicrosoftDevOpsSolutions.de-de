---
lab:
  title: Paketverwaltung mit Azure Artifacts
  module: 'Module 07: Design and implement a dependency management strategy'
---

# Paketverwaltung mit Azure Artifacts

## Labanforderungen

- Für dieses Lab ist **Microsoft Edge** oder ein von [Azure DevOps unterstützter Browser](https://docs.microsoft.com/azure/devops/server/compatibility) erforderlich.

- **Einrichten einer Azure DevOps-Organisation:** Wenn Sie noch keine Azure DevOps-Organisation haben, die Sie für dieses Lab verwenden können, erstellen Sie eine, indem Sie den Anweisungen unter [AZ-400 Lab-Voraussetzungen](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/AZ400_M00_Validate_lab_environment.html) folgen.

- **Einrichten des eShopOnWeb-Beispielprojekts:** Wenn Sie noch kein eShopOnWeb-Beispielprojekt besitzen, das Sie für dieses Lab verwenden können, erstellen Sie eins, indem Sie die Anweisungen unter [AZ-400: Voraussetzungen für das Lab](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/AZ400_M00_Validate_lab_environment.html) befolgen.

- Visual Studio 2022 Community Edition ist auf der [Seite „Visual Studio Downloads“](https://visualstudio.microsoft.com/downloads/) verfügbar. Die Installation von Visual Studio 2022 sollte **ASP<nolink>.NET und Webentwicklung**, **Azure-Entwicklung** und **plattformübergreifende .NET Core-Entwicklung** umfassen.

- **.NET Core SDK:** [.NET Core SDK (2.1.400 oder höher) herunterladen und installieren](https://go.microsoft.com/fwlink/?linkid=2103972)

- **Azure Artifacts-Anmeldeinformationsanbieter:** [Anmeldeinformationsanbieter herunterladen und installieren](https://go.microsoft.com/fwlink/?linkid=2099625)

## Übersicht über das Labor

Azure Artifacts vereinfachen die Ermittlung, Installation und Veröffentlichung von NuGet-, npm- und Maven-Paketen in Azure DevOps. Sie sind tief in andere Azure DevOps-Features wie Build integriert und machen die Paketverwaltung so zu einem nahtlosen Teil Ihrer vorhandenen Workflows.

## Ziele

In diesem Lab lernen Sie Folgendes:

- Erstellen eines Feeds und Herstellen einer Verbindung.
- Erstellen und Veröffentlichen eines NuGet-Pakets.
- Importieren eines NuGet-Pakets.
- Aktualisieren eines NuGet-Pakets.

## Geschätzte Dauer: 35 Minuten

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

#### Aufgabe 3: (überspringen, wenn erledigt) Legen Sie den Mainbranch als Standardbranch fest

1. Wechseln Sie zu **Repos > Branches**.
1. Bewegen Sie den Mauszeiger auf den **Main**-Branch und klicken Sie dann rechts neben der Spalte auf die Auslassungspunkte.
1. Klicken Sie auf **Als Mainbranch festlegen**.

#### Aufgabe 4: Konfigurieren der eShopOnWeb-Lösung in Visual Studio

In dieser Aufgabe werden Sie Visual Studio konfigurieren, um sich auf das Lab vorzubereiten.

1. Stellen Sie sicher, dass Sie das **eShopOnWeb**-Teamprojekt im Azure DevOps-Portal sehen.

   > **Hinweis:** Sie können direkt auf die Projektseite zugreifen, indem Sie zur URL [https://dev.azure.com/`<your-Azure-DevOps-account-name>`/eShopOnWeb](https://dev.azure.com/`<your-Azure-DevOps-account-name>`/eShopOnWeb) navigieren, wobei der `<your-Azure-DevOps-account-name>`-Platzhalter Ihren Azure DevOps-Organisationsnamen darstellt.

1. Klicken Sie im vertikalen Menü auf der linken Seite des Bereichs **eShopOnWeb** auf **Repos**.
1. Klicken Sie im Bereich **Dateien** auf **Klonen**, wählen Sie den Dropdownpfeil neben **In VS Code klonen** aus, und wählen Sie im Dropdownmenü **Visual Studio** aus.
1. Wenn Sie gefragt werden, ob Sie fortfahren möchten, klicken Sie auf **Öffnen**.
1. Wenn Sie dazu aufgefordert werden, melden Sie sich mit dem Benutzerkonto an, das Sie zum Einrichten Ihrer Azure DevOps-Organisation verwendet haben.
1. Akzeptieren Sie in der Visual Studio-Benutzeroberfläche im Popupfenster **Azure DevOps** den lokalen Standardpfad (C:\eShopOnWeb), und klicken Sie auf **Klonen**. Dadurch wird das Projekt automatisch in Visual Studio importiert.
1. Lassen Sie das Visual Studio-Fenster zur Verwendung in Ihrem Lab geöffnet.

### Übung 1: Arbeiten mit Azure Artifacts.

In dieser Übung erfahren Sie, wie Sie mit Azure Artifacts arbeiten, indem Sie die folgenden Schritte ausführen:

- Erstellen eines Feeds und Herstellen einer Verbindung.
- Erstellen und Veröffentlichen eines NuGet-Pakets.
- Importieren eines NuGet-Pakets.
- Aktualisieren eines NuGet-Pakets.

#### Aufgabe 1: Erstellen eines Feeds und Verbinden mit einem Feed

In dieser Aufgabe erstellen Sie einen Feed und stellen eine Verbindung dazu her.

1. Wählen Sie im Webbrowserfenster, das Ihre Projekteinstellungen im Azure DevOps-Portal anzeigt, im vertikalen Navigationsbereich **Artefakte** aus.
1. Wenn der Hub **Artefakte** angezeigt wird, klicken Sie oben im Bereich auf **+ Feed erstellen**.

   > **Hinweis**: Bei diesem Feed handelt es sich um eine Sammlung von NuGet-Paketen, die den Benutzer*innen innerhalb der Organisation zur Verfügung stehen und neben dem öffentlichen NuGet-Feed als Peer fungieren. Das Szenario in dieser Übung konzentriert sich auf den Arbeitsablauf für die Verwendung von Azure Artifacts, sodass die tatsächlichen Architektur- und Entwicklungsentscheidungen rein illustrativ sind. Dieser Feed wird gemeinsame Funktionen enthalten, die von allen Projekten in dieser Organisation gemeinsam genutzt werden können.

1. Geben Sie im Bereich **Neuen Feed erstellen** in das Textfeld **Name** **`eShopOnWebShared`** ein. Wählen Sie im Abschnitt **Sichtbarkeit** die Option „Bestimmte Personen“ und im Abschnitt **Geltungsbereich** die Option **Projekt:eShopOnWeb**, belassen Sie die anderen Einstellungen auf den Standardwerten und klicken Sie auf **Erstellen**.

   > **Hinweis**: Alle Benutzer*innen, die eine Verbindung zu diesem NuGet-Feed herstellen möchten, müssen ihre Umgebung konfigurieren.

1. Wenn Sie zurück auf dem Hub **Artefakte** sind, klicken Sie auf **Mit Feed verbinden**.
1. Wählen Sie im Bereich **Mit Feed verbinden** im Abschnitt **NuGet** die Option **Visual Studio** aus und kopieren Sie im Bereich **Visual Studio** die URL **Quelle**. `https://pkgs.dev.azure.com/Azure-DevOps-Org-Name/_packaging/eShopOnWebShared/nuget/v3/index.json`
1. Wechseln Sie zurück zum Fenster **Visual Studio**.
1. Klicken Sie im Visual Studio-Fenster auf die Menüleiste **Tools**, wählen Sie im Dropdownmenü **NuGet Package-Manager** aus und wählen Sie im Kaskadenmenü **Package Manager-Einstellungen** aus.
1. Klicken Sie im Dialogfeld **Optionen** auf **Paketquellen** und klicken Sie auf das Pluszeichen, um eine neue Paketquelle hinzuzufügen.
1. Ersetzen Sie unten im Dialogfeld im Textfeld **Name** die **Paketquelle** durch **eShopOnWebShared**, und fügen Sie im Textfeld **Quelle** die URL ein, die Sie im Azure DevOps-Portal kopiert haben.
1. Klicken Sie auf **Aktualisieren** und dann auf **OK**, um das Hinzufügen abzuschließen.

   > **Hinweis**: Visual Studio ist jetzt mit dem neuen Feed verbunden.

#### Aufgabe 2: Erstellen und Veröffentlichen eines selbst entwickelten NuGet-Pakets

In dieser Aufgabe erstellen und veröffentlichen Sie ein selbst entwickeltes benutzerdefiniertes NuGet-Paket.

1. In dem Visual Studio-Fenster, das Sie zum Konfigurieren der neuen Paketquelle verwendet haben, klicken Sie im Hauptmenü auf **Datei**, im Dropdownmenü auf **Neu** und dann im Kaskadenmenü auf **Projekt**.

   > **Hinweis**: Wir werden nun eine gemeinsame Assembly erstellen, die als NuGet-Paket veröffentlicht wird, damit andere Teams sie integrieren und auf dem neuesten Stand bleiben können, ohne direkt mit dem Projektquelltext arbeiten zu müssen.

1. Verwenden Sie auf der Seite **Zuletzt verwendete Projektvorlagen** des Bereichs **Neues Projekt erstellen** das Suchtextfeld, um die Vorlage **Klassenbibliothek** zu finden, wählen Sie die Vorlage für C# aus und klicken Sie auf **Weiter**.
1. Geben Sie auf der Seite **Klassenbibliothek** des Bereichs **Neues Projekt erstellen** die folgenden Einstellungen an und klicken Sie auf **Erstellen**:

   | Einstellung       | Wert                    |
   | ------------- | ------------------------ |
   | Projektname  | **eShopOnWeb.Shared**    |
   | Standort      | Akzeptieren Sie den Standardwert. |
   | Lösung      | **Neue Lösung erstellen**  |
   | Lösungsname | **eShopOnWeb.Shared**    |

   Aktivieren Sie das Kontrollkästchen **Platzieren Sie die Projektmappe und das Projekt im selben Verzeichnis**.

1. Klicken Sie auf Weiter. Akzeptieren Sie **.NET 8** als Frameworkoption.
1. Bestätigen Sie die Projekterstellung durch Drücken der Schaltfläche **Erstellen**.
1. Klicken Sie in der Visual Studio-Benutzeroberfläche im Bereich **Solution Explorer** mit der rechten Maustaste auf **Class1.cs**, wählen Sie im Rechtsklick-Menü **Löschen** aus und klicken Sie auf **OK**, wenn Sie zur Bestätigung aufgefordert werden.
1. Drücken Sie **Strg+Umschalt+B** oder **Klicken Sie mit der rechten Maustaste auf das EShopOnWeb.Shared Projekt** und wählen Sie **Erstellen** aus, um das Projekt zu erstellen.
1. Öffnen Sie in Ihrer Lab-Workstation das Startmenü und suchen Sie nach **Windows PowerShell**. Klicken Sie anschließend im kaskadierenden Menü auf **Windows PowerShell als Administrator öffnen**.
1. Klicken Sie im Fenster **Administrator: Windows PowerShell**, navigieren Sie zum Ordner „eShopOnWeb.Shared“, indem Sie den folgenden Befehl ausführen:

   ```powershell
   cd c:\eShopOnWeb\eShopOnWeb.Shared
   ```

   > **Hinweis:** Der Ordner **eShopOnWeb.Shared** ist der Speicherort der Datei **eShopOnWeb.Shared.csproj**. Wenn Sie einen anderen Ort oder Projektnamen gewählt haben, navigieren Sie stattdessen zu diesem Ort.

1. Führen Sie Folgendes aus, um eine **.nupkg**-Datei aus dem Projekt zu erstellen (ändern Sie den Wert des Platzhalters `XXXXXX` mit einer eindeutigen Zeichenfolge).

   ```powershell
   dotnet pack .\eShopOnWeb.Shared.csproj -p:PackageId=eShopOnWeb-XXXXX.Shared
   ```

   > **Hinweis:** Der Befehl **dotnet pack** kompiliert das Projekt und erstellt ein NuGet-Paket im Ordner **bin\Release**. Wenn Sie keinen **Freigabe**-Ordner haben, können Sie stattdessen den **Debug**-Ordner verwenden.

   > **Hinweis**: Ignorieren Sie alle Warnungen, die im Fenster **Administrator: Windows PowerShell** angezeigt werden.

   > **Hinweis**: Der Befehl „dotnet pack“ kompiliert ein minimales Paket basierend auf den Informationen, die er aus dem Projekt ermitteln kann. Mit dem Argument `-p:PackageId=eShopOnWeb-XXXXXX.Shared` können Sie stattdessen ein Paket mit einem bestimmten Namen erstellen, indem Sie den im Projekt enthaltenen Namen verwenden. Wenn Sie beispielsweise die Zeichenfolge `12345` durch den Platzhalter `XXXXXX` ersetzen, lautet der Name des Pakets **eShopOnWeb-12345.Shared.1.0.0.nupkg**. Die Versionsnummer wurde aus der Assembly abgerufen.

1. Führen Sie im PowerShell-Fenster den folgenden Befehl aus, um den Ordner **bin\Release** zu öffnen:

   ```powershell
   cd .\bin\Release
   ```

1. Führen Sie Folgendes aus, um das Paket im **eShopOnWebShared**-Feed zu veröffentlichen. Ersetzen Sie die Quelle durch die URL, die Sie zuvor aus der Visual Studio **Quellen**-URL `https://pkgs.dev.azure.com/Azure-DevOps-Org-Name/_packaging/eShopOnWebShared/nuget/v3/index.json` kopiert haben.

   ```powershell
   dotnet nuget push --source "https://pkgs.dev.azure.com/Azure-DevOps-Org-Name/_packaging/eShopOnWebShared/nuget/v3/index.json" --api-key az "eShopOnWeb.Shared.1.0.0.nupkg"
   ```

   > **Wichtig:** Sie müssen den Anmeldeinformationsanbieter für Ihr Betriebssystem installieren, um sich bei Azure DevOps authentifizieren zu können. Die Installationsanweisungen finden Sie unter [Azure Artifacts-Anmeldeinformationsanbieter](https://go.microsoft.com/fwlink/?linkid=2099625). Sie können ihn installieren, indem Sie den folgenden Befehl im PowerShell-Fenster ausführen: `iex "& { $(irm https://aka.ms/install-artifacts-credprovider.ps1) } -AddNetfx"`.

   > **Hinweis**: Sie müssen einen **API-Schlüssel** angeben, der eine beliebige nicht-leere Zeichenkette sein kann. Wir verwenden hier **az**. Wenn Sie dazu aufgefordert werden, melden Sie sich bei Ihrer Azure DevOps-Organisation an.

   > **Hinweis**: Wenn die Eingabeaufforderung nicht angezeigt wird oder Sie die folgende Warnung erhalten: **Warnung: Der Plugin-Anmeldeinformationsanbieter konnte keine Anmeldeinformationen abrufen. Die Authentifizierung erfordert möglicherweise eine manuelle Aktion. Führen Sie den Befehl erneut mit --interactive für `dotnet`, /p:NuGetInteractive=true für MSBuild aus oder entfernen Sie den Schalter „-NonInteractive“ für NuGet"**, können Sie den Parameter **--interactive** zum Befehl hinzufügen.

1. Warten Sie auf die Bestätigung des erfolgreichen Paket-Push-Vorgangs.
1. Wechseln Sie zum Webbrowserfenster, das das Azure DevOps-Portal anzeigt, und wählen Sie im vertikalen Navigationsbereich **Artefakte** aus.
1. Klicken Sie im Hubbereich **Artefakte** auf die Dropdownliste in der oberen linken Ecke, und wählen Sie in der Liste der Feeds den Eintrag **eShopOnWebShared** aus.

   > **Hinweis:** Der **eShopOnWebShared**-Feed sollte das neu veröffentlichte NuGet-Paket enthalten.

1. Klicken Sie auf das NuGet-Paket, um seine Details anzuzeigen.

#### Aufgabe 3: Importieren eines Open-Source-NuGet-Pakets in den Azure DevOps-Paket-Feed

Sie können Pakete nicht nur selbst entwickeln, sondern auch die Open-Source-Paketbibliothek für NuGet (<https://www.nuget.org>) und .NET verwenden. Bei mehreren Millionen verfügbaren Paketen findet sich immer etwas Nützliches für Ihre Anwendung.

In dieser Aufgabe verwenden wir ein generisches „Newtonsoft.Json“-Beispielpaket, Sie können jedoch denselben Ansatz für andere Pakete in der Bibliothek verwenden.

1. Navigieren Sie im selben PowerShell-Fenster, das Sie in der vorherigen Aufgabe zum Pushen des neuen Pakets verwendet haben, zurück zum Ordner **eShopOnWeb.Shared** (`cd..`), und führen Sie den folgenden **dotnet**-Befehl aus, um das Beispielpaket zu installieren:

   ```powershell
   dotnet add package Newtonsoft.Json
   ```

1. Überprüfen Sie die Ausgabe des Installationsvorgangs. Es zeigt die verschiedenen Feeds an, für die es versucht, das Paket herunterzuladen:

   ```powershell
   Feeds used:
     https://api.nuget.org/v3/registration5-gz-semver2/newtonsoft.json/index.json
     https://pkgs.dev.azure.com/<AZURE_DEVOPS_ORGANIZATION>/eShopOnWeb/_packaging/eShopOnWebShared/nuget/v3/index.json
   ```

1. Als Nächstes wird eine zusätzliche Ausgabe zum eigentlichen Installationsvorgang angezeigt.

   ```powershell
   Determining projects to restore...
   Writing C:\Users\AppData\Local\Temp\tmpxnq5ql.tmp
   info : X.509 certificate chain validation will use the default trust store selected by .NET for code signing.
   info : X.509 certificate chain validation will use the default trust store selected by .NET for timestamping.
   info : Adding PackageReference for package 'Newtonsoft.Json' into project 'c:\eShopOnWeb\eShopOnWeb.Shared\eShopOnWeb.Shared.csproj'.
   info :   GET https://api.nuget.org/v3/registration5-gz-semver2/newtonsoft.json/index.json
   info :   OK https://api.nuget.org/v3/registration5-gz-semver2/newtonsoft.json/index.json 124ms
   info : Restoring packages for c:\eShopOnWeb\eShopOnWeb.Shared\eShopOnWeb.Shared.csproj...
   info :   GET https://api.nuget.org/v3/vulnerabilities/index.json
   info :   OK https://api.nuget.org/v3/vulnerabilities/index.json 84ms
   info :   GET https://api.nuget.org/v3-vulnerabilities/2024.02.15.23.23.24/vulnerability.base.json
   info :   GET https://api.nuget.org/v3-vulnerabilities/2024.02.15.23.23.24/2024.02.17.11.23.35/vulnerability.update.json
   info :   OK https://api.nuget.org/v3-vulnerabilities/2024.02.15.23.23.24/vulnerability.base.json 14ms
   info :   OK https://api.nuget.org/v3-vulnerabilities/2024.02.15.23.23.24/2024.02.17.11.23.35/vulnerability.update.json 30ms
   info : Package 'Newtonsoft.Json' is compatible with all the specified frameworks in project 'c:\eShopOnWeb\eShopOnWeb.Shared\eShopOnWeb.Shared.csproj'.
   info : PackageReference for package 'Newtonsoft.Json' version '13.0.3' added to file 'c:\eShopOnWeb\eShopOnWeb.Shared\eShopOnWeb.Shared.csproj'.
   info : Writing assets file to disk. Path: c:\eShopOnWeb\eShopOnWeb.Shared\obj\project.assets.json
   log  : Restored c:\eShopOnWeb\eShopOnWeb.Shared\eShopOnWeb.Shared.csproj (in 294 ms).
   ```

1. Das Newtonsoft.Json-Paket wurde unter „Pakete“ als **Newtonsoft.Json** installiert. Navigieren Sie im **Projektmappen-Explorer** von Visual Studio zum **eShopOnWeb.Shared**-Projekt, erweitern Sie die Abhängigkeiten, und Sie sehen **Newtonsoft.Json** unter den Paketen. Klicken Sie auf den kleinen Pfeil links neben den Paketen, um den Ordner- und Dateiliste zu öffnen.

Als Sie den Azure DevOps Artifacts Package Feed erstellt haben, war es beabsichtigt, **Upstreamquellen** wie nuget.org im dotnet-Beispiel zuzulassen, das das Newtonsoft.Json-Paket von dem Ort einbringt, an dem das Paket gehostet wird. Dies ist eine gängige Praxis, um doppelte Pakete zu vermeiden und sicherzustellen, dass immer die neueste Version verwendet wird.

1. **Aktualisieren** Sie im Azure DevOps-Portal die Seite „Artifacts Package Feed“. Die Liste der Pakete enthält sowohl das benutzerdefinierte **eShopOnWeb.Shared**-Paket als auch das öffentliche **Newtonsoft.Json**-Paket.
1. Klicken Sie in der Visual Studio-Projektmappe **eShopOnWeb.Shared** mit der rechten Maustaste auf das **eShopOnWeb.Shared**-Projekt, und wählen Sie **NuGet-Pakete verwalten** aus dem Kontextmenü aus.
1. Überprüfen Sie im Fenster „NuGet-Paket-Manager“, ob die **Paketquelle** auf **eShopOnWebShared** festgelegt ist.
1. Klicken Sie auf **Durchsuchen** und warten Sie, bis die Liste der NuGet-Pakete geladen ist.
1. In dieser Liste werden auch das benutzerdefinierte **eShopOnWeb.Shared**-Paket und das öffentliche **Newtonsoft.Json**-Paket angezeigt.

## Überprüfung

In dieser Übung haben Sie anhand der folgenden Schritte gelernt, wie Sie mit Azure-Artefakten arbeiten können:

- Erstellen und Verbinden mit einem Feed.
- Erstellen und Veröffentlichen eines NuGet-Pakets.
- Importieren eines benutzerdefiniertes NuGet-Pakets.
