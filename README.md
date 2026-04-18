# Armaturenprüfung

Eine Web-App zur digitalen Prüfung und Dokumentation von Armaturen – gehostet auf **GitHub Pages**, Datenbank auf **Supabase**.

---

## 🚀 Schnellstart

### 1. GitHub Pages einrichten

1. Dieses Repository forken oder klonen
2. Unter **Settings → Pages** `main` Branch und `/ (root)` auswählen
3. Die App ist erreichbar unter `https://<dein-username>.github.io/<repo-name>/`

---

### 2. Supabase Projekt erstellen

1. Auf [supabase.com](https://supabase.com) einloggen und ein neues Projekt anlegen
2. Im **SQL Editor** folgendes ausführen:

```sql
-- Armaturen-Tabelle
CREATE TABLE armaturen (
  id          uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  nummer      text NOT NULL UNIQUE,
  ort         text NOT NULL,
  stockwerk   text NOT NULL,
  bezeichnung text NOT NULL,
  created_at  timestamptz DEFAULT now()
);

-- Prüfungen-Tabelle
CREATE TABLE pruefungen (
  id                uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  armatur_id        uuid REFERENCES armaturen(id) ON DELETE CASCADE,
  pruefer           text NOT NULL,
  ist_zustand       text NOT NULL,   -- 'offen' | 'geschlossen'
  in_ordnung        boolean NOT NULL,
  notizen           text,
  unterschrift_url  text,
  bild_url          text,
  geprueft_am       timestamptz DEFAULT now()
);

-- Storage Bucket für Bilder & Unterschriften
INSERT INTO storage.buckets (id, name, public)
VALUES ('pruefungen', 'pruefungen', true);

-- Row Level Security (öffentlicher Zugriff – für interne Nutzung OK)
ALTER TABLE armaturen ENABLE ROW LEVEL SECURITY;
ALTER TABLE pruefungen ENABLE ROW LEVEL SECURITY;
CREATE POLICY "public all" ON armaturen FOR ALL USING (true);
CREATE POLICY "public all" ON pruefungen FOR ALL USING (true);
CREATE POLICY "public upload" ON storage.objects
  FOR ALL USING (bucket_id = 'pruefungen');
```

3. **Project URL** und **anon public key** aus **Settings → API** kopieren

---

### 3. App verbinden

Beim ersten Aufruf der App erscheint ein Setup-Dialog.  
Dort URL und Anon-Key eintragen → **Verbinden**.

Die Zugangsdaten werden im `localStorage` des Browsers gespeichert.

---

## 📋 Armaturen-Nummerierung

| Präfix | Bedeutung | Soll-Zustand |
|--------|-----------|--------------|
| `LO`   | Locked Open – dauerhaft geöffnet | Offen |
| `LC`   | Locked Closed – dauerhaft gesperrt | Geschlossen |
| `NO`   | Normally Open – normalerweise offen | Offen |
| `NC`   | Normally Closed – normalerweise geschlossen | Geschlossen |

Die App prüft bei jeder Inspektion automatisch, ob der **Ist-Zustand** dem **Soll-Zustand** entspricht.

---

## 📱 Features

- ✅ Armaturen verwalten (anlegen, bearbeiten, löschen)
- ✅ Prüfung mit Ist-Zustand + automatischem OK/NOK-Check
- ✅ Digitale Unterschrift (Finger/Maus)
- ✅ Foto direkt von der Handykamera oder Datei-Upload
- ✅ Prüfverlauf je Armatur
- ✅ Filter: Alle / OK / NOK / Ungeprüft
- ✅ Volltext-Suche
- ✅ Mobiloptimiert (PWA-ready)
