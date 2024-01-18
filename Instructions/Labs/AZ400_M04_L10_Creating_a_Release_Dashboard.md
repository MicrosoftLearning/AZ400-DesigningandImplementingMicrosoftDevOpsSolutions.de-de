---
lab:
  title: Erstellen eines Release-Dashboards
  module: 'Module 04: Design and implement a release strategy'
---

# Erstellen eines Release-Dashboards

# Lab-Handbuch für Kursteilnehmer

## Labanforderungen

- Für dieses Lab ist **Microsoft Edge** oder ein von [Azure DevOps unterstützter Browser](https://docs.microsoft.com/en-us/azure/devops/server/compatibility?view=azure-devops#web-portal-supported-browsers) erforderlich.

- **Einrichten einer Azure DevOps-Organisation**: Wenn Sie nicht bereits eine Azure DevOps-Organisation haben, die Sie für dieses Lab verwenden können, müssen Sie diese erstellen, indem Sie die unter [Erstellen einer Organisation oder Projektsammlung](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/create-organization?view=azure-devops) beschriebenen Anweisungen befolgen.

- Identifizieren Sie ein vorhandenes Azure-Abonnement, oder erstellen Sie ein neues Abonnement.

- Vergewissern Sie sich, dass Sie über ein Microsoft-Konto oder ein Azure AD-Konto in der Besitzerrolle im Azure-Abonnement verfügen. Ausführliche Informationen finden Sie unter [Auflisten von Azure-Rollenzuweisungen mithilfe des Azure-Portals](https://docs.microsoft.com/en-us/azure/role-based-access-control/role-assignments-list-portal).

## Übersicht über das Labor

In diesem Lab erstellen Sie ein Releasedashboard und verwenden eine REST-API zum Abrufen von Azure DevOps-Releasedaten, die Sie Ihren benutzerdefinierten Anwendungen oder Dashboards zur Verfügung stellen können.

Das Lab verwendet die Azure DevOps Starter-Ressource – diese erstellt automatisch ein Azure DevOps-Projekt, das eine Anwendung erstellt und in Azure bereitstellt.

## Ziele

In diesem Lab lernen Sie Folgendes:

- Erstellen Sie ein Releasedashboard.
- Verwenden Sie eine REST-API, um Releaseinformationen abzufragen.

## Geschätzte Dauer: 45 Minuten

## Anweisungen

### Übung 1: Erstellen eines Release-Dashboards

In dieser Übung erstellen Sie ein Release-Dashboard in einer Azure DevOps-Organisation.

#### Aufgabe 1: Erstellen einer Azure DevOps Starter-Ressource

In dieser Aufgabe erstellen Sie eine Azure DevOps Starter-Ressource in Ihrem Azure-Abonnement. Dadurch wird automatisch ein entsprechendes Projekt in Ihrer Azure DevOps-Organisation erstellt.

1. Starten Sie auf Ihrem Lab-Computer einen Webbrowser, navigieren Sie zum [**Azure-Portal**](https://portal.azure.com), und melden Sie sich mit dem Benutzerkonto an, das in dem Abonnement, das Sie in diesem Lab verwenden, über die Besitzerrolle oder Mitwirkenderrolle verfügt.
1. Suchen Sie im Azure-Portal nach dem **DevOps Starter**-Ressourcentyp, und klicken Sie auf dem **DevOps Starter**-Blatt auf **+ Erstellen**.
1. Wählen Sie auf dem **DevOps Starter**-Blatt in dem Bereich **Von vorn beginnen mit neuer Anwendung** die **.NET**-Kachel aus. Oben neben **Einrichten von DevOps Starter mit GitHub**, Einstellungen ändern, klicken Sie **hier** und wählen Sie **Azure DevOps**, **Fertig** und **Weiter: Framework >** aus.
1. Wählen Sie auf dem **DevOps Starter**-Blatt im Bereich **Anwendungsframework auswählen** die **ASP<nolink>.NET Core**-Kachel aus, verschieben Sie den Schieberegler **Datenbank hinzufügen** auf die Position **Ein** und klicken Sie auf **Weiter: Dienst >**.
1. Stellen Sie sicher, dass auf dem **DevOps Starter**-Blatt im Bereich **Azure-Dienst zum Bereitstellen der Anwendung auswählen** die Kachel **Windows Web App** ausgewählt ist und klicken Sie auf **Weiter: Erstellen >**.
1. Geben Sie auf dem **DevOps Starter**-Blatt im Bereich **Fast geschafft** die folgenden Einstellungen an:

    | Einstellung | Wert |
    | ------- | ----- |
    | Projektname | **Erstellen eines Release-Dashboards** |
    | Azure DevOps-Organisation | Name der Azure DevOps-Organisation, die Sie in dieser Registerkarte verwenden werden. |
    | Subscription | Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden. |
    | Web-App-Name | Eine global eindeutige Zeichenfolge zwischen 2 und 60 Zeichen, die aus Buchstaben, Ziffern und Bindestrichen besteht und entweder mit einem Buchstaben oder einer Zahl beginnt und endet |
    | Standort | Name der Azure-Region, in der Sie eine Azure Web App und eine Azure SQL-Datenbank bereitstellen möchten |

1. Klicken Sie auf dem **DevOps Starter**-Blatt im Bereich **Fast geschafft** auf **Zusätzliche Einstellungen**.
1. Geben Sie im Bereich **Zusätzliche Einstellungen** die folgenden Einstellungen an, und klicken Sie auf **OK**.

    | Einstellung | Wert |
    | ------- | ----- |
    | Ressourcengruppe | **az400m10l02-rg** |
    | Tarif | **F1 Free** |
    | Application Insights-Standort | Name der gleichen Azure-Region, die Sie für den Standort der Azure Web-App ausgewählt haben |
    | Servername | Eine global eindeutige Zeichenfolge zwischen 3 und 63 Zeichen, die aus Buchstaben, Ziffern und Bindestrichen besteht und entweder mit einem Buchstaben oder einer Zahl beginnt und endet |
    | Benutzernamen eingeben | **dbadmin** |
    | Standort | Name der gleichen Azure-Region, die Sie für den Standort der Azure Web-App ausgewählt haben |
    | Database Name | **az400m10l02-db** |

1. Zurück auf dem **DevOps Starter**-Blatt klicken Sie im Bereich **Fast geschafft** auf **Fertig** und dann auf **Überprüfen und erstellen**.

    > **Hinweis**: Warten Sie, bis die Bereitstellung abgeschlossen ist. Die Bereitstellung der **DevOps Starter**-Ressource sollte ungefähr 2 Minuten dauern.

1. Nachdem Sie die Bestätigung erhalten, dass die DevOps Starter-Ressource bereitgestellt wurde, klicken Sie auf die Schaltfläche **Zu Ressource wechseln**. Dadurch wird der Browser auf das DevOps Starter-Blatt umgeleitet.
1. Verfolgen Sie auf dem DevOps Starter-Blatt den Fortschritt der CI/CD-Pipeline, bis sie erfolgreich abgeschlossen wurde.

    > **Hinweis**: Die Erstellung der entsprechenden Azure Web-App und Azure SQL-Datenbank kann etwa 5 Minuten dauern. Der Prozess erstellt automatisch ein Azure DevOps-Projekt, das ein bereitstellungsfertiges Repository sowie die Build- und Release-Pipelines enthält. Die Azure-Ressourcen werden als Teil der automatisch ausgelösten Bereitstellungspipeline erstellt.

#### Aufgabe 2: Erstellen von Azure DevOps-Versionen

In dieser Aufgabe erstellen Sie mehrere Azure DevOps-Versionen, einschließlich einer, die zu einer fehlgeschlagenen Bereitstellung führt.

1. Klicken Sie im Webbrowser, in dem das Azure-Portal angezeigt wird, auf der DevOps Starter-Seite auf der Symbolleiste auf die **Projektstartseite**. Dadurch wird automatisch eine andere Browserregisterkarte geöffnet, auf der das Projekt **Erstellen eines Releasedashboards** im Azure Devops-Portal angezeigt wird. Wenn Sie aufgefordert werden, sich anzumelden, authentifizieren Sie sich mit Ihren Azure DevOps-Organisationsanmeldeinformationen.

    > **Hinweis**: Zuerst erstellen Sie eine neue Version, die erfolgreich bereitgestellt wird.

1. Klicken Sie im Azure DevOps-Portal im vertikalen Menü auf der linken Seite auf **Repos**, in der Liste der Ordner im Repository, navigieren Sie zum Ordner **Application\\aspnet-core-dotnet-core\\Pages** und klicken Sie auf den Eintrag **Index.cshtml**.
1. Klicken Sie im Bereich **Index.cshtml** in Zeile **20** auf **Bearbeiten**, ersetzenSie `<div class="description line-2"> Your ASP.NET Core app is up and running on Azure</div>` durch `<div class="description line-2"> Your ASP.NET Core app v1.1 is up and running on Azure</div>` und klicken Sie **Commit**. Klicken Sie im Bereich **Commit** erneut auf **Commit**. Dadurch wird die Buildpipeline automatisch gestartet.
1. Klicken Sie im Azure DevOps-Portal im vertikalen Navigationsbereich auf der linken Seite auf **Pipelines**.
1. Klicken Sie auf der Registerkarte **Zuletzt verwendet** im Bereich **Pipelines** auf den Eintrag **az400m10l02-CI**. Wählen Sie auf der Registerkarte **Ausführen** des Bereichs **az400m10l02-CI** auf der Registerkarte **Zusammenfassung** der Ausführung im Abschnitt **Aufträge** auf **Build** und überwachen Sie den Auftrag bis zum erfolgreichen Abschluss.
1. Nachdem der Auftrag abgeschlossen ist, klicken Sie im Azure DevOps-Portal im vertikalen Navigationsbereich auf der linken Seite im Abschnitt **Pipelines** auf **Versionen**.
1. Klicken Sie im Bereich **az400m10l02-CD** auf der Registerkarte **Versionen** auf den Eintrag **Release-2**. Klicken Sie auf der Registerkarte **Pipeline** des Bereichs **Release-2** auf die Phase **Entwicklung**. Klicken Sie im Bereich **Entwicklung** auf **Protokolle anzeigen** und überwachen Sie den Fortschritt der Bereitstellung bis zum erfolgreichen Abschluss.

    > **Hinweis**: Jetzt erstellen Sie eine neue Version, bei der die Bereitstellung fehlschlägt. Der Fehler wird durch den integrierten Assemblys-Test verursacht, bei dem die Änderung, die der neuen Version zugeordnet ist, als ungültig betrachtet wird.

1. Klicken Sie im Azure DevOps-Portal im vertikalen Menü auf der linken Seite auf **Repos**, in der Liste der Ordner im Repository, navigieren Sie zum Ordner **Application\\aspnet-core-dotnet-core\\Pages** und klicken Sie auf den Eintrag **Index.cshtml**.
1. Klicken Sie im Bereich **Index.cshtml** auf **Bearbeiten**. Ersetzen Sie in Zeile **4** `ViewData["Title"] = "Home Page - ASP.NET Core";` durch `ViewData["Title"] = "Home Page v1.2 - ASP.NET Core";`. Klicke auf **Commit ausführen**. Klicken Sie im Bereich **Commit** erneut auf **Commit**. Dadurch wird die Buildpipeline automatisch gestartet.
1. Klicken Sie im Azure DevOps-Portal im vertikalen Navigationsbereich auf der linken Seite auf **Pipelines**.
1. Klicken Sie auf der Registerkarte **Zuletzt verwendet** im Bereich **Pipelines** auf den Eintrag **az400m10l02-CI**. Wählen Sie auf der Registerkarte **Ausführen** des Bereichs **az400m10l02-CI** auf der Registerkarte **Zusammenfassung** der Ausführung im Abschnitt **Aufträge** auf **Build** und überwachen Sie den Auftrag bis zum erfolgreichen Abschluss.
1. Nachdem der Auftrag abgeschlossen ist, klicken Sie im Azure DevOps-Portal im vertikalen Navigationsbereich auf der linken Seite im Abschnitt **Pipelines** auf **Versionen**.
1. Klicken Sie im Bereich **Az400m10l02-CD** auf der Registerkarte **Versionen** auf den Eintrag **Release-3**. Klicken Sie auf der Registerkarte **Pipeline** des Bereichs **Release-3** auf die Phase **Entwicklung**. Klicken Sie im Bereich **Entwicklung** auf **Protokolle anzeigen** und überwachen Sie den Fortschritt der Bereitstellung, bis der Fehler während der Phase **Assembly-Test** auftritt.

#### Aufgabe 3: Erstellen eines Azure DevOps-Versionsdashboards

In dieser Aufgabe erstellen Sie ein Dashboard und fügen ihm versionsbezogene Widgets hinzu.

1. Klicken Sie im Azure DevOps-Portal im vertikalen Menü auf der linken Seite auf **Übersicht**, klicken Sie im Abschnitt **Übersicht** auf **Dashboards** und dann auf **Widget hinzufügen**.
1. Scrollen Sie im Bereich **Widget hinzufügen** in der Liste der Widgets nach unten, wählen Sie den Eintrag **Bereitstellungsstatus** aus, und klicken Sie auf **Hinzufügen**.
1. Verwenden Sie das im vorherigen Schritt beschriebene Verfahren, um die Widgets **Versionsintegritätsdetails**, **Versionsintegritätsübersicht** und **Versionspipelineübersicht** hinzuzufügen.
    > **Hinweis**: Installieren Sie **Versionsintegritätsdetails** und ** Versionsintegritätsübersicht** über das Marketplace-[Teamprojektintegrität](https://marketplace.visualstudio.com/items?itemName=ms-devlabs.TeamProjectHealth)
1. Verwenden Sie die Maus, um die ** Versionspipelineübersicht** auf die rechte Seite des **Bereitstellungsstatus**-Widgets zu ziehen, um zu vermeiden, dass ein vertikaler Bildlauf durch das Dashboard erforderlich ist, und klicken Sie auf **Bearbeitung fertig**.
1. Klicken Sie im Dashboard-Bereich im Rechteck, das das **Bereitstellungsstatus**-Widget darstellt, auf **Widget konfigurieren**.
1. Geben Sie im Bereich **Konfiguration** die folgenden Einstellungen an (übernehmen Sie die Standardwerte für andere Einstellungen), und klicken Sie auf **Speichern**.

    | Einstellung | Wert |
    | ------- | ----- |
    | Buildpipeline | **az400m10l02 - CI** |
    | Verknüpfte Versionspipelines | **az400m10l02 - CD; az400m10l02 - CD\dev** |

1. Zeigen Sie im Dashboard-Bereich auf die obere rechte Ecke des Rechtecks, das das Widget **Versionsintegritätsübersicht** darstellt, um das Auslassungszeichen anzuzeigen, das das Menü **Weitere Aktionen** darstellt, klicken Sie darauf, und klicken Sie im Dropdown-Menü  auf **Konfigurieren**.  
1. Geben Sie im Bereich **Konfiguration** die folgenden Einstellungen an (übernehmen Sie die Standardwerte für andere Einstellungen), und klicken Sie auf **Speichern**.

    | Einstellung | Wert |
    | ------- | ----- |
    | Versionsdefinition(en) auswählen | **az400m10l02 - CD** |

1. Zeigen Sie im Dashboard-Bereich auf die obere rechte Ecke des Rechtecks, das das Widget **Versionsintegritätsdetails** darstellt, um das Auslassungszeichen anzuzeigen, das das Menü  **Weitere Aktionen** darstellt, klicken Sie darauf, und klicken Sie im Dropdown-Menü  auf **Konfigurieren**.  
1. Geben Sie im Bereich **Konfiguration** die folgenden Einstellungen an (übernehmen Sie die Standardwerte für andere Einstellungen), und klicken Sie auf **Speichern**.

    | Einstellung | Wert |
    | ------- | ----- |
    | Definition | **az400m10l02 - CD** |

1. Zeigen Sie im Dashboard-Bereich mit der Maus auf die obere rechte Ecke des Rechtecks, das das Widget **Versionspipelineübersicht** darstellt, um das Auslassungszeichen anzuzeigen, das das Menü  **Weitere Aktionen** darstellt, klicken Sie darauf, und klicken Sie im Dropdownmenü  auf **Konfigurieren**.  
1. Geben Sie im Bereich **Konfiguration** die folgenden Einstellungen an (übernehmen Sie die Standardwerte für andere Einstellungen), und klicken Sie auf **Speichern**.

    | Einstellung | Wert |
    | ------- | ----- |
    | Releasepipeline | **az400m10l02 - CD** |

1. Zurück im Dashboard-Bereich klicken Sie auf **Aktualisieren**, um den inhalt zu aktualisieren, der von den Widgets angezeigt wird.

    > **Hinweis**: Mit den Links auf den Widgets können Sie direkt zu den entsprechenden Bereichen im Azure DevOps-Portal navigieren.

### Übung 2: Abfragen von Versionsinformationen über eine REST-API

In dieser Übung fragen Sie Versionsinformationen über die REST-API mithilfe von Postman ab.

#### Aufgabe 1: Generieren eines persönlichen Azure DevOps-Zugriffstokens

In dieser Aufgabe generieren Sie ein persönliches Azure DevOps-Zugriffstoken, das zur Authentifizierung über die Postman-App verwendet wird, die Sie in der nächsten Aufgabe dieser Übung installieren.

1. Klicken Sie auf dem Lab-Computer im Webbrowserfenster mit dem Azure DevOps-Portal in der oberen rechten Ecke der Azure DevOps-Seite auf das Symbol **Benutzereinstellungen**, klicken Sie im Dropdownmenü  auf **Persönliche Zugriffstoken** im Bereich **Persönliche Zugriffstoken** und klicken Sie auf **+ Neues Token**.
1. Klicken Sie im Bereich **Neues persönlichen Zugriffstoken erstellen** auf den Link **Alle Bereiche anzeigen** und geben Sie die folgenden Einstellungen an. Klicken Sie dann auf **Erstellen** (bei alle anderen Standardwerte lassen):

    | Einstellung | Wert |
    | --- | --- |
    | Name | **Erstellen eines Release-Dashboards** |
    | `Scope` | **Release** |
    | Berechtigungen | **Lesen** |
    | `Scope` | **Build** |
    | Berechtigungen | **Lesen** |

1. Kopieren Sie im Bereich **Erfolg** den Wert des persönlichen Zugriffstokens in die Zwischenablage.

    > **Hinweis**: Stellen Sie sicher, dass Sie den Wert des Tokens speichern. Sie können ihn nicht mehr abrufen, nachdem Sie diesen Bereich geschlossen haben.

1. Klicken Sie im Bereich **Erfolg** auf **Schließen**.

#### Aufgabe 2: Abfragen von Versionsinformationen über REST-API mithilfe von Postman

In dieser Aufgabe fragen Sie Versionsinformationen über REST-API mithilfe von Postman ab.

1. Starten Sie auf dem Lab-Computer einen Webbrowser und navigieren Sie zur [Postman-Downloadseite](https://www.postman.com/downloads/), klicken Sie im Dropdownmenü  auf die Schaltfläche **App herunterladen**, klicken Sie auf **Windows 64-Bit**, klicken Sie auf die heruntergeladene Datei und führen Sie die Installation aus. Nach Abschluss der Installation wird die Postman-Desktop-App automatisch gestartet.
1. Geben Sie im Bereich **Postman-Konto erstellen** Ihre E-Mail-Adresse, einen Benutzernamen und ein Kennwort ein, und klicken Sie auf **Kostenloses Konto erstellen**.

    > **Hinweis**: Sie erhalten eine E-Mail von Postman, um Ihr Postman-Konto zu aktivieren, um den Prozess der Kontobereitstellung abzuschließen.

1. Nachdem Sie sich angemeldet haben, klicken Sie im Fenster der Postman-Desktop-App in der oberen linken Ecke auf **+Neu**, klicken Sie im Bereich **Neu erstellen** auf **Anforderung**. Geben Sie im Bereich ** ANFORDERUNG SPEICHERN** im Textfeld **Anforderungsname** den Text **Get-Releases** ein und klicken Sie auf **+ Sammlung erstellen**. Geben Sie im Textfeld **Sammlung benennen** den Text **Azure DevOps az400m10l02-Abfragen** ein. Klicken Sie auf das Häkchen auf der rechten Seite und klicken Sie dann auf die Schaltfläche **In Azure DevOps az400m10l02-Abfragen speichern**.
1. Öffnen Sie ein weiteres Webbrowserfenster, und navigieren Sie zur [Microsoft Docs-Seite **Versionen – Liste**](https://docs.microsoft.com/en-us/rest/api/azure/devops/release/releases/list?view=azure-devops-rest-6.0) und überprüfen Sie deren Inhalt.
1. Wechseln Sie zurück zur Postman-Desktop-App, klicken Sie im Launchpad-Bereich im oberen rechten Abschnitt des App-Fensters auf die Registerkarte **Autorisierung**, wählen Sie in der Dropdownliste **TYP** den Eintrag **Standardauthentifizierung** aus und fügen Sie im Textfeld **Kennwort** den Wert des Tokens ein, das Sie in die vorherige Aufgabe kopiert haben (legen Sie den Wert des Textfelds **Benutzername** nicht fest).
1. Stellen Sie sicher, dass im Launchpad-Bereich im oberen rechten Abschnitt des App-Fensters **GET** in der Dropdownliste angezeigt wird. Geben Sie im Textfeld **Anforderungs-URL eingeben** Folgendes ein, und klicken Sie auf **Senden** (ersetzen Sie den Wert des `<organization_name>` mit dem Namen Ihrer Azure DevOps-Organisation), um alle Versionen auflisten zu können:

    ```url
    https://vsrm.dev.azure.com/<organization_name>/Creating%20a%20Release%20Dashboard/_apis/release/releases?api-version=6.0
    ```

1. Überprüfen Sie die auf der Registerkarte **Textkörper** im unteren rechten Abschnitt des App-Fensters aufgeführte Ausgabe, und stellen Sie sicher, dass sie die Auflistung der Versionen enthält, die Sie in der vorherigen Übung dieses Labs erstellt haben.
1. Wechseln Sie zum Fenster des Webbrowsers, in dem der Inhalt der Microsoft-Dokumentation angezeigt wird und navigieren Sie zur [Microsoft-Dokumentationsseite **Bereitstellungen – Liste**](https://docs.microsoft.com/en-us/rest/api/azure/devops/release/deployments/list?view=azure-devops-rest-6.0), und sehen Sie sich deren Inhalt an.
1. Stellen Sie sicher, dass im Launchpad-Bereich im oberen rechten Abschnitt des App-Fensters **GET** in der Dropdown-Liste angezeigt wird. Geben Sie im Textfeld **Anforderungs-URL eingeben** Folgendes ein und klicken Sie auf **Senden** (ersetzen Sie den Wert des `<organization_name>` mit dem Namen Ihrer Azure DevOps-Organisation), um alle Bereitstellungen aufzulisten:

    ```url
    https://vsrm.dev.azure.com/<organization_name>/Creating%20a%20Release%20Dashboard/_apis/release/deployments?api-version=6.0
    ```

1. Überprüfen Sie die auf der Registerkarte **Textkörper** im unteren rechten Abschnitt des App-Fensters aufgeführte Ausgabe, und stellen Sie sicher, dass sie die Auflistung der Bereitstellungen enthält, die Sie in der vorherigen Übung dieses Lab initiiert haben.
1. Stellen Sie sicher, dass im Launchpad-Bereich im oberen rechten Abschnitt des App-Fensters **GET** in der Dropdown-Liste angezeigt wird. Geben Sie im Textfeld **Anforderungs-URL eingeben** Folgendes ein und klicken Sie auf **Senden** (ersetzen Sie den Wert des `<organization_name>` mit dem Namen Ihrer Azure DevOps-Organisation), um alle Bereitstellungen aufzulisten:

    ```url
    https://vsrm.dev.azure.com/<organization_name>/Creating%20a%20Release%20Dashboard/_apis/release/deployments?DeploymentStatus=failed&api-version=6.0
    ```

1. Überprüfen Sie die auf der Registerkarte **Textkörper** im unteren rechten Abschnitt des App-Fensters aufgeführte Ausgabe, und überprüfen Sie, ob sie nur die fehlgeschlagene Bereitstellung enthält, die Sie in der vorherigen Übung dieses Labs initiiert haben.

### Übung 3: Entfernen der Azure-Laborressourcen

In dieser Übung entfernen Sie die in diesem Lab bereitgestellten Azure-Ressourcen, um unerwartete Gebühren zu vermeiden.

>**Hinweis**: Denken Sie daran, alle neu erstellten Azure-Ressourcen zu entfernen, die Sie nicht mehr verwenden. Durch das Entfernen nicht verwendeter Ressourcen wird sichergestellt, dass keine unerwarteten Kosten anfallen.

#### Aufgabe 1: Entfernen der Azure Lab-Ressourcen

In dieser Aufgabe verwenden Sie Azure Cloud Shell, um die in diesem Lab bereitgestellten Azure-Ressourcen zu entfernen, um unnötige Gebühren zu vermeiden.

1. Öffnen Sie im Azure-Portal im **Cloud Shell**-Bereich die **Bash**-Sitzung.
1. Listen Sie alle Ressourcengruppen auf, die während der Labs in diesem Modul erstellt wurden, indem Sie den folgenden Befehl ausführen:

    ```sh
    az group list --query "[?starts_with(name,'az400m10l02-rg')].name" --output tsv
    ```

1. Löschen Sie alle Ressourcengruppen, die Sie während der praktischen Übungen in diesem Modul erstellt haben, indem Sie den folgenden Befehl ausführen:

    ```sh
    az group list --query "[?starts_with(name,'az400m10l02-rg')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**Hinweis**: Der Befehl wird (dem --nowait-Parameter entsprechend) asynchron ausgeführt. Dies bedeutet, dass Sie zwar einen weiteren Azure CLI-Befehl in derselben Bash-Sitzung direkt im Anschluss ausführen können, es jedoch einige Minuten dauert, bis die Ressourcengruppen tatsächlich entfernt werden.

## Überprüfung

In dieser Übung haben Sie erfahren, wie Sie das Releasedashboard erstellen und konfigurieren und wie Sie eine REST-API zum Abrufen von Azure DevOps-Versionsdaten verwenden.
