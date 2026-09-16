# 🚀 Firebase Firestore Setup – Getränkefundus HBS

Dieses Dokument erklärt dir Schritt für Schritt, wie du in unter 3 Minuten eine kostenlose **Google Firebase Cloud-Datenbank** für den Getränkefundus einrichtest.

---

## 💡 Warum Firebase Firestore?
* **Echtzeit-Synchronisation**: Sobald jemand am Lehrerzimmer-Tablet bucht, aktualisiert sich der Bestand auf allen Lehrer-Smartphones und beim Admin in derselben Sekunde.
* **100 % kostenlos**: Das kostenlose monatliche Kontingent von Google Firebase (50.000 Lesevorgänge / Tag) reicht für das gesamte Schuljahr um ein Vielfaches aus.
* **Kein Server nötig**: Läuft direkt im Browser ohne Installation oder Wartung.
* **Offline-Unterstützung**: Buchungen funktionieren selbst dann, wenn das Schul-WLAN kurzzeitig ausfällt (automatischer Sync, sobald das Netz wieder da ist).

---

## 💡 Du hast bereits bestehende Firebase-Projekte? (Kein neues Projekt nötig!)

Wenn dein kostenloses Kontingent an neuen Firebase-Projekten fast ausgeschöpft ist, ist das **überhaupt kein Problem**:
Du musst **kein neues Firebase-Projekt erstellen**, sondern kannst den Getränkefundus einfach in eines deiner **bereits existierenden Firebase-Projekte** integrieren!

### Wie funktioniert das, ohne dass sich Daten überschreiben?
Cloud Firestore organisiert Daten in sogenannten **Collections (Sammlungen)**. Unterschiedliche Sammlungen sind in Firestore zu 100 % voneinander isoliert.
Damit der Getränkefundus deine bestehenden Daten (z. B. `users`, `products`, `orders`) niemals berührt oder überschreibt, nutzt die App einen **einstellbaren Namensraum / Tabellen-Präfix** (standardmäßig `hbs_`):

* Deine bestehende App nutzt weiter: `users`, `posts`, `daten` ...
* Der Getränkefundus nutzt völlig getrennt: `hbs_drinks` und `hbs_bookings`!

> [!TIP]
> **So bindest du den Fundus in ein bestehendes Projekt ein:**
> 1. Öffne dein bestehendes Projekt in der Firebase Console.
> 2. Gehe auf **Zahnrad (⚙️) → Projekteinstellungen** und kopiere die Web-Konfiguration (`firebaseConfig`).
> 3. Im Getränkefundus unter *Admin → Cloud & PIN → Firebase-Zugangsdaten konfigurieren* einfügen.
> 4. Das Feld **Tabellen-Präfix** steht standardmäßig auf `hbs_` (kannst du beliebig anpassen, z. B. `getraenke_`).
> 5. Speichern und auf **"Standard-Sortiment in Cloud laden (Seed)"** klicken – fertig!

### Firestore-Sicherheitsregeln für geteilte Projekte:
Falls in deinem bestehenden Projekt bereits Sicherheitsregeln aktiv sind, ergänze im Tab *Firestore → Regeln* einfach diese Zeilen für den Getränkefundus:
```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    
    // --- DEINE BESTEHENDEN REGELN BLEIBEN VOLL ERHALTEN ---
    // match /users/{userId} { ... }

    // --- NEU: REGELN NUR FÜR DEN GETRÄNKEFUNDUS ---
    match /hbs_drinks/{docId} {
      allow read, write: if true;
    }
    match /hbs_bookings/{docId} {
      allow read, write: if true;
    }
  }
}
```

---

## 🛠️ Schritt-für-Schritt-Anleitung (Neues Projekt)

