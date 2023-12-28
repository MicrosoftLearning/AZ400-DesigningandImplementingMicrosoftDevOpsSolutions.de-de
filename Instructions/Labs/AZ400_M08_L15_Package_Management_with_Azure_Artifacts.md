---
lab:
  title: Paketverwaltung mit Azure Artifacts
  module: 'Module 08: Design and implement a dependency management strategy'
---

# Paketverwaltung mit Azure Artifacts

## Lab-Handbuch für Kursteilnehmer

## Labanforderungen

- Für dieses Lab ist **Microsoft Edge** oder ein [von Azure DevOps unterstützter Browser](https://docs.microsoft.com/azure/devops/server/compatibility) erforderlich.

- **Einrichten einer Azure DevOps-Organisation**: Wenn Sie nicht bereits eine Azure DevOps-Organisation haben, die Sie für dieses Lab verwenden können, müssen Sie diese erstellen, indem Sie die unter [Erstellen einer Organisation oder Projektsammlung](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/AZ400_M00_Validate_lab_environment.html) beschriebenen Anweisungen befolgen.

- **Richten Sie das Beispiel-EShopOnWeb-Projekt ein:** Wenn Sie noch nicht über das Beispiel-EShopOnWeb-Projekt verfügen, das Sie für dieses Labor verwenden können, erstellen Sie ein Projekt, indem Sie die anweisungen befolgen, die unter [AZ-400 Lab Prerequisites](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/AZ400_M00_Validate_lab_environment.html) verfügbar sind.

- Visual Studio 2022 Community Edition steht auf der [Visual Studio-Downloadseite](https://visualstudio.microsoft.com/downloads/) zur Verfügung. Die Visual Studio 2022-Installation sollte die Workloads **ASP.NET und Webentwicklung<nolink>, **Azure-Entwicklung** und **plattformübergreifende .NET Core-Entwicklung umfassen.

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

### Übung 0: Konfigurieren der Voraussetzungen für das Lab.

In dieser Übung möchten wir Sie daran erinnern, die Lab-Voraussetzungen zu überprüfen, sowohl eine Azure DevOps-Organisation bereit zu haben als auch das EShopOnWeb-Projekt erstellt zu haben. Weitere Informationen finden Sie in diesen Anweisungen.

#### Aufgabe 1: Konfigurieren der EShopOnWeb-Lösung in Visual Studio

In dieser Aufgabe konfigurieren Sie Visual Studio für die Vorbereitung auf die Übung.

1. Stellen Sie sicher, dass Sie das **EShopOnWeb-Teamprojekt** im Azure DevOps-Portal anzeigen.

    > **Hinweis**: Sie können direkt auf die Projektseite zugreifen, indem Sie zur [https://dev.azure.com/`<your-Azure-DevOps-account-name>`/EShopOnWeb-URL](https://dev.azure.com/`<your-Azure-DevOps-account-name>`/EShopOnWeb) navigieren, in der der `<your-Azure-DevOps-account-name>` Platzhalter Ihren Azure DevOps-Organisationsnamen darstellt.

2. Klicken Sie im vertikalen Menü auf der linken Seite des **Bereichs "EShopOnWeb** " auf **"Neu verfassen"**.
3. Klicken Sie im **Bereich "Dateien** " auf " **Klonen**", wählen Sie den Dropdownpfeil neben " **Klonen" in VS Code** aus, und wählen Sie **im Dropdownmenü Visual Studio** aus.
4. Wenn Sie gefragt werden, ob der Vorgang fortgesetzt werden soll, klicken Sie auf " **Öffnen**".
5. Melden Sie sich mit dem Benutzerkonto an, das Sie in Ihrer Azure DevOps-Organisation () verwenden möchten.
6. Übernehmen Sie in der Visual Studio-Schnittstelle im **Azure DevOps-Popupfenster** den lokalen Standardpfad (C:\EShopOnWeb), und klicken Sie auf Klonen****. Dadurch wird das Projekt automatisch in Visual Studio importiert.
7. Lassen Sie das Visual Studio-Fenster für die Verwendung in Ihrer Übung geöffnet.

### Übung 1: Arbeiten mit Azure Artifacts.

In dieser Übung erfahren Sie, wie Sie mit Azure Artifacts arbeiten, indem Sie die folgenden Schritte ausführen:

- Erstellen eines Feeds und Herstellen einer Verbindung.
- Erstellen und Veröffentlichen eines NuGet-Pakets.
- Importieren eines NuGet-Pakets.
- Aktualisieren eines NuGet-Pakets.

#### Aufgabe 1: Erstellen und Herstellen einer Verbindung mit einem Feed

In dieser Aufgabe erstellen und konfigurieren Sie Benutzer.

1. Wählen Sie im Webbrowserfenster, in dem Ihre Projekteinstellungen im Azure DevOps-Portal angezeigt werden, im vertikalen Navigationsbereich Artefakte** aus**.
2. Wenn der **Artefakthub** angezeigt wird, klicken Sie oben im Bereich auf **+Feed** erstellen.

    > **Hinweis**: Dieser Feed ist eine Sammlung von NuGet-Paketen, die Für Benutzer innerhalb der Organisation verfügbar sind und sich neben dem öffentlichen NuGet-Feed als Peer befinden. Das Szenario in dieser Übung konzentriert sich auf den Workflow für die Verwendung von Azure Artifacts, sodass die tatsächlichen Architektur- und Entwicklungsentscheidungen rein illustrativ sind.  Dieser Feed enthält allgemeine Funktionen, die für projekte in dieser Organisation freigegeben werden können.

3. Geben Sie im Feld "Neuen Feed** erstellen" im **Textfeld "Name**" "EShopOnWebShared **" im **Abschnitt "Bereich**" die **Option "Organisation**" aus, lassen Sie andere Einstellungen mit ihren Standardwerten, und klicken Sie auf "Erstellen **"**.****

    > **Hinweis**: Jeder Benutzer, der eine Verbindung mit diesem NuGet-Feed herstellen möchte, muss seine Umgebung konfigurieren.

4. Klicken Sie wieder auf dem **Artefakthub** auf **Verbinden, um zu feeden**.
5. **Wählen Sie **im Verbinden zum Feedbereich** im **Abschnitt "NuGet**" Visual Studio** aus, und kopieren Sie im **Visual Studio-Bereich** die **Quell-URL**. (https://pkgs.dev.azure.com/<Azure-DevOps-Org-Name>_packaging/EShopOnWebShared/nuget/v3/index.json)
6. Wechseln Sie zurück zum **Visual Studio Code**-Fenster.
7. Klicken Sie in der Visual Studio-Konsole auf das Menü **Extras**, klicken Sie im Dropdownmenü auf **NuGet-Paket-Manager**, und klicken Sie im hierarchischen Menü auf **Paket-Manager-Konsole**.
8. Klicken Sie im **Dialogfeld "Optionen** " auf **"Paketquellen** ", und klicken Sie auf das Pluszeichen, um eine neue Paketquelle hinzuzufügen.
9. Ersetzen **Sie unten im Dialogfeld im Textfeld "Name **" die **Paketquelle** durch **"EShopOnWebShared**", und fügen Sie im **Textfeld "Quelle**" die URL ein, die Sie im Azure DevOps-Portal kopiert haben.
10. Klicken Sie auf "Aktualisieren"**, und klicken Sie **dann auf **"OK**", um die Ergänzung abzuschließen.

    > **Hinweis**: Visual Studio ist jetzt mit dem neuen Feed verbunden.

#### Aufgabe 2: Erstellen und Veröffentlichen eines internen entwickelten NuGet-Pakets

In dieser Aufgabe erstellen und veröffentlichen Sie ein internes entwickeltes benutzerdefiniertes NuGet-Paket.

1. Klicken Sie im Visual Studio-Fenster, das Sie zum Konfigurieren der neuen Paketquelle verwendet haben, im Menü Standard auf "Datei **", klicken Sie **im Dropdownmenü auf "**Neu**", und klicken Sie dann im überlappenden Menü auf **"Projekt**".

    > **Hinweis**: Wir erstellen jetzt eine freigegebene Assembly, die als NuGet-Paket veröffentlicht wird, damit andere Teams sie integrieren und auf dem neuesten Stand bleiben können, ohne direkt mit der Projektquelle arbeiten zu müssen.

2. Verwenden Sie auf der Seite " **Zuletzt verwendete Projektvorlagen** " des **Bereichs "Neues Projekt** erstellen" das Suchtextfeld, um die **Vorlage "Klassenbibliothek** " zu suchen, wählen Sie die Vorlage für C# aus, und klicken Sie auf " **Weiter"**.
3. Geben Sie auf der **Seite "Klassenbibliothek** " im **Bereich "Neues Projekt** erstellen" die folgenden Einstellungen an, und klicken Sie auf " **Erstellen**":

    | Einstellung | Wert |
    | --- | --- |
    | Projektname | **EShopOnWeb.Shared** |
    | Standort | Akzeptieren Sie den Standardwert. |
    | Lösung | Neue Projektmappe erstellen |
    | Lösungsname | **EShopOnWeb.Shared** |

    Aktivieren Sie das Kontrollkästchen **Legen Sie die Projektmappe und das Projekt im selben Verzeichnis ab**.

4. Klicken Sie auf Weiter. Akzeptieren Sie **.NET 7.0** als Framework-Option.
5. Bestätigen Sie die Projekterstellung, indem Sie die **Schaltfläche "Erstellen** " drücken.
6. Klicken Sie in der Visual Studio-Schnittstelle im Bereich Projektmappen-Explorer mit der **rechten Maustaste auf **"Class1.cs**", klicken Sie im Kontextmenü auf "Löschen **", **und klicken Sie, wenn Sie zur Bestätigung aufgefordert werden, auf **"OK****".
7. Drücken Sie **STRG+UMSCHALT+B** , oder **klicken Sie mit der rechten Maustaste auf das EShopOnWeb.Shared-Projekt** , und wählen Sie **"Erstellen"** aus, um das Projekt zu erstellen.

    > **Hinweis**: In der nächsten Aufgabe verwenden **wir NuGet.exe** , um ein NuGet-Paket direkt aus dem erstellten Projekt zu generieren, aber das Projekt muss zuerst erstellt werden.

8. Wechseln Sie zur Webbrowserregisterkarte zurück, die das Azure DevOps-Portal anzeigt.
9. Navigieren Sie zum **Verbinden zum Feedbereich** im **Abschnitt "NuGet**", und wählen Sie **"NuGet.exe**" aus. Dadurch wird der **Bereich "NuGet.exe** " angezeigt.
10. Klicken Sie im **Bereich "NuGet.exe** " auf " **Tools abrufen"**.
11. Klicken Sie im **Bereich "Tools** abrufen" auf den **neuesten NuGet-Link** herunterladen. Dadurch wird automatisch eine andere Browserregisterkarte geöffnet, auf der die **Seite "Verfügbare NuGet-Verteilungsversionen** " angezeigt wird.
12. Wählen Sie auf der **Seite "Verfügbare NuGet-Verteilungsversionen**" "nuget.exe" aus – empfohlen, neueste v6.x** und laden Sie die ausführbare Datei in den lokalen **Ordner "EShopOnWeb.Shared Project**" herunter (Wenn Sie die Standardordnerspeicherorte beibehalten haben, sollte dies C:\EShopOnWeb\EShopOnWeb.Shared **sein).
13. Wählen Sie die **Datei nuget.exe** aus, und öffnen Sie die Eigenschaften, indem Sie mit der rechten Maustaste auf die Datei klicken und im Kontextmenü "Eigenschaften"** auswählen**.
14. Wählen Sie im Kontextfenster "Eigenschaften" auf der **Registerkarte "Allgemein** " unter dem Abschnitt "Sicherheit" die Option **"Blockierung** aufheben" aus. Bestätigen Sie, indem Sie "Übernehmen" und **"OK**"** drücken**.
15. Öffnen Sie auf Ihrer Laborarbeitsstation die Menü , und suchen Sie nach **Windows PowerShell**. Klicken Sie als Nächstes im überlappenden Menü auf **"Windows PowerShell als Administrator öffnen"**.
16. Navigieren Sie im **Fenster "Administrator: Windows PowerShell** " zum Ordner "EShopOnWeb.Shared", indem Sie den folgenden Befehl ausführen:

    ```text
    cd c:\EShopOnWeb\EShopOnWeb.Shared
    ```

    Führen Sie den folgenden Befehl im Projektstamm aus, um eine -Datei zu erstellen:

    > **Hinweis**: Dies ist eine Verknüpfung zum Packen der NuGet-Bits für die Bereitstellung. NuGet ist hochgradig anpassbar. Weitere Informationen finden Sie auf der Seite[ zum Erstellen von ](https://docs.microsoft.com/nuget/create-packages/overview-and-workflow)NuGet-Paketen.

    ```text
    .\nuget.exe pack ./EShopOnWeb.Shared.csproj
    ```

    > **Hinweis**: Ignorieren Sie alle Warnungen, die **im Fenster "Administrator: Windows PowerShell** " angezeigt werden.

    > **Hinweis**: NuGet erstellt ein minimales Paket basierend auf den Informationen, die es aus dem Projekt identifizieren kann. Beachten Sie beispielsweise, dass der Name "EShopOnWeb.Shared.1.0.0.nupkg **" lautet**. Diese Versionsnummer wurde aus der Assembly abgerufen.

17. Führen Sie nach der erfolgreichen Erstellung des Pakets Folgendes aus, um das Paket im **EShopOnWebShared-Feed** zu veröffentlichen:

    > **Hinweis**: Sie müssen einen **API-Schlüssel** bereitstellen, der eine beliebige nicht leere Zeichenfolge sein kann. Hier verwenden **wir AzDO** . Melden Sie sich bei Ihrer Azure DevOps-Organisation () an.

    ```text
    .\nuget.exe push -source "EShopOnWebShared" -ApiKey AzDO EShopOnWeb.Shared.1.0.0.nupkg
    ```

18. Warten Sie auf die Bestätigung des erfolgreichen Paket-Pushvorgangs.
19. Wechseln Sie zum Webbrowserfenster, in dem das Azure DevOps-Portal angezeigt wird, und wählen Sie im vertikalen Navigationsbereich Artefakte aus****.
20. Klicken Sie im Hubbereich " **Artefakte"** in der oberen linken Ecke auf die Dropdownliste, und wählen Sie in der Liste der Feeds den **Eintrag "EShopOnWebShared** " aus.

    > **Hinweis**: Der **EShopOnWebShared-Feed** sollte das neu veröffentlichte NuGet-Paket enthalten.

21. Klicken Sie auf das NuGet-Paket, um dessen Details anzuzeigen.

#### Aufgabe 3: Importieren eines Open Source NuGet-Pakets in den Azure DevOps-Paketfeed

Warum nicht die Open Source NuGet -Bibliothek (https://www.nuget.org) DotNet-Paketbibliothek) verwenden, um ihre eigenen Pakete zu entwickeln? Mit einigen Millionen verfügbaren Paketen gibt es immer etwas Nützliches für Ihre Anwendung.

In dieser Aufgabe verwenden wir ein generisches "Hallo Welt"-Beispielpaket, Sie können jedoch denselben Ansatz für andere Pakete in der Bibliothek verwenden.

1. Führen Sie im gleichen PowerShell-Fenster den folgenden **Nuget-Befehl** aus, um das Beispielpaket zu installieren:

    ```text
    .\nuget install HelloWorld -ExcludeVersion
    ```

2. Überprüfen Sie die Ausgabe des Installationsprozesses. In der ersten Zeile werden die verschiedenen Feeds angezeigt, die es versucht, das Paket herunterzuladen:

    ```text
    Feeds used:
      https://api.nuget.org/v3/index.json
      https://pkgs.dev.azure.com/<AZURE_DEVOPS_ORGANIZATION>/eShopOnWeb/_packaging/EShopOnWebPFeed/nuget/v3/index.json
    ```

3. Als Nächstes wird eine zusätzliche Ausgabe im Hinblick auf den tatsächlichen Installationsprozess selbst angezeigt.

    ```text
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

4. Das HelloWorld-Paket wurde in einem Unterordner **"HelloWorld**" unter dem Ordner "EShopOnWeb.Shared" installiert. Navigieren Sie aus dem Visual Studio-Projektmappen-Explorer **** zum **EShopOnWeb.Shared-Projekt**, und beachten Sie den **Unterordner "HelloWorld**". Klicken Sie links neben dem Unterordner auf den kleinen Pfeil, um den Ordner und die Dateiliste zu öffnen.
5. Beachten Sie den **Lib-Unterordner** mit einer **Signaturdatei "signature.p7s** ", die den Ursprung des Pakets nachweist. Beachten Sie als Nächstes die **Paketdatei "HelloWorld.nupkg** " selbst.

#### Aufgabe 4: Hochladen des Open-Source-NuGet-Pakets in Azure Artifacts

Betrachten wir dieses Paket als genehmigtes Paket für unser DevOps-Team, um es wiederzuverwenden, indem wir es in den zuvor erstellten Azure Artifacts-Paketfeed hochladen.

1. Führen Sie im PowerShell-Fenster die folgenden Befehle aus. Diese bewirken Folgendes:

    ```text
    .\nuget.exe push -source "EShopOnWebShared" -ApiKey AzDO c:\EShopOnWeb\EShopOnWeb.Shared\HelloWorld\HelloWorld.nupkg
    ```

    > Dadurch wird eine Fehlermeldung ausgelöst.

    ```text
    Response status code does not indicate success: 409 (Conflict - 'HelloWorld 1.3.0.17' cannot be published to the feed because it exists in at least one of the feed's upstream sources. Publishing this copy would prevent you from using 'HelloWorld 1.3.0.17' from 'NuGet Gallery'. For more information, see https://go.microsoft.com/fwlink/?linkid=864880 (DevOps Activity ID: AE08BE89-C2FA-4FF7-89B7-90805C88972C)).
    ```

Wenn Sie den Azure DevOps Artifacts-Paketfeed nach Design erstellt haben, ermöglicht **es upstream-Quellen**, z. B. nuget.org im Dotnet-Beispiel. Ihr DevOps-Team blockiert jedoch nichts, um einen **"internen"** Paketfeed zu erstellen.

1. Navigieren Sie zum Azure DevOps-Portal, navigieren Sie zu **Artefakten**, und wählen Sie den **EShopOnWebShared-Feed** aus.
2. Klicken Sie auf **"Upstreamquellen durchsuchen"**
3. Wählen Sie **im **Fenster "Gehe zu einem Upstream-Paket**" "NuGet**" als Pakettyp aus, und geben Sie **"HelloWorld**" in das Suchfeld ein.
4. Bestätigen Sie, indem Sie die **Schaltfläche "Suchen** " drücken.
5. Dies führt zu einer Liste aller HelloWorld-Pakete mit den verschiedenen verfügbaren Versionen.
6. Klicken Sie auf die **NACH-LINKS-TASTE** , um zum **EShopOnWebShared-Feed** zurückzukehren.
7. Klicken Sie auf das Zahnrad, um feed Einstellungen** zu öffnen**. Wählen Sie auf der Seite "Feed Einstellungen" die Option **"Upstreamquellen**" aus.
8. Beachten Sie die verschiedenen Upstream-Paket-Manager für verschiedene Entwicklungssprachen. Wählen Sie **den NuGet-Katalog** aus der Liste aus. Drücken Sie die **Entf-Taste** , gefolgt von der **Schaltfläche "Speichern** ".

9. Mit diesen gespeicherten Änderungen ist es möglich, das HelloWorld-Paket** mithilfe der **NuGet.exe aus dem PowerShell-Fenster hochzuladen, indem sie den folgenden Befehl neu starten:

    ```text
     .\nuget.exe push -source "EShopOnWebShared" -ApiKey AzDO c:\EShopOnWeb\EShopOnWeb.Shared\HelloWorld\HelloWorld.nupkg
    ```

    > **Hinweis**: Dies sollte jetzt zu einem erfolgreichen Upload führen 

    ```text
    Pushing HelloWorld.nupkg to 'https://pkgs.dev.azure.com/pdtdemoworld/7dc3351f-bb0c-42ba-b3c9-43dab8e0dc49/_packaging/188ec0d5-ff93-4eb7-b9d3-293fbf759f06/nuget/v2/'...
      PUT https://pkgs.dev.azure.com/<AZUREDEVOPSORGANIZATION>/7dc3351f-bb0c-42ba-b3c9-43dab8e0dc49/_packaging/188ec0d5-ff93-4eb7-b9d3-293fbf759f06/nuget/v2/
    MSBuild auto-detection: using msbuild version '17.5.0.10706' from 'C:\Program Files\Microsoft Visual Studio\2022\Professional\MSBuild\Current\bin'.
      Accepted https://pkgs.dev.azure.com/pdtdemoworld<AZUREDEVOPSORGANIZATION>/7dc3351f-bb0c-42ba-b3c9-43dab8e0dc49/_packaging/188ec0d5-ff93-4eb7-b9d3-293fbf759f06/nuget/v2/ 1645ms
    Your package was pushed.
    PS C:\eShopOnWeb\EShopOnWeb.Shared>
    ```

10. Aktualisieren Sie** im Azure DevOps-Portal **die Seite "Artifacts Package Feed". Die Liste der Pakete enthält sowohl das **benutzerdefinierte EShopOnWeb.Shared-Paket** als auch das **öffentliche HelloWorld-Quellpaket** .
11. Klicken Sie in der Visual Studio **EShopOnWeb.Shared-Projektmappe** mit der rechten Maustaste auf das **Projekt "EShopOnWeb.Shared** ", und wählen Sie " **NuGet-Pakete** verwalten" im Kontextmenü aus.
12. Überprüfen Sie im Fenster "NuGet Paket-Manager", ob die **Paketquelle** auf **"EShopOnWebShared**" festgelegt ist.
13. Klicken Sie auf "Durchsuchen **", und warten Sie**, bis die Liste der NuGet-Pakete geladen wird.
14. In dieser Liste werden auch das **benutzerdefinierte EShopOnWeb.Shared-Paket** sowie das **öffentliche HelloWorld-Quellpaket** angezeigt.

## Überprüfung

In dieser Übung haben Sie gelernt, wie Sie mit Azure Artifacts arbeiten, indem Sie die folgenden Schritte ausführen:

- Erstellen eines Feeds und Herstellen einer Verbindung.
- Erstellen und Veröffentlichen eines NuGet-Pakets.
- Importiert ein benutzerdefiniertes NuGet-Paket.
- Importiert ein öffentliches NuGet-Paket.
