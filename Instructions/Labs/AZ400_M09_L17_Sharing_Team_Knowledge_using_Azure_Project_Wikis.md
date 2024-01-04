---
lab:
  title: Wissensaustausch im Team mithilfe von Azure-Projekt-Wikis
  module: 'Module 09: Implement continuous feedback'
---

# Wissensaustausch im Team mithilfe von Azure-Projekt-Wikis

## Lab-Handbuch für Kursteilnehmer

## Labanforderungen

- Für dieses Lab ist **Microsoft Edge** oder ein von [Azure DevOps unterstützter Browser](https://docs.microsoft.com/azure/devops/server/compatibility) erforderlich.

- **Einrichten einer Azure DevOps-Organisation:** Wenn Sie noch keine Azure DevOps-Organisation haben, die Sie für dieses Labor verwenden können, erstellen Sie eine, indem Sie den Anweisungen unter [AZ-400 Laborvoraussetzungen](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/AZ400_M00_Validate_lab_environment.html) folgen.

- **Einrichten des EShopOnWeb-Beispielprojekts:** Wenn Sie noch kein EShopOnWeb-Beispielprojekt haben, das Sie für diese Übung verwenden können, erstellen Sie eines, indem Sie die Anweisungen unter [AZ-400 Übungsvoraussetzungen](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/AZ400_M00_Validate_lab_environment.html) befolgen.

## Übersicht über das Labor

In diesem Lab erstellen und konfigurieren Sie ein Wiki in Azure DevOps, einschließlich der Verwaltung von Markdown-Inhalten und der Erstellung eines Mermaid-Diagramms.

## Ziele

In diesem Lab lernen Sie Folgendes:

- Erstellen Sie ein Wiki in einem Azure-Project.
- Hinzufügen und Bearbeiten von Markdown.
- Erstellen Sie ein Mermaid-Diagramm.

## Geschätzte Dauer: 45 Minuten

## Anweisungen

### Übung 0: Konfigurieren der Voraussetzungen für das Lab

In dieser Übung möchten wir Sie daran erinnern, dass Sie die Laborvoraussetzungen validieren müssen, dass Sie eine Azure DevOps-Organisation bereit haben und dass Sie das EShopOnWeb-Projekt erstellt haben. Weitere Einzelheiten finden Sie in den obigen Anweisungen.

### Übung 1: Code als Wiki veröffentlichen

In dieser Übung werden Sie durch die Veröffentlichung eines Azure DevOps-Repositorys als Wiki und die Verwaltung des veröffentlichten Wikis geführt.

> **Hinweis**: Inhalte, die Sie in einem Git-Repository pflegen, können in einem Azure DevOps-Wiki veröffentlicht werden. So können beispielsweise Inhalte, die zur Unterstützung eines Software Development Kits, einer Produktdokumentation oder einer README-Datei geschrieben wurden, direkt in einem Wiki veröffentlicht werden. Sie haben die Möglichkeit, mehrere Wikis innerhalb desselben Azure DevOps-Team-Projekts zu veröffentlichen.

#### Aufgabe 1: Veröffentlichen einer Verzweigung eines Azure DevOps-Repos als Wiki

In dieser Aufgabe werden Sie eine Verzweigung eines Azure DevOps-Repos als Wiki veröffentlichen.

> **Hinweis**: Wenn Ihr veröffentlichtes Wiki einer Produktversion entspricht, können Sie neue Verzweigungen veröffentlichen, wenn Sie neue Versionen Ihres Produkts herausbringen.

1. Klicken Sie im vertikalen Menü auf der linken Seite auf **Repos**. Stellen Sie im oberen Bereich des Fensters **Dateien** sicher, dass Sie das **EShopOnWeb**-Repos ausgewählt haben (wählen Sie es aus dem Dropdown-Menü oben mit dem Git-Symbol). Wählen Sie in der Verzweigungs-Dropdown-Liste (oberhalb von "Dateien" mit dem Verzweigungssymbol) **main**, und überprüfen Sie den Inhalt der Haupterzweigung.
2. Auf der linken Seite des Fensters **Dateien**, in der Auflistung der Repo-Ordner und der Dateihierarchie, erweitern Sie den Ordner **src** und wechseln Sie zum Unterordner **Web-> wwwroot -> images**. Suchen Sie im Unterordner **Images** den Eintrag **brand.png**, fahren Sie mit dem Mauszeiger über dessen rechtes Ende, um die vertikalen Auslassungspunkte (drei Punkte) zu sehen, das das Menü **More** darstellt, klicken Sie auf **Download**, um die Datei **brand.png** in den lokalen Ordner **Downloads** auf Ihrem Laborcomputer herunterzuladen.

    >**Hinweis**: Sie werden dieses Bild im nächsten Teil dieser Übung verwenden.

3. Wir werden die Wiki-Quelldateien in einem separaten Ordner innerhalb der aktuellen Ordnerstruktur des Repos speichern. Wählen Sie innerhalb von **Repos** die Option **Dateien** aus. Beachten Sie den Repo-Titel **EShopOnWeb** oben in der Ordnerstruktur. **Markieren Sie die Auslassungspunkte (3 Punkte)**, wählen Sie **Neu/Ordner**, und geben Sie **Dokumente** als Titel für den neuen Ordnernamen ein. Da ein Repo es nicht erlaubt, einen leeren Ordner zu erstellen, geben Sie **READ.ME** als neuen Dateinamen an.
4. Bestätigen Sie die Erstellung des Ordners und der Datei durch **Drücken der Schaltfläche Erstellen**.
5. Die Datei READ.ME wird im integrierten Ansichtsmodus geöffnet. Da dies als Code gespeichert wird, müssen Sie die Änderungen **Commit**, indem Sie auf die Schaltfläche **Commit** klicken. Bestätigen Sie im Fenster Commit noch einmal mit **Commit**.

6. Klicken Sie im vertikalen Menü von Azure DevOps auf der linken Seite auf **Übersicht**, wählen Sie im Abschnitt **Übersicht** die Option **Wiki** und wählen Sie **Code als Wiki veröffentlichen*.
7. Geben Sie im Bereich **Code als Wiki veröffentlichen** die folgenden Einstellungen an und klicken Sie auf **Veröffentlichen**.

    | Einstellung | Wert |
    | ------- | ----- |
    | Repository | **EShopOnWeb** |
    | Bankfiliale | **main** |
    | Ordner | **Dokumente** |
    | Wiki-Name | **EShopOnWeb (Dokumente)** |

    >**Hinweis**: Dies öffnet automatisch den Wiki-Bereich und veröffentlicht **den Editor**, wo Sie einen Wiki-Seitentitel eingeben und den eigentlichen Inhalt hinzufügen können. Beachten Sie, dass Sie das MarkDown-Format verwenden sollten, aber nutzen Sie das Menüband, um sich mit einigen der MarkDown-Layout-Syntaxen vertraut zu machen.

8. In das Feld Wiki Page **Title** geben Sie "Willkommen in unserem Online Shop!" ein.

9. Fügen Sie im Text der Wiki-Seite den folgenden Text ein:

    ```text
    ##Welcome to Our Online Retail Store!
    At our online retail store, we offer a **wide range of products** to meet the **needs of our customers**. Our selection includes everything from *clothing and accessories to electronics, home decor, and more*.
    
    We pride ourselves on providing a seamless shopping experience for our customers. Our website offers the following benefits:
    1. user-friendly,
    1. and easy to navigate, 
    1. allowing you to find what you're looking for,
    1. quickly and easily. 
    
    We also offer a range of **_payment and shipping options_** to make your shopping experience as convenient as possible.
    
    ### about the team
    Our team is dedicated to providing exceptional customer service. If you have any questions or concerns, our knowledgeable and friendly support team is always available to assist you. We also offer a hassle-free return policy, so if you're not completely satisfied with your purchase, you can easily return it for a refund or exchange.
    
    ### Physical Stores
    |Location|Area|Hours|
    |--|--|--|
    | New Orleans | Home and DIY  |07.30am-09.30pm  |
    | Seattle | Gardening | 10.00am-08.30pm  |
    | New York | Furniture Specialists  | 10.00am-09.00pm |
    
    ## Our Store Qualities
    - We're committed to providing high-quality products
    - Our products are offered at affordable prices 
    - We work with reputable suppliers and manufacturers 
    - We ensure that our products meet our strict standards for quality and durability. 
    - Plus, we regularly offer sales and discounts to help you save even more.
    
    #Summary
    Thank you for choosing our online retail store for your shopping needs. We look forward to serving you!
    ```

10. Dieser Beispieltext gibt Ihnen einen Überblick über verschiedene gängige MarkDown-Syntaxfunktionen, wie z. B. Titel und Untertitel (##), Fettdruck (**), Kursivdruck (*), das Erstellen von Tabellen und vieles mehr.

11. Sobald Sie fertig sind, **drücken Sie** die Schaltfläche "Speichern" in der oberen rechten Ecke.

12. **Aktualisieren Sie** Ihren Browser, oder wählen Sie eine andere DevOps-Portaloption aus, und kehren Sie zum Wiki-Bereich zurück. Beachten Sie, dass Ihnen jetzt das **Wiki "EshopOnWeb (Dokumente)** " angezeigt wird, sowie **Willkommen in unserem Online Shop** als **Startseite** des Wikis.

#### Aufgabe 2: Verwalten von Inhalten eines veröffentlichten Wikis

In dieser Aufgabe verwalten Sie Inhalte des Wikis, das Sie in der vorherigen Aufgabe veröffentlicht haben.

1. Klicken Sie im vertikalen Menü auf der linken Seite auf **Repos**, vergewissern Sie sich, dass das Dropdown-Menü im oberen Bereich des Bereichs **Dateien** das **EShopOnWeb**-Repo und der Verzweigung **main** anzeigt, wählen Sie in der Repo-Ordnerhierarchie den Ordner **Dokumente** aus und wählen Sie die Datei **Willkommen in unserem Online Shop!.md**.
2. Beachten Sie, dass das MarkDown-Format hier als Rohtextformat sichtbar ist, sodass Sie auch die Bearbeitung des Dateiinhalts von hier aus fortsetzen können.

> **Hinweis**: Da die Quelldateien des Wikis wie Quellcode behandelt werden, können alle Praktiken der traditionellen Versionskontrolle (Klonen, Pull Requests, Genehmigungen und mehr) nun auch auf Wiki-Seiten angewendet werden.

### Übung 2: Erstellen und verwalten eines Projekt-Wikis.

In dieser Übung durchlaufen Sie das Erstellen und Verwalten eines Projekt-Wikis.

> **Hinweis**: Sie können das Wiki unabhängig von den bestehenden Repos erstellen und verwalten.

#### Aufgabe 1: Erstellen Sie ein Projekt-Wiki mit einem Mermaid-Diagramm und einem Bild

In dieser Aufgabe erstellen Sie ein Projekt-Wiki und fügen diesem ein Mermaid-Diagramm und ein Bild hinzu.

1. Zeigen Sie auf Ihrem Laborcomputer im Azure DevOps Portal den **Wiki-Bereich** des **EShopOnweb**-Projekts an, wobei der Inhalt des **EShopOnWeb (Dokumente)**-Wikis ausgewählt ist, Klicken Sie oben im Bereich auf die Kopfzeile der Dropdown-Liste **EShopOnWeb (Dokumente)** (das Pfeil-nach-unten-Symbol) und wählen Sie in der Dropdown-Liste die Option **Neues Projekt-Wiki erstellen**.
2. Geben Sie in das Textfeld **Seitentitel** **Projektentwurf** ein.
3. Platzieren Sie den Cursor im Hauptteil der Seite, klicken Sie auf das Symbol ganz links in der Symbolleiste, das die Kopfzeileneinstellung darstellt, und klicken Sie in der Dropdown-Liste auf **Kopfzeile 1**. Dadurch wird automatisch das Rautenzeichen (**#**) am Anfang der Zeile hinzugefügt.
4. Geben Sie direkt nach dem neu hinzugefügten Zeichen **#** **Authentifizierung und Autorisierung** ein und drücken Sie die Taste **Eingabe**.
5. Klicken Sie auf das Symbol ganz links in der Symbolleiste, das die Kopfzeileneinstellung darstellt, und klicken Sie in der Dropdown-Liste auf **Kopfzeile 2**. Dadurch wird automatisch das Rautenzeichen (**##**) am Anfang der Zeile hinzugefügt.
6. Geben Sie direkt nach dem neu hinzugefügten Zeichen **##** **Azure DevOps OAuth 2.0 Authorization Flow** ein und drücken Sie die Taste **Eingabe**.
7. **Kopieren Sie den folgenden Code, und fügen Sie ihn ein, um ein Mermaid-Diagramm in Ihr Wiki einzufügen** .

    ```
    ::: mermaid
    sequenceDiagram
     participant U as User
     participant A as Your app
     participant D as Azure DevOps
     U->>A: Use your app
     A->>D: Request authorization for user
     D-->>U: Request authorization
     U->>D: Grant authorization
     D-->>A: Send authorization code
     A->>D: Get access token
     D-->>A: Send access token
     A->>D: Call REST API with access token
     D-->>A: Respond to REST API
     A-->>U: Relay REST API response
    :::
    ```

    >**Hinweis**: Für Details zur Mermaid-Syntax siehe [Über Mermaid](https://mermaid-js.github.io/mermaid/#/)

8. Klicken Sie rechts neben dem Editorfenster im Vorschaufenster auf **Diagramm laden** und überprüfen Sie das Ergebnis.

    >**Hinweis**: Die Ausgabe sollte dem Flussdiagramm ähneln, das veranschaulicht, wie man [Zugriff auf REST-APIs mit OAuth 2.0 autorisiert](https://docs.microsoft.com/azure/devops/integrate/get-started/authentication/oauth)

9. Klicken Sie in der oberen rechten Ecke des Editorbereichs auf das nach unten zeigende Caret neben der Schaltfläche **Speichern** und klicken Sie im Dropdown-Menü auf **Speichern mit Revisionsmeldung**.
10. Geben Sie im Dialogfeld **Seite speichern** den Abschnitt **Authentifizierung und Autorisierung mit dem OAuth 2.0 Mermaid-Diagramm** ein und klicken Sie auf **Speichern**.
11. Setzen Sie den Cursor im Editorbereich **Projektdesign** an das Ende des Mermaid-Elements, das Sie zuvor in dieser Aufgabe hinzugefügt haben, drücken Sie die Taste **Eingabe**, um eine zusätzliche Zeile hinzuzufügen, klicken Sie auf das Symbol ganz links in der Symbolleiste, das die Einstellung für die Überschrift darstellt, und klicken Sie in der Dropdown-Liste auf **Überschrift 2**. Dadurch wird automatisch das doppelte Rautenzeichen (**##**) am Anfang der Zeile hinzugefügt.
12. Direkt nach dem neu hinzugefügten Zeichen **##** geben Sie **Benutzeroberfläche** ein und drücken die Taste **Eingabe**.
13. Klicken Sie im Editorbereich **Projektdesign** in der Symbolleiste auf das Büroklammer-Symbol, das die Aktion **Datei einfügen** darstellt, navigieren Sie im Dialogfeld **Öffnen** zum Ordner **Downloads**, wählen Sie die Datei **Brand.png** aus, die Sie in der vorherigen Übung heruntergeladen haben, und klicken Sie auf **Öffnen**.
14. Gehen Sie zurück in den Editorbereich **Projektdesign** und überprüfen Sie, ob das Bild richtig angezeigt wird.
15. Klicken Sie in der oberen rechten Ecke des Editorbereichs auf das nach unten zeigende Caret neben der Schaltfläche **Speichern** und klicken Sie im Dropdown-Menü auf **Speichern mit Revisionsmeldung**.
16. Im Dialogfeld **Seite speichern** geben Sie **Benutzeroberflächenabschnitt mit dem Tailwind Traders-Bild** ein und klicken auf **Speichern**.
17. Zurück im Editorbereich, klicken Sie in der oberen rechten Ecke auf **Schließen**.

#### Aufgabe 2: Verwalten eines Projekt-Wikis

In dieser Aufgabe werden Sie das neu erstellte Projekt-Wiki verwalten.

>**Hinweis**: Sie beginnen damit, die letzte Änderung an der Wikiseite rückgängig zu machen.

1. Klicken Sie auf Ihrem Laborcomputer im Azure DevOps-Portal, das den **Wiki-Bereich** des **EShopOnWeb**-Projekts anzeigt, bei ausgewähltem Inhalt des **Projektdesign**-Wikis in der oberen rechten Ecke auf die vertikalen Auslassungspunkte und klicken Sie im Dropdown-Menü auf **Überarbeitungen anzeigen**.
2. Klicken Sie im Bereich **Überarbeitungen** auf den Eintrag, der die letzte Änderung darstellt.
3. Überprüfen Sie im daraufhin angezeigten Fenster den Vergleich zwischen der vorherigen und der aktuellen Version des Dokuments, klicken Sie auf **Zurücksetzen**, klicken Sie bei der Aufforderung zur Bestätigung erneut auf **Zurücksetzen** und dann auf **Seite durchsuchen**.
4. Überprüfen Sie im Bereich **Projektentwurf**, ob die Änderung erfolgreich rückgängig gemacht wurde.

    >**Hinweis**: Jetzt fügen Sie dem Projektwiki eine weitere Seite hinzu und legen sie als Wiki-Startseite fest.

5. Klicken Sie im Bereich **Projektentwurf** in der unteren linken Ecke auf **+Neue Seite**.
6. Geben Sie im Seiten-Editor-Bereich im Textfeld **Seitentitel** den Namen **Projektentwurfsübersicht** ein, klicken Sie auf **Speichern**, und klicken Sie dann auf **Schließen**.
7. Suchen Sie im Bereichmit der Auflistung der Seiten im Projektwiki vom **Projektentwurf** den Eintrag **Projektentwurfsübersicht**, wählen Sie ihn mit dem Mauszeiger aus und ziehen Sie ihn per Drag&Drop über den Eintrag **Projektentwurf**.
8. Bestätigen Sie die Änderungen, indem Sie im angezeigten Fenster die Schaltfläche **Verschieben** drücken.
9. Stellen Sie sicher, dass der Eintrag **Projektentwurfsübersicht** als Seite auf oberster Ebene steht und das Startseitensymbol der Wiki-Homepage hat.

## Überprüfung

In diesem Lab haben Sie ein Wiki in Azure DevOps erstellt und konfiguriert, einschließlich der Verwaltung von Markdown-Inhalten und der Erstellung eines Mermaid-Diagramms.
