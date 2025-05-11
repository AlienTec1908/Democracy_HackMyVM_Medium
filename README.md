# Democracy - HackMyVM (Medium)

![Democracy.png](Democracy.png)

## Übersicht

*   **VM:** Democracy
*   **Plattform:** [HackMyVM](https://hackmyvm.eu/machines/machine.php?vm=Democracy)
*   **Schwierigkeit:** Medium
*   **Autor der VM:** DarkSpirit
*   **Datum des Writeups:** 23. Juli 2023
*   **Original-Writeup:** https://alientec1908.github.io/Democracy_HackMyVM_Medium/
*   **Autor:** Ben C.

## Kurzbeschreibung

Das Ziel dieser "Medium"-Challenge war es, Zugriff auf die Maschine "Democracy" zu erlangen und die Flags zu finden. Nach der initialen Enumeration eines Webservers (Apache) wurden über `nikto` und `gobuster` verschiedene PHP-Seiten wie `login.php`, `register.php` und `vote.php` sowie eine (leere) `config.php` identifiziert. Ein spezielles Bash-Skript wurde verwendet, um durch wiederholte POST-Anfragen an `vote.php` einen Schutzmechanismus zu umgehen. Anschließend wurde mit `sqlmap` eine SQL-Injection-Schwachstelle im Parameter `candidate` der Seite `vote.php` (nach Authentifizierung) erfolgreich ausgenutzt. Dies ermöglichte das Auslesen der Datenbankstruktur und das Dumpen der `users`-Tabelle, die Benutzernamen und Passwörter im Klartext enthielt. Der Bericht endet nach dem Dump der Benutzerdaten. Es ist anzunehmen, dass eine der gefundenen Kombinationen für einen SSH-Login verwendet wurde, um die User- und Root-Flags zu erlangen.

## Disclaimer / Wichtiger Hinweis

Die in diesem Writeup beschriebenen Techniken und Werkzeuge dienen ausschließlich zu Bildungszwecken im Rahmen von legalen Capture-The-Flag (CTF)-Wettbewerben und Penetrationstests auf Systemen, für die eine ausdrückliche Genehmigung vorliegt. Die Anwendung dieser Methoden auf Systeme ohne Erlaubnis ist illegal. Der Autor übernimmt keine Verantwortung für missbräuchliche Verwendung der hier geteilten Informationen. Handeln Sie stets ethisch und verantwortungsbewusst.

## Verwendete Tools

*   `arp-scan`
*   `nmap`
*   `nikto`
*   `gobuster`
*   `curl`
*   `bash` (für Skripting)
*   `sqlmap`
*   Standard Linux-Befehle (`vi`, `cat`)

## Lösungsweg (Zusammenfassung)

Der Angriff auf die Maschine "Democracy" gliederte sich in folgende Phasen:

1.  **Reconnaissance & Web Enumeration:**
    *   IP-Findung mittels `arp-scan` (Ziel: `192.168.2.126`, Hostname `democracy.hmv`).
    *   `nmap`-Scan identifizierte SSH (22/tcp) und Apache (80/tcp) mit dem Titel "Vote for Your Candidate".
    *   `nikto` fand Hinweise auf `/login.php`, `/config.php` und Directory Indexing in `/images/`.
    *   `gobuster` identifizierte `index.php`, `login.php`, `register.php`, `vote.php` (Weiterleitung zu Login) und eine leere `config.php`.

2.  **SQL Injection Exploit:**
    *   Es wurde angenommen, dass eine Benutzerregistrierung und ein Login erfolgten, um auf `vote.php` zugreifen zu können.
    *   Ein Bash-Skript wurde erstellt und parallel ausgeführt, um durch wiederholte POST-Anfragen (`reset=1`) an `vote.php` einen serverseitigen Schutzmechanismus (vermutlich gegen wiederholtes Voten oder schnelle Anfragen) zu umgehen.
    *   Nachdem das Skript einige Zeit gelaufen war, wurde `sqlmap` verwendet, um die Seite `vote.php` auf SQL-Injection zu testen. Die Cookies (`PHPSESSID`, `voted=1`) einer authentifizierten Sitzung wurden mitgesendet.
    *   `sqlmap` identifizierte eine SQL-Injection-Schwachstelle im POST-Parameter `candidate`.
    *   Die Datenbank wurde als MySQL/MariaDB identifiziert. Die Datenbank `voting` mit den Tabellen `users` und `votes` wurde gefunden.
    *   Der Inhalt der Tabelle `users` wurde mit `sqlmap` gedumpt. Diese Tabelle enthielt 135 Einträge mit Benutzernamen und Passwörtern im **Klartext**.

3.  **Zugriffserlangung und Flag-Extraktion (Vermutet):**
    *   Der dokumentierte Bericht endet nach dem Dump der Klartext-Passwörter.
    *   Es ist anzunehmen, dass eine der Kombinationen aus Benutzername und Passwort aus der `users`-Tabelle verwendet wurde, um sich per SSH (Port 22) auf dem System anzumelden.
    *   Nach erfolgreichem Login wurden (vermutlich) die User- und Root-Flags gefunden und ausgelesen. Die genauen Schritte zur Privilegieneskalation (falls notwendig) sind nicht dokumentiert.

## Wichtige Schwachstellen und Konzepte

*   **SQL-Injection:** Eine klassische SQLi-Schwachstelle im `candidate`-Parameter der `vote.php` ermöglichte den Datenbankzugriff.
*   **Speicherung von Passwörtern im Klartext:** Die `users`-Tabelle enthielt Passwörter unverschlüsselt, was eine schwerwiegende Sicherheitslücke darstellt.
*   **Umgehung von Schutzmechanismen:** Ein serverseitiger Schutzmechanismus (Rate Limiting oder ähnliches) musste durch wiederholte Anfragen umgangen werden, bevor die SQLi erfolgreich war.
*   **Information Disclosure:** `nikto` und `gobuster` lieferten wichtige Informationen über die Webanwendungsstruktur.

## Flags

*(Die Erlangung dieser Flags und die genauen Pfade sind basierend auf dem Writeup nicht vollständig dokumentiert und werden hier als Ergebnis des vermuteten SSH-Logins angenommen.)*

*   **User Flag (vermuteter Pfad):** `c7d0a8de1e03b25a6f7ed2d91b94dad6`
*   **Root Flag (vermuteter Pfad):** `5C42D6BB0EE9CE4CB7E7349652C45C4A`

## Tags

`HackMyVM`, `Democracy`, `Medium`, `Web`, `Apache`, `PHP`, `SQL Injection`, `sqlmap`, `Klartext-Passwörter`, `Information Disclosure`, `Linux`
