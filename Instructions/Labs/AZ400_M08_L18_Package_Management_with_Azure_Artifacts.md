---
lab:
  title: Paketverwaltung mit Azure Artifacts
  module: 'Module 08: Design and implement a dependency management strategy'
---

# Paketverwaltung mit Azure Artifacts

# Lab-Handbuch für Kursteilnehmer

## Labanforderungen

- Für dieses Lab ist **Microsoft Edge** oder ein von [Azure DevOps unterstützter Browser](https://docs.microsoft.com/en-us/azure/devops/server/compatibility?view=azure-devops#web-portal-supported-browsers) erforderlich.

- **Einrichten einer Azure DevOps-Organisation:** Wenn Sie noch keine Azure DevOps-Organisation haben, die Sie für dieses Lab verwenden können, erstellen Sie eine, indem Sie den Anweisungen unter [AZ-400 Lab-Voraussetzungen](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/AZ400_M00_Validate_lab_environment.html) folgen.

- **Einrichten des EShopOnWeb-Beispielprojekts:** Wenn Sie noch kein EShopOnWeb-Beispielprojekt haben, das Sie für diese Übung verwenden können, erstellen Sie eines, indem Sie die Anweisungen unter [AZ-400 Lab-Voraussetzungen](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/AZ400_M00_Validate_lab_environment.html) befolgen.

- Visual Studio 2022 Community Edition ist auf der [Seite „Visual Studio Downloads“](https://visualstudio.microsoft.com/downloads/) verfügbar. Die Installation von Visual Studio 2022 sollte **ASP<nolink>.NET und Webentwicklung**, **Azure-Entwicklung** und **plattformübergreifende .NET Core-Entwicklung** umfassen.

## Übersicht über das Labor

Azure Artifacts vereinfachen die Ermittlung, Installation und Veröffentlichung von NuGet-, npm- und Maven-Paketen in Azure DevOps. Sie sind tief in andere Azure DevOps-Features wie Build integriert und machen die Paketverwaltung so zu einem nahtlosen Teil Ihrer vorhandenen Workflows.

## Ziele

In diesem Lab lernen Sie Folgendes:

- Erstellen eines Feeds und Herstellen einer Verbindung.
- Erstellen und Veröffentlichen eines NuGet-Pakets.
- Importieren eines NuGet-Pakets.
- Aktualisieren eines NuGet-Pakets.

## Geschätzte Zeit: 40 Minuten

## Anweisungen

### Übung 0: Konfigurieren der Voraussetzungen für das Lab

In dieser Übung möchten wir Sie daran erinnern, dass Sie die Lab-Voraussetzungen validieren müssen, dass Sie eine Azure DevOps-Organisation bereit haben und dass Sie das EShopOnWeb-Projekt erstellt haben. Weitere Einzelheiten finden Sie in den obigen Anweisungen.

#### Aufgabe 1: Konfigurieren der EShopOnWeb-Lösung in Visual Studio

In dieser Aufgabe werden Sie Visual Studio konfigurieren, um sich auf das Lab vorzubereiten.

1. Stellen Sie sicher, dass Sie das Teamprojekt **EShopOnWeb** im Azure DevOps-Portal anzeigen.

    > **Hinweis**: Sie können direkt auf die Projektseite zugreifen, indem Sie zur URL [https://dev.azure.com/`<your-Azure-DevOps-account-name>`/EShopOnWeb](https://dev.azure.com/`<your-Azure-DevOps-account-name>`/EShopOnWeb) navigieren, wobei der Platzhalter `<your-Azure-DevOps-account-name>` den Namen Ihrer Azure DevOps-Organisation darstellt.

1. Klicken Sie im vertikalen Menü auf der linken Seite des Bereichs **EShopOnWeb** auf **Repos**.
1. Klicken Sie im Bereich **Dateien** auf **Klonen**, wählen Sie den Dropdownpfeil neben **In VS Code klonen** aus, und wählen Sie im Dropdownmenü **Visual Studio** aus.
1. Wenn Sie gefragt werden, ob Sie fortfahren möchten, klicken Sie auf **Öffnen**.
1. Wenn Sie dazu aufgefordert werden, melden Sie sich mit dem Benutzerkonto an, das Sie zum Einrichten Ihrer Azure DevOps-Organisation verwendet haben.
1. Akzeptieren Sie in der Visual Studio-Oberfläche im Popupfenster **Azure DevOps** den standardmäßigen lokalen Pfad (C:\EShopOnWeb) und klicken Sie auf **Klonen**. Dadurch wird das Projekt automatisch in Visual Studio importiert.
1. Lassen Sie das Visual Studio-Fenster zur Verwendung in Ihrem Lab geöffnet.

### Übung 1: Arbeiten mit Azure Artifacts.

In dieser Übung erfahren Sie, wie Sie mit Azure Artifacts arbeiten, indem Sie die folgenden Schritte ausführen:

- Einen Feed erstellen und verbinden.
- Ein NuGet-Paket erstellen und veröffentlichen.
- Ein NuGet-Pakets importieren.
- Ein NuGet-Pakets aktualisieren.

#### Aufgabe 1: Erstellen eines Feeds und Verbinden mit einem Feed

In dieser Aufgabe erstellen Sie einen Feed und stellen eine Verbindung dazu her.

1. Wählen Sie im Webbrowserfenster, das Ihre Projekteinstellungen im Azure DevOps-Portal anzeigt, im vertikalen Navigationsbereich **Artefakte** aus.
1. Wenn der Hub **Artefakte** angezeigt wird, klicken Sie oben im Bereich auf **+ Feed erstellen**.

    > **Hinweis**: Bei diesem Feed handelt es sich um eine Sammlung von NuGet-Paketen, die den Benutzer*innen innerhalb der Organisation zur Verfügung stehen und neben dem öffentlichen NuGet-Feed als Peer fungieren. Das Szenario in dieser Übung konzentriert sich auf den Arbeitsablauf für die Verwendung von Azure Artifacts, sodass die tatsächlichen Architektur- und Entwicklungsentscheidungen rein illustrativ sind.  Dieser Feed wird gemeinsame Funktionen enthalten, die von allen Projekten in dieser Organisation gemeinsam genutzt werden können.

1. Geben Sie im Bereich **Neuen Feed erstellen** in das Textfeld **Name** den Text **EShopOnWebShared** ein, wählen Sie im Bereich **Umfang** die Option **Organisation** aus, belassen Sie die anderen Einstellungen auf den Standardwerten und klicken Sie auf **Erstellen**.

    > **Hinweis**: Alle Benutzer*innen, die eine Verbindung zu diesem NuGet-Feed herstellen möchten, müssen ihre Umgebung konfigurieren.

1. Wenn Sie zurück auf dem Hub **Artefakte** sind, klicken Sie auf **Mit Feed verbinden**.
1. Wählen Sie im Bereich **Mit Feed verbinden** im Abschnitt **NuGet** die Option **Visual Studio** aus und kopieren Sie im Bereich **Visual Studio** die URL **Quelle**. (https://pkgs.dev.azure.com/<Azure-DevOps-Org-Name>_packaging/EShopOnWebShared/nuget/v3/index.json)
1. Wechseln Sie zurück zum Fenster **Visual Studio**.
1. Klicken Sie im Visual Studio-Fenster auf die Menüleiste **Tools**, wählen Sie im Dropdownmenü **NuGet Package-Manager** aus und wählen Sie im Kaskadenmenü **Package Manager-Einstellungen** aus.
1. Klicken Sie im Dialogfeld **Optionen** auf **Paketquellen** und klicken Sie auf das Pluszeichen, um eine neue Paketquelle hinzuzufügen.
1. Ersetzen Sie am unteren Ende des Dialogfelds im Textfeld **Name** **Paketquelle** durch **EShopOnWebShared** und fügen im Textfeld **Quelle** die URL ein, die Sie im Azure DevOps-Portal kopiert haben.
1. Klicken Sie auf **Aktualisieren** und dann auf **OK**, um das Hinzufügen abzuschließen.

    > **Hinweis**: Visual Studio ist jetzt mit dem neuen Feed verbunden.

#### Aufgabe 2: Erstellen und Veröffentlichen eines selbst entwickelten NuGet-Pakets

In dieser Aufgabe erstellen und veröffentlichen Sie ein selbst entwickeltes benutzerdefiniertes NuGet-Paket.

1. In dem Visual Studio-Fenster, das Sie zum Konfigurieren der neuen Paketquelle verwendet haben, klicken Sie im Hauptmenü auf **Datei**, im Dropdownmenü auf **Neu** und dann im Kaskadenmenü auf **Projekt**.

    > **Hinweis**: Wir werden nun eine gemeinsame Assembly erstellen, die als NuGet-Paket veröffentlicht wird, damit andere Teams sie integrieren und auf dem neuesten Stand bleiben können, ohne direkt mit dem Projektquelltext arbeiten zu müssen.

1. Verwenden Sie auf der Seite **Zuletzt verwendete Projektvorlagen** des Bereichs **Neues Projekt erstellen** das Suchtextfeld, um die Vorlage **Klassenbibliothek** zu finden, wählen Sie die Vorlage für C# aus und klicken Sie auf **Weiter**.
1. Geben Sie auf der Seite **Klassenbibliothek** des Bereichs **Neues Projekt erstellen** die folgenden Einstellungen an und klicken Sie auf **Erstellen**:

    | Einstellung | Wert |
    | --- | --- |
    | Projektname | **EShopOnWeb.Shared** |
    | Standort | Akzeptieren Sie den Standardwert. |
    | Lösung | **Neue Lösung erstellen** |
    | Lösungsname | **EShopOnWeb.Shared** |
    
    Lassen Sie die Einstellung **Lösung und Projekt in das gleiche Verzeichnis legen** aktiviert. 

1. Klicken Sie auf Weiter. Akzeptieren Sie **.NET 6.0 (langfristiger Support)** als Framework-Option.
1. Bestätigen Sie die Projekterstellung durch Drücken der Schaltfläche **Erstellen**.
    
1. Klicken Sie in der Visual Studio-Benutzeroberfläche im Bereich **Solution Explorer** mit der rechten Maustaste auf **Class1.cs**, wählen Sie im Rechtsklick-Menü **Löschen** aus und klicken Sie auf **OK**, wenn Sie zur Bestätigung aufgefordert werden.
1. Drücken Sie **Strg+Umschalt+B** oder **Klicken Sie mit der rechten Maustaste auf das EShopOnWeb.Shared Projekt** und wählen Sie **Erstellen** aus, um das Projekt zu erstellen.

    > **Hinweis**: In der nächsten Aufgabe werden wir **NuGet.exe** verwenden, um ein NuGet-Paket direkt aus dem erstellten Projekt zu generieren, aber dafür muss das Projekt zuerst erstellt werden.

1. Wechseln Sie zu dem Webbrowser, der das Azure DevOps-Portal anzeigt.
1. Navigieren Sie zum Bereich **Mit Feed verbinden** im Abschnitt **NuGet** und wählen Sie **NuGet.exe** aus. Dadurch wird der Bereich **NuGet.exe** angezeigt.
1. Klicken Sie im Fenster **NuGet.exe** auf **Tools abrufen**.
1. Klicken Sie im Bereich **Tools abrufen** auf den Link **Neuestes NuGet herunterladen**. Dadurch wird automatisch eine weitere Browserregisterkarte geöffnet, die die Seite **Verfügbare NuGet-Verteilungsversionen** anzeigt.
1. Wählen Sie auf der Seite **Verfügbare NuGet-Verteilungsversionen** die Option **nuget.exe - recommended latest v6.x** aus und laden Sie die ausführbare Datei in den lokalen Ordner **EShopOnWeb.Shared Project** herunter (wenn Sie die Standardordner beibehalten haben, sollte dies C:\EShopOnWeb\EShopOnWeb.Shared sein).
1. Wählen Sie die Datei **nuget.exe** aus und öffnen Sie ihre Eigenschaften, indem Sie mit der rechten Maustaste auf die Datei klicken und **Eigenschaften** aus dem Kontextmenü auswählen.
1. Wählen Sie im Kontextfenster „Eigenschaften“ auf der Registerkarte **Allgemein** unter dem Abschnitt „Sicherheit“ die Option **Entsperren** aus. Bestätigen Sie mit **Anwenden** und **OK**.
1. Öffnen Sie in Ihrer Lab-Workstation das Startmenü und suchen Sie nach **Windows PowerShell**. Klicken Sie anschließend im kaskadierenden Menü auf **Windows PowerShell als Administrator öffnen**.
1. Navigieren Sie Im Fenster **Administrator: Windows PowerShell** zu dem Ordner „EShopOnWeb.Shared“, indem Sie den folgenden Befehl ausführen: 

```
cd c:\EShopOnWeb\EShopOnWeb.Shared
```
Führen Sie Folgendes aus, um eine **.nupkg**-Datei aus dem Projekt zu erstellen.

    > **Note**: This is a shortcut to package the NuGet bits for deployment. NuGet is highly customizable. To learn more, refer to the [NuGet package creation page](https://docs.microsoft.com/en-us/nuget/create-packages/overview-and-workflow).

    ```
    .\nuget.exe pack ./EShopOnWeb.Shared.csproj
    ```

    > **Note**: Disregard any warnings displayed in the **Administrator: Windows PowerShell** window.

    > **Note**: NuGet builds a minimal package based on the information it is able to identify from the project. For example, note that the name is **EShopOnWeb.Shared.1.0.0.nupkg**. That version number was retrieved from the assembly.

1. Nach der erfolgreichen Erstellung des Pakets führen Sie den folgenden Befehl aus, um das Paket im **EShopOnWebShared**-Feed zu veröffentlichen:

    > **Hinweis**: Sie müssen einen **API-Schlüssel** angeben, der eine beliebige nicht-leere Zeichenkette sein kann. Wir verwenden hier **AzDO**. Wenn Sie dazu aufgefordert werden, melden Sie sich bei Ihrer Azure DevOps-Organisation an.

    ```
    .\nuget.exe push -source "EShopOnWebShared" -ApiKey AzDO EShopOnWeb.Shared.1.0.0.nupkg
    ```
1. Warten Sie auf die Bestätigung des erfolgreichen Paket-Push-Vorgangs.
1. Wechseln Sie zum Webbrowserfenster, das das Azure DevOps-Portal anzeigt, und wählen Sie im vertikalen Navigationsbereich **Artefakte** aus.
1. Klicken Sie im Hub-Bereich **Artefakte** auf die Dropdownliste in der oberen linken Ecke und wählen Sie in der Liste der Feeds den Eintrag **EShopOnWebShared** aus.

    > **Hinweis**: Der Feed ** EShopOnWebShared** sollte das neu veröffentlichte NuGet-Paket enthalten.

1. Klicken Sie auf das NuGet-Paket, um seine Details anzuzeigen.

#### Aufgabe 3: Importieren eines Open-Source-NuGet-Pakets in den Azure DevOps-Paket-Feed

Warum verwenden Sie neben der Entwicklung eigener Pakete nicht auch die Open-Source-Paketbibliothek DotNet von Nuget (https://www.nuget.org)? Bei mehreren Millionen verfügbaren Paketen findet sich immer etwas Nützliches für Ihre Anwendung.

In dieser Aufgabe werden wir ein allgemeines „Hello World“-Beispielpaket verwenden, Sie können jedoch denselben Ansatz für andere Pakete in der Bibliothek wählen.

1. Führen Sie in demselben PowerShell-Fenster den folgenden **nuget**-Befehl aus, um das Beispielpaket zu installieren:

```
.\nuget install HelloWorld -ExcludeVersion
```

1. Überprüfen Sie die Ausgabe des Installationsvorgangs. In der ersten Zeile werden die verschiedenen Feeds angezeigt, mit denen versucht wird, das Paket herunterzuladen:
```
Feeds used:
  https://api.nuget.org/v3/index.json
  https://pkgs.dev.azure.com/<AZURE_DEVOPS_ORGANIZATION>/eShopOnWeb/_packaging/EShopOnWebPFeed/nuget/v3/index.json
```
1. Als Nächstes wird eine zusätzliche Ausgabe zum eigentlichen Installationsvorgang angezeigt.

```
Installing package 'Helloworld' to 'C:\eShopOnWeb\EShopOnWeb.Shared'.
  GET https://api.nuget.org/v3/registration5-gz-semver2/helloworld/index.json
  OK https://api.nuget.org/v3/registration5-gz-semver2/helloworld/index.json 114ms
MSBuild auto-detection: using msbuild version '17.5.0.10706' from 'C:\Program Files\Microsoft Visual Studio\2022\Professional\MSBuild\Current\bin'.
  GET https://pkgs.dev.azure.com/pdtdemoworld/7dc3351f-bb0c-42ba-b3c9-43dab8e0dc49/_packaging/188ec0d5-ff93-4eb7-b9d3-293fbf759f06/nuget/v3/registrations2-semver2/helloworld/index.json
  OK https://pkgs.dev.azure.com/pdtdemoworld/7dc3351f-bb0c-42ba-b3c9-43dab8e0dc49/_packaging/188ec0d5-ff93-4eb7-b9d3-293fbf759f06/nuget/v3/registrations2-semver2/helloworld/index.json 698ms

Attempting to gather dependency information for package 'Helloworld.1.3.0.17' with respect to project 'C:\eShopOnWeb\EShopOnWeb.Shared', targeting 'Any,Version=v0.0'
Gathering dependency information took 21 ms
Attempting to resolve dependencies for package 'Helloworld.1.3.0.17' with DependencyBehavior 'Lowest'
Resolving dependency information took 0 ms
Resolving actions to install package 'Helloworld.1.3.0.17'
Resolved actions to install package 'Helloworld.1.3.0.17'
Retrieving package 'HelloWorld 1.3.0.17' from 'nuget.org'.
  GET https://api.nuget.org/v3-flatcontainer/helloworld/1.3.0.17/helloworld.1.3.0.17.nupkg
  OK https://api.nuget.org/v3-flatcontainer/helloworld/1.3.0.17/helloworld.1.3.0.17.nupkg 133ms
Installed HelloWorld 1.3.0.17 from https://api.nuget.org/v3/index.json with content hash 1Pbk5sGihV5JCE5hPLC0DirUypeW8hwSzfhD0x0InqpLRSvTEas7sPCVSylJ/KBzoxbGt2Iapg72WPbEYxLX9g==.
Adding package 'HelloWorld.1.3.0.17' to folder 'C:\eShopOnWeb\EShopOnWeb.Shared'
Added package 'HelloWorld.1.3.0.17' to folder 'C:\eShopOnWeb\EShopOnWeb.Shared'
Successfully installed 'HelloWorld 1.3.0.17' to C:\eShopOnWeb\EShopOnWeb.Shared
Executing nuget actions took 686 ms
```
1. Das HelloWorld-Paket wurde in einem Unterordner **HelloWorld** unter dem Ordner „EShopOnWeb.Shared“ installiert. Navigieren Sie im Visual Studio **Solution Explorer** zum Projekt **EShopOnWeb.Shared** und beachten Sie den Unterordner **HelloWorld**. Klicken Sie auf den kleinen Pfeil links neben dem Unterordner, um die Ordner- und Dateiliste zu öffnen. 
1. Beachten Sie den Unterordner **lib**, der eine Signaturdatei **signature.p7s** enthält, die den Ursprung des Pakets beweist. Als Nächstes sehen wir uns die Paketdatei  **HelloWorld.nupkg** selbst an. 

#### Aufgabe 4: Hochladen des Open-Source-NuGet-Pakets in Azure Artifacts
Betrachten wir dieses Paket als ein „genehmigtes“ Paket, das unser DevOps-Team wiederverwenden kann, indem es in den zuvor erstellten Paketfeed von Azure Artifacts hochgeladen wird.

1. Führen Sie im PowerShell-Fenster den folgenden Befehl aus:

 ```
.\nuget.exe push -source "EShopOnWebShared" -ApiKey AzDO c:\EShopOnWeb\EShopOnWeb.Shared\HelloWorld\HelloWorld.nupkg
```
    > **Hinweis**: Dies führt zu einer Fehlermeldung:

```
Response status code does not indicate success: 409 (Conflict - 'HelloWorld 1.3.0.17' cannot be published to the feed because it exists in at least one of the feed's upstream sources. Publishing this copy would prevent you from using 'HelloWorld 1.3.0.17' from 'NuGet Gallery'. For more information, see https://go.microsoft.com/fwlink/?linkid=864880 (DevOps Activity ID: AE08BE89-C2FA-4FF7-89B7-90805C88972C)).
```

Bei der Erstellung des Azure DevOps Artifacts Package Feeds wurden **Upstream-Quellen**, wie z.B. nuget.org im dotnet-Beispiel, berücksichtigt. Nichts hindert Ihr DevOps-Team jedoch daran, einen **„rein internen“** Paketfeed zu erstellen.

1. Navigieren Sie zum Azure DevOps Portal, gehen Sie zu **Artifacts** und wählen Sie den Feed **EShopOnWebShared** aus. 
1. Klicken Sie auf **Upstreamquellen durchsuchen**
1. Wählen Sie Fenster **Zu Upstreampaket wechseln** die Option **Nuget** als Pakettyp aus, und geben Sie **HelloWorld** in das Suchfeld ein.
1. Bestätigen Sie durch Drücken der Schaltfläche **Suchen**.
1. Das Ergebnis ist eine Liste aller HelloWorld-Pakete mit den verschiedenen verfügbaren Versionen.
1. Klicken Sie auf die **linke Pfeiltaste**, um zum **EShopOnWebShared**-Feed zurückzukehren.   
1. Klicken Sie auf das Zahnrad, um die **Feedeinstellungen** zu öffnen. Wählen Sie auf der Seite „Feedeinstellungen“ die Option **Upstreamquellen** aus. 
1. Beachten Sie die unterschiedlichen Upstream-Paketmanager für die verschiedenen Entwicklungssprachen. Wählen Sie **Nuget.org** aus der Liste aus. Wählen Sie **Löschen** und anschließend **Speichern** aus.

1. Mit diesen gespeicherten Änderungen ist es möglich, das **HelloWorld**-Paket mit der Datei „Nuget.exe“ aus dem PowerShell-Fenster hochzuladen, indem Sie den folgenden Befehl erneut aufrufen:

```
 .\nuget.exe push -source "EShopOnWebShared" -ApiKey AzDO c:\EShopOnWeb\EShopOnWeb.Shared\HelloWorld\HelloWorld.nupkg
```

> **Hinweis**: Dies sollte jetzt zu einem erfolgreichen Upload führen 

```
Pushing HelloWorld.nupkg to 'https://pkgs.dev.azure.com/pdtdemoworld/7dc3351f-bb0c-42ba-b3c9-43dab8e0dc49/_packaging/188ec0d5-ff93-4eb7-b9d3-293fbf759f06/nuget/v2/'...
  PUT https://pkgs.dev.azure.com/<AZUREDEVOPSORGANIZATION>/7dc3351f-bb0c-42ba-b3c9-43dab8e0dc49/_packaging/188ec0d5-ff93-4eb7-b9d3-293fbf759f06/nuget/v2/
MSBuild auto-detection: using msbuild version '17.5.0.10706' from 'C:\Program Files\Microsoft Visual Studio\2022\Professional\MSBuild\Current\bin'.
  Accepted https://pkgs.dev.azure.com/pdtdemoworld<AZUREDEVOPSORGANIZATION>/7dc3351f-bb0c-42ba-b3c9-43dab8e0dc49/_packaging/188ec0d5-ff93-4eb7-b9d3-293fbf759f06/nuget/v2/ 1645ms
Your package was pushed.
PS C:\eShopOnWeb\EShopOnWeb.Shared>
```

1. **Aktualisieren** Sie im Azure DevOps-Portal die Seite „Artifacts Package Feed“. Die Liste der Pakete zeigt sowohl das Paket **EShopOnWeb.Shared**, das vom Kunden entwickelt wurde, als auch das Paket **HelloWorld**, das aus öffentlichen Quellen stammt.
1. Klicken Sie in der Visual Studio-Lösung **EShopOnWeb.Shared** mit der rechten Maustaste auf das Projekt **EShopOnWeb.Shared**, und wählen Sie aus dem Kontextmenü die Option **NuGet-Pakete verwalten** aus. 
1. Überprüfen Sie im Fenster „NuGet-Paket-Manager“, ob die **Paketquelle** auf **EShopOnWebShared** eingestellt ist.
1. Klicken Sie auf **Durchsuchen**, und warten Sie, bis die Liste der Nuget-Pakete geladen ist.
1. Diese Liste zeigt sowohl das Paket **EShopOnWeb.Shared** an, das vom Kunden entwickelt wurde, als auch das Paket **HelloWorld**, das aus öffentlichen Quellen stammt.

## Überprüfung

In dieser Übung haben Sie anhand der folgenden Schritte gelernt, wie Sie mit Azure-Artefakten arbeiten können:

- Erstellen und Verbinden mit einem Feed.
- Erstellen und Veröffentlichen eines NuGet-Pakets.
- Importieren eines benutzerdefiniertes NuGet-Pakets.
- Importieren eines NuGet-Pakets aus einer öffentlicher Quelle.
