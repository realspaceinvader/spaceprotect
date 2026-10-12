# 🚀 Spaceprotect

<p align="center">
  <img src="assets/spaceprotect-logo.png" alt="Spaceprotect logo" width="160">
</p>

Jump to: [🇬🇧 English](#-english) | [🇩🇪 Deutsch](#-deutsch)

[⬇️ Download](https://github.com/YOUR-USERNAME/spaceprotect/releases) · [☕ Support](https://buymeacoffee.com/spaceinvader) · [🌐 spaceinvader.at](https://spaceinvader.at)

---

## 🇬🇧 English

Spaceprotect is a simple **screen-time control for Windows** as a **single, self-contained `Spaceprotect.exe`**. It limits the daily computer time of a Windows standard user account (for example your child's account), works completely offline, and needs no cloud, no registration and no database server.

> **Status:** Version 1.0.0. Please try it first on a spare account or in a virtual machine before relying on it. See [Security Notes & Limitations](#security-notes--limitations).

### Table of Contents

- [How It Works](#how-it-works)
- [Features](#features)
- [Requirements](#requirements)
- [Installation](#installation)
- [Initial Setup](#initial-setup)
- [Admin App](#admin-app)
- [How Time Is Counted](#how-time-is-counted)
- [Data Storage & Security](#data-storage--security)
- [Security Notes & Limitations](#security-notes--limitations)
- [Troubleshooting](#troubleshooting)
- [Uninstall](#uninstall)
- [Build from Source](#build-from-source)
- [Responsible Use](#responsible-use)
- [License & Support](#license--support)

### How It Works

1. As administrator you start `Spaceprotect.exe`, set a password, choose the child account and define a limit for every weekday.
2. Clicking **Install** copies the program to `C:\Program Files\Spaceprotect`, creates a scheduled task that runs as **SYSTEM** at every boot, and registers a small tray icon that starts at every logon.
3. The background process checks every 5 seconds whether the child account is logged in and counts the time. At **10** and **2** minutes remaining the child gets a message.
4. When the daily budget is used up, the session is logged off after a short notice. If the child logs in again that day, the session is logged off immediately.
5. You can grant **extra time for today** at any time. The next day the normal weekly plan applies again.

One file, three roles:

| Start | Role |
|---|---|
| double-click | Admin app (asks for UAC and the password) |
| `--service` | Background process as SYSTEM (scheduled task): counts time, warns, logs off |
| `--agent` | Tray icon at logon: shows that the account is supervised |

### Features

- Single `Spaceprotect.exe`, no installer, no database server, no cloud
- Daily limit **per weekday** (0 – 24 h, or "no limit" for a day)
- **Extra time for today**: +15 / +30 / +60 minutes or a custom value, booked only once per request
- Warnings at 10 and 2 minutes remaining (times and texts editable, placeholder `{RestMinuten}`)
- Automatic logoff when the budget is used up; limited retries, no endless loops
- Runs at boot as SYSTEM, restarts itself on failure, works without an administrator logged in
- **Visible tray icon** in the supervised account, without settings or exit entry
- Password-protected admin app: PBKDF2-SHA256 (600,000 iterations) plus Windows UAC
- Usage overview (7 / 30 days) and event log, service state and diagnostics
- Light / dark theme, German / English interface
- Self-repair: broken data permissions are fixed at start; an unusable database is quarantined so protection keeps running
- No telemetry, no network access, no keylogging, no screenshots

### Requirements

- Windows 10 (version 2004 / build 19041 or newer) or Windows 11, **64-bit**
- A **local standard user account** for the child (not an administrator)
- A separate administrator account for setup
- Nothing else: no Python and no .NET runtime on the target PC

### Installation

1. Download `Spaceprotect.exe` from the [Releases](https://github.com/YOUR-USERNAME/spaceprotect/releases) page.
2. Log in with your **administrator** account and start `Spaceprotect.exe`. Confirm the UAC prompt.
3. Follow [Initial Setup](#initial-setup).

> Windows SmartScreen may warn because the executable is not code-signed. Compare the SHA-256 checksum from the release page or [build it yourself](#build-from-source).

### Initial Setup

1. **Set the admin password** – at least 12 characters, at least three of: lowercase, uppercase, digits, special characters.
2. **Overview** tab → choose the **child account** → **Apply account**. Administrators and the current account are never offered.
3. **Weekly plan** tab → active yes/no, hours and minutes for each day → **Save**.
   - *Inactive* = no limit on that day.
   - *Active with 00:00* = no use allowed.
4. **Messages** tab (optional) → adjust warning times and texts.
5. **Overview** tab → *Service* → **Install**.
6. The top right must show **● running** (a heartbeat is written every 10 seconds).
7. Log out, log in with the child account: the tray icon appears and the time starts counting. Check **Used** in the admin app.

### Admin App

| Tab | Purpose |
|---|---|
| Overview | Child account, limit / extension / used / remaining, progress bar, service control, "Give more time today" |
| Weekly plan | Seven days: active, hours, minutes |
| Messages | Warning times, texts for warning, final warning, time expired, login no longer allowed |
| Log | Usage of the last 7 / 30 days and the event log |
| Settings | Change password, data folder, version, diagnostics, uninstall |

The ☰ menu offers theme (light / dark), language (German / English), info and quit.

### How Time Is Counted

- Time counts while the supervised account has a logged-in session (locked or disconnected sessions included).
- Sleep and hibernate are **not** counted; gaps larger than 30 seconds are logged and skipped.
- Several simultaneous sessions of the same account are counted **once** (intervals are merged).
- The running time is measured with a monotonic clock, so changing the system clock does not add or remove time; large clock jumps are logged.
- Time is split at midnight and assigned to the correct calendar day, including daylight-saving changes.
- Checkpoints are written every 10 seconds. After a crash or power loss at most about 10 seconds are lost.

### Data Storage & Security

Everything lives in the program folder `C:\Program Files\Spaceprotect\`:

| File | Purpose |
|---|---|
| `Spaceprotect.exe` | The program (all three roles) |
| `data\spaceprotect.db` | SQLite database: settings, weekly plan, usage intervals, extensions, event log, password hash |
| `data\logs\spaceprotect.log` | Log of the background process |
| `data\logs\gui-error.log` | Errors of the admin app |

Additionally: scheduled task `Spaceprotect` (SYSTEM, at boot) and the autostart value `Spaceprotect` under `HKLM\Software\Microsoft\Windows\CurrentVersion\Run` for the tray icon.

Permissions: the program folder is read-and-execute for standard users; `data\` is accessible only for SYSTEM and administrators. The admin password is stored only as a salted PBKDF2-SHA256 hash (600,000 iterations, constant-time comparison). Failed password attempts are delayed (2, 4, 8, 16 seconds), after five failures the app closes.

### Security Notes & Limitations

- **Administrators can change or remove everything.** The supervised account must not be an administrator.
- The **tray icon** can be closed by the child via Task Manager; it comes back at the next logon. The background process runs as SYSTEM and cannot be ended by a standard user.
- The admin app can be closed in Task Manager at any time; protection keeps running because it does not depend on it.
- Logoff is performed through the Windows API **without** a "save your work" prompt. Unsaved data in the child's session can be lost – the warnings exist to prevent that.
- The executable is **not code-signed**. Windows SmartScreen or antivirus tools may warn about unknown PyInstaller programs.
- Spaceprotect is a helper for family routines, **not** a tamper-proof security product. Physical access, another operating system or administrator rights bypass it.

### Troubleshooting

| Problem | What to do |
|---|---|
| Status shows "stopped" | *Settings → Diagnostics* shows rights, database and the latest log lines. Click **Repair**, then **Start**. |
| Child account missing in the list | Administrators and the logged-in account are not offered. |
| Time is not counted | Check that the status is **● running** and that the correct child account is applied. |
| Admin app does not start | Run as administrator. Details: `data\logs\gui-error.log`. |
| SmartScreen warning | Click *More info → Run anyway* after verifying the checksum. |

### Uninstall

Admin app → **Uninstall** (top right). Removes the scheduled task, the autostart entry, the program and all data. This cannot be undone.

### Build from Source

```bash
git clone https://github.com/YOUR-USERNAME/spaceprotect.git
cd spaceprotect
pip install -r requirements.txt
python -m pytest tests
python build.py          # creates release\Spaceprotect.exe
```

Requires Python 3.12 on Windows. Libraries: customtkinter, pywin32, pystray, Pillow; PyInstaller for the build.

### Responsible Use

Use Spaceprotect only on devices you own or administer, and talk to your child about why limits exist. Spaceprotect does not record what is done on the computer.

### License & Support

[MIT License](LICENSE). If you like the project: [☕ Buy me a coffee](https://buymeacoffee.com/spaceinvader) · [spaceinvader.at](https://spaceinvader.at)

---

## 🇩🇪 Deutsch

Spaceprotect ist eine einfache **Bildschirmzeit-Begrenzung für Windows** als **einzelne, autarke `Spaceprotect.exe`**. Sie begrenzt die tägliche Computerzeit eines Windows-Standardkontos (zum Beispiel das Konto deines Kindes), arbeitet komplett offline und braucht keine Cloud, keine Registrierung und keinen Datenbankserver.

> **Stand:** Version 1.0.0. Bitte zuerst mit einem Testkonto oder in einer virtuellen Maschine ausprobieren, bevor du dich darauf verlässt. Siehe [Sicherheitshinweise & Grenzen](#sicherheitshinweise--grenzen).

### Inhaltsverzeichnis

- [Funktionsweise](#funktionsweise)
- [Funktionen](#funktionen)
- [Voraussetzungen](#voraussetzungen)
- [Installation](#installation-de)
- [Ersteinrichtung](#ersteinrichtung)
- [Verwaltung](#verwaltung)
- [Wie die Zeit gezählt wird](#wie-die-zeit-gezählt-wird)
- [Datenablage & Sicherheit](#datenablage--sicherheit)
- [Sicherheitshinweise & Grenzen](#sicherheitshinweise--grenzen)
- [Fehlersuche](#fehlersuche)
- [Deinstallation](#deinstallation)
- [Aus dem Quellcode bauen](#aus-dem-quellcode-bauen)
- [Verantwortungsvolle Nutzung](#verantwortungsvolle-nutzung)
- [Lizenz & Unterstützung](#lizenz--unterstützung)

### Funktionsweise

1. Als Administrator startest du `Spaceprotect.exe`, legst ein Passwort fest, wählst das Kinderkonto und definierst für jeden Wochentag ein Limit.
2. Ein Klick auf **Installieren** kopiert das Programm nach `C:\Program Files\Spaceprotect`, legt eine geplante Aufgabe an, die bei jedem Start als **SYSTEM** läuft, und richtet ein kleines Tray-Symbol ein, das bei jeder Anmeldung startet.
3. Der Hintergrundprozess prüft alle 5 Sekunden, ob das Kinderkonto angemeldet ist, und zählt die Zeit. Bei **10** und **2** Minuten Restzeit bekommt das Kind eine Meldung.
4. Ist das Tagesbudget aufgebraucht, wird die Sitzung nach einem kurzen Hinweis abgemeldet. Meldet sich das Kind am selben Tag erneut an, wird sofort abgemeldet.
5. **Heute mehr Zeit** kannst du jederzeit vergeben. Am nächsten Tag gilt wieder der normale Wochenplan.

Eine Datei, drei Aufgaben:

| Start | Aufgabe |
|---|---|
| Doppelklick | Verwaltung (fragt UAC und Passwort ab) |
| `--service` | Hintergrundprozess als SYSTEM (geplante Aufgabe): zählt Zeit, warnt, meldet ab |
| `--agent` | Tray-Symbol bei der Anmeldung: zeigt, dass das Konto überwacht wird |

### Funktionen

- Eine einzige `Spaceprotect.exe`, kein Installer, kein Datenbankserver, keine Cloud
- Tageslimit **je Wochentag** (0 – 24 Std., oder „keine Begrenzung“ für einen Tag)
- **Heute mehr Zeit**: +15 / +30 / +60 Minuten oder eigener Wert, jede Anfrage wird nur einmal gebucht
- Warnungen bei 10 und 2 Minuten Restzeit (Zeiten und Texte änderbar, Platzhalter `{RestMinuten}`)
- Automatische Abmeldung bei aufgebrauchtem Budget; begrenzte Wiederholungen, keine Endlosschleifen
- Läuft beim Hochfahren als SYSTEM, startet sich bei Fehlern neu, funktioniert ohne angemeldeten Administrator
- **Sichtbares Tray-Symbol** im überwachten Konto, ohne Einstellungen und ohne Beenden-Eintrag
- Passwortgeschützte Verwaltung: PBKDF2-SHA256 (600.000 Iterationen) plus Windows-UAC
- Nutzungsübersicht (7 / 30 Tage) und Ereignisprotokoll, Dienststatus und Diagnose
- Heller / dunkler Modus, Oberfläche auf Deutsch / Englisch
- Selbstreparatur: defekte Datenrechte werden beim Start korrigiert; eine unbrauchbare Datenbank wird beiseitegelegt, damit der Schutz weiterläuft
- Keine Telemetrie, kein Netzwerkzugriff, kein Keylogger, keine Screenshots

### Voraussetzungen

- Windows 10 (Version 2004 / Build 19041 oder neuer) oder Windows 11, **64 Bit**
- Ein **lokales Standardkonto** für das Kind (kein Administrator)
- Ein eigenes Administratorkonto für die Einrichtung
- Sonst nichts: kein Python und keine .NET-Laufzeit auf dem Ziel-PC

### Installation (DE)

1. `Spaceprotect.exe` von der Seite [Releases](https://github.com/YOUR-USERNAME/spaceprotect/releases) herunterladen.
2. Mit deinem **Administratorkonto** anmelden und `Spaceprotect.exe` starten. UAC bestätigen.
3. Weiter mit [Ersteinrichtung](#ersteinrichtung).

> Windows SmartScreen kann warnen, weil die EXE nicht code-signiert ist. Vergleiche die SHA-256-Prüfsumme von der Release-Seite oder [baue das Programm selbst](#aus-dem-quellcode-bauen).

### Ersteinrichtung

1. **Verwaltungspasswort festlegen** – mindestens 12 Zeichen, mindestens drei von: Kleinbuchstaben, Großbuchstaben, Ziffern, Sonderzeichen.
2. Tab **Übersicht** → **Kinderkonto** wählen → **Konto übernehmen**. Administratoren und das aktuelle Konto werden nie angeboten.
3. Tab **Wochenplan** → je Tag aktiv ja/nein, Stunden und Minuten → **Speichern**.
   - *Inaktiv* = an diesem Tag keine Begrenzung.
   - *Aktiv mit 00:00* = keine Nutzung erlaubt.
4. Tab **Meldungen** (optional) → Warnzeiten und Texte anpassen.
5. Tab **Übersicht** → *Dienst* → **Installieren**.
6. Oben rechts muss **● läuft** stehen (alle 10 Sekunden wird ein Lebenszeichen geschrieben).
7. Abmelden, mit dem Kinderkonto anmelden: Das Tray-Symbol erscheint und die Zeit wird gezählt. In der Verwaltung **Verbraucht** prüfen.

### Verwaltung

| Tab | Zweck |
|---|---|
| Übersicht | Kinderkonto, Limit / Verlängerung / Verbraucht / Restzeit, Fortschrittsbalken, Dienststeuerung, „Heute mehr Zeit geben“ |
| Wochenplan | Sieben Tage: aktiv, Stunden, Minuten |
| Meldungen | Warnzeiten, Texte für Warnung, Endwarnung, Zeit abgelaufen, Anmeldung nicht mehr erlaubt |
| Protokoll | Nutzung der letzten 7 / 30 Tage und Ereignisprotokoll |
| Einstellungen | Passwort ändern, Datenordner, Version, Diagnose, Deinstallation |

Das ☰-Menü bietet Theme (hell / dunkel), Sprache (Deutsch / English), Info und Beenden.

### Wie die Zeit gezählt wird

- Zeit zählt, solange das überwachte Konto eine angemeldete Sitzung hat (gesperrte oder getrennte Sitzungen eingeschlossen).
- Schlaf- und Ruhezustand zählen **nicht**; Lücken über 30 Sekunden werden protokolliert und übersprungen.
- Mehrere gleichzeitige Sitzungen desselben Kontos werden **einmal** gezählt (Intervalle werden zusammengeführt).
- Die laufende Zeit wird mit einer monotonen Uhr gemessen. Das Verstellen der Systemuhr fügt also keine Zeit hinzu oder nimmt sie weg; große Zeitsprünge werden protokolliert.
- Die Zeit wird um Mitternacht geteilt und dem richtigen Kalendertag zugeordnet, auch bei Sommer-/Winterzeitwechsel.
- Checkpoints werden alle 10 Sekunden gespeichert. Nach einem Absturz oder Stromausfall gehen höchstens etwa 10 Sekunden verloren.

### Datenablage & Sicherheit

Alles liegt im Programmordner `C:\Program Files\Spaceprotect\`:

| Datei | Zweck |
|---|---|
| `Spaceprotect.exe` | Das Programm (alle drei Aufgaben) |
| `data\spaceprotect.db` | SQLite-Datenbank: Einstellungen, Wochenplan, Nutzungsintervalle, Verlängerungen, Ereignisprotokoll, Passwort-Hash |
| `data\logs\spaceprotect.log` | Protokoll des Hintergrundprozesses |
| `data\logs\gui-error.log` | Fehler der Verwaltung |

Zusätzlich: geplante Aufgabe `Spaceprotect` (SYSTEM, beim Start) und der Autostart-Wert `Spaceprotect` unter `HKLM\Software\Microsoft\Windows\CurrentVersion\Run` für das Tray-Symbol.

Rechte: Der Programmordner ist für Standardbenutzer nur lesbar und ausführbar; `data\` ist nur für SYSTEM und Administratoren zugänglich. Das Verwaltungspasswort wird nur als gesalzener PBKDF2-SHA256-Hash gespeichert (600.000 Iterationen, zeitkonstanter Vergleich). Fehlversuche werden verzögert (2, 4, 8, 16 Sekunden), nach fünf Fehlversuchen schließt sich die Verwaltung.

### Sicherheitshinweise & Grenzen

- **Administratoren können alles ändern oder entfernen.** Das überwachte Konto darf kein Administrator sein.
- Das **Tray-Symbol** kann das Kind über den Task-Manager beenden; es erscheint bei der nächsten Anmeldung wieder. Der Hintergrundprozess läuft als SYSTEM und ist für Standardbenutzer nicht beendbar.
- Die Verwaltung lässt sich jederzeit im Task-Manager schließen; der Schutz läuft weiter, weil er nicht von ihr abhängt.
- Die Abmeldung erfolgt über die Windows-API **ohne** Rückfrage zum Speichern. Ungespeicherte Daten in der Sitzung des Kindes können verloren gehen – die Warnungen sollen das verhindern.
- Die EXE ist **nicht code-signiert**. Windows SmartScreen oder Virenscanner können vor unbekannten PyInstaller-Programmen warnen.
- Spaceprotect ist ein Hilfsmittel für Familienregeln, **kein** manipulationssicheres Sicherheitsprodukt. Physischer Zugriff, ein anderes Betriebssystem oder Administratorrechte umgehen es.

### Fehlersuche

| Problem | Was tun |
|---|---|
| Status zeigt „gestoppt“ | *Einstellungen → Diagnose* zeigt Rechte, Datenbank und die letzten Protokollzeilen. **Reparieren** klicken, dann **Starten**. |
| Kinderkonto fehlt in der Liste | Administratoren und das angemeldete Konto werden nicht angeboten. |
| Zeit wird nicht gezählt | Prüfen, ob der Status **● läuft** anzeigt und das richtige Kinderkonto übernommen ist. |
| Verwaltung startet nicht | Als Administrator starten. Details: `data\logs\gui-error.log`. |
| SmartScreen-Warnung | Nach Prüfung der Prüfsumme auf *Weitere Informationen → Trotzdem ausführen* klicken. |

### Deinstallation

Verwaltung → **Deinstallieren** (oben rechts). Entfernt die geplante Aufgabe, den Autostart-Eintrag, das Programm und alle Daten. Das kann nicht rückgängig gemacht werden.

### Aus dem Quellcode bauen

```bash
git clone https://github.com/YOUR-USERNAME/spaceprotect.git
cd spaceprotect
pip install -r requirements.txt
python -m pytest tests
python build.py          # erzeugt release\Spaceprotect.exe
```

Benötigt Python 3.12 unter Windows. Bibliotheken: customtkinter, pywin32, pystray, Pillow; PyInstaller zum Bauen.

### Verantwortungsvolle Nutzung

Nutze Spaceprotect nur auf Geräten, die dir gehören oder die du verwaltest, und sprich mit deinem Kind darüber, warum es Grenzen gibt. Spaceprotect zeichnet nicht auf, was am Computer gemacht wird.

### Lizenz & Unterstützung

[MIT-Lizenz](LICENSE). Wenn dir das Projekt gefällt: [☕ Buy me a coffee](https://buymeacoffee.com/spaceinvader) · [spaceinvader.at](https://spaceinvader.at)