### Schritt 1: Kostenloses Firebase-Projekt erstellen
1. Öffne die **[Firebase Console](https://console.firebase.google.com/)** in deinem Browser und melde dich mit einem Google-Konto an.
2. Klicke auf **"Projekt hinzufügen"** (oder *"Projekt erstellen"*).
3. Gib dem Projekt einen Namen, z. B. `getraenkefundus-hbs`.
4. *Google Analytics*: Kannst du deaktivieren (wird für den Getränkefundus nicht benötigt) und auf **"Projekt erstellen"** klicken.

---

### Schritt 2: Cloud Firestore-Datenbank anlegen
1. Klicke im linken Menü auf **"Erstellen" → "Firestore-Datenbank"**.
2. Klicke auf **"Datenbank erstellen"**.
3. **Standort wählen**: Wähle am besten `eur3 (europe-west)` oder `europe-west3 (Frankfurt)` für optimale Ladezeiten in Deutschland.
4. **Sicherheitsregeln**:
   * Für den Einstieg wählst du **"Im Testmodus starten"** (erlaubt 30 Tage lang uneingeschränkten Lese-/Schreibzugriff).
   * Klicke auf **"Aktivieren"**.

> [!TIP]
> **Dauerhafte Sicherheitsregeln für die Schule**:
> Sobald alles läuft, kannst du im Tab *Firestore Datenbank → Regeln* folgende einfache Regel einfügen, damit der Zugriff dauerhaft aktiv bleibt:
> ```javascript
> rules_version = '2';
> service cloud.firestore {
>   match /databases/{database}/documents {
>     match /{document=**} {
>       allow read, write: if true;
>     }
>   }
> }
> ```

---

### Schritt 3: Web-App anlegen & Konfiguration kopieren
1. Klicke links oben neben "Projektübersicht" auf das **Zahnrad-Symbol** (⚙️) → **Projekteinstellungen**.
2. Scrolle ganz nach unten zum Bereich *"Deine Apps"* und klicke auf das **Web-Symbol** (`</>`).
3. Gib als App-Spitznamen z. B. `Getraenkefundus Web` ein und klicke auf **"App registrieren"**.
4. Firebase zeigt dir nun ein Skript mit dem `firebaseConfig`-Objekt an. Kopiere diesen Block:
   ```javascript
   const firebaseConfig = {
     apiKey: "AIzaSy...",
     authDomain: "getraenkefundus-hbs.firebaseapp.com",
     projectId: "getraenkefundus-hbs",
     storageBucket: "getraenkefundus-hbs.firebasestorage.app",
     messagingSenderId: "1234567890",
     appId: "1:1234567890:web:abcdef..."
   };
   ```

---

### Schritt 4: In den Getränkefundus einfügen
1. Öffne die [index.html](file:///Users/nicolekeller/Desktop/Antigravity%20Projekte/GetraenkefundusHBS/index.html) in deinem Browser.
2. Klicke oben rechts auf **"Admin"** (Standard-PIN: `1234`).
3. Wechsle auf den Reiter **"Cloud & PIN"** (oder klicke im Header auf die Status-Pille *Lokaler Speicher*).
4. Klicke auf **"Firebase-Zugangsdaten konfigurieren"**.
5. Füge den eben kopierten Block in das Textfeld ein und klicke auf **"Verbindung speichern & testen"**.
6. Fertig! Die Status-Pille oben springt sofort auf **🟢 Cloud-Sync aktiv**.
7. *Einmaliger Seed*: Klicke im Reiter "Cloud & PIN" auf **"Standard-Sortiment in Cloud laden (Seed)"**, um Wasser, Schorle, Saft, Spezi, Mate und Bionade direkt in die Datenbank zu schreiben.

---

## 🛡️ Was passiert, wenn kein Firebase eingerichtet ist?
Die Anwendung funktioniert **auch ohne Firebase zu 100 %**:
* Daten werden automatisch im lokalen Browserspeicher (**LocalStorage**) abgelegt.
* **Kein Datenverlust bei Seiten-Reloads oder Schließen des Fensters.**
* Sobald du Firebase aktivierst, synchronisiert sich die App mit der Cloud.
