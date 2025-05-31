# Quick - HackMyVM (Easy)

![quick.png](quick.png)

## Übersicht

*   **VM:** Quick
*   **Plattform:** [HackMyVM](https://hackmyvm.eu/machines/machine.php?vm=Quick)
*   **Schwierigkeit:** Easy
*   **Autor der VM:** DarkSpirit
*   **Datum des Writeups:** 31. Mai 2025
*   **Original-Writeup:** https://alientec1908.github.io/Quick_HackMyVM_Easy/
*   **Autor:** Ben C.

## Kurzbeschreibung

Die Challenge "Quick" auf der Plattform HackMyVM ist eine als "Easy" eingestufte virtuelle Maschine. Ziel war es, Root-Rechte auf dem System zu erlangen. Der Lösungsweg umfasste die Identifizierung eines Webservers, die Ausnutzung einer Local File Inclusion (LFI)-Schwachstelle in einem PHP-Skript zur Remote Code Execution (RCE) und anschließende Privilege Escalation durch ein falsch konfiguriertes SUID-Binary (PHP).

## Disclaimer / Wichtiger Hinweis

Die in diesem Writeup beschriebenen Techniken und Werkzeuge dienen ausschließlich zu Bildungszwecken im Rahmen von legalen Capture-The-Flag (CTF)-Wettbewerben und Penetrationstests auf Systemen, für die eine ausdrückliche Genehmigung vorliegt. Die Anwendung dieser Methoden auf Systeme ohne Erlaubnis ist illegal. Der Autor übernimmt keine Verantwortung für missbräuchliche Verwendung der hier geteilten Informationen. Handeln Sie stets ethisch und verantwortungsbewusst.

## Verwendete Tools

*   `arp-scan` (impliziert)
*   `nmap`
*   `curl`
*   `nikto`
*   `gobuster`
*   `wfuzz`
*   `feroxbuster`
*   `php_filter_chain_generator.py` (benutzerdefiniertes Skript)
*   `nc` (Netcat)
*   Standard Linux-Befehle (`ls`, `cat`, `find`, `php`, etc.)

## Lösungsweg (Zusammenfassung)

Der Angriff auf die Maschine "Quick" gliederte sich in folgende Phasen:

1.  **Reconnaissance & Enumeration:**
    *   Identifizierung der Ziel-IP (`192.168.2.208`) und des Hostnamens (`quick.hmv`) mittels ARP-Scan und manueller Zuweisung.
    *   Portscan mit Nmap enthüllte einen offenen Port 80 (HTTP) mit einem Apache-Webserver Version 2.4.41 unter Ubuntu. Betriebssystem als Linux identifiziert.

2.  **Web Enumeration & Schwachstellensuche:**
    *   Bestätigung des Webservers mit `curl`.
    *   `nikto` identifizierte fehlende Sicherheitsheader, Directory Indexing in `/images/`, eine exponierte `phpinfo()`-Seite in `index.php` und eine kritische potenzielle Remote File Inclusion (RFI) im Parameter `page` von `index.php`.
    *   `gobuster` und `feroxbuster` fanden diverse PHP-Seiten (`index.php`, `contact.php`, `about.php`, `home.php`, `cars.php`) und bestätigten das Directory Listing.

3.  **Initial Access (LFI zu RCE):**
    *   `wfuzz` bestätigte eine Local File Inclusion (LFI) im Parameter `page` von `index.php` durch erfolgreiches Lesen von `/etc/passwd`.
    *   Mittels `php_filter_chain_generator.py` wurde ein Payload erstellt, um die LFI zu Remote Code Execution (RCE) zu eskalieren. Der Befehl `id` wurde erfolgreich als `www-data` ausgeführt.
    *   Eine Bash-Reverse-Shell wurde über den RCE-Vektor als Benutzer `www-data` etabliert.

4.  **Post-Exploitation / Privilege Escalation (von `www-data` zu `root`):**
    *   Nach Erhalt der Shell als `www-data` wurde eine systeminterne Enumeration durchgeführt.
    *   Der Befehl `find / -type f -perm -4000 -ls 2>/dev/null` identifizierte `/usr/bin/php7.0` als SUID-Root-Binary.

5.  **Privilege Escalation (von `www-data` zu root):**
    *   Die SUID-Berechtigung von `/usr/bin/php7.0` wurde ausgenutzt mit dem Befehl `php7.0 -r 'pcntl_exec("/bin/bash", ["-p"]);'`, um eine Shell mit effektiven Root-Rechten (`euid=0`) zu erhalten.

## Wichtige Schwachstellen und Konzepte

*   **Local File Inclusion (LFI) zu Remote Code Execution (RCE):** Eine Schwachstelle im `page`-Parameter von `index.php` erlaubte das Inkludieren lokaler Dateien. Durch die Verwendung von PHP-Filtern (`php://filter`) und einem Payload-Generator (`php_filter_chain_generator.py`) konnte dies zu RCE eskaliert werden, um Befehle als `www-data`-Benutzer auszuführen.
*   **SUID Misconfiguration (PHP SUID Root):** Das PHP-Binary `/usr/bin/php7.0` war mit SUID-Root-Rechten konfiguriert. Dies ist eine schwere Fehlkonfiguration und ermöglichte es, durch Ausführen eines einfachen PHP-Befehls (`pcntl_exec`) eine Root-Shell zu erlangen.
*   **Web Enumeration:** Einsatz verschiedener Tools (Nikto, Gobuster, Feroxbuster, Wfuzz) zur Identifizierung von Webanwendungsstruktur, -technologien und -schwachstellen.

## Flags

*   **User Flag (`/home/andrew/user.txt`):** `HMV{QUICK-user}`
*   **Root Flag (`/root/root.txt`):** `HMV{6ff5f1b9238a96b3c3871c67a215ec80}`

## Tags

`HackMyVM`, `Quick`, `Easy`, `LFI`, `RCE`, `SUID`, `PHP Filter Chains`, `Linux`, `Web`, `Privilege Escalation`
