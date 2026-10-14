# SAP Projektradar: Anbindung des Recherche-Claudes

Dashboard: https://claude.ai/artifact/JDLESw8J8gVSgPWf1fww7d

Hänge **einen** der beiden Bausteine an die bestehende Anweisung des Recherche-Claudes an.

- **Weg A** für Claude Code oder Cowork, also überall dort, wo Claude die Artifact-Tools hat. Claude schreibt direkt ins Dashboard.
- **Weg B** für den normalen Claude-Chat. Claude gibt einen JSON-Block aus, den du im Dashboard unter „Anbindung → Recherche einfügen“ einfügst.

Den Bearbeitungsstand (Neu, Vorgemerkt, Profil gesendet, Im Gespräch, Zusage, Verworfen), deine Notizen und welche Kandidaten du angeboten hast, speichert das Dashboard in einer eigenen Collection `status`, je Ausschreibung. Neue Recherchen überschreiben sie nicht.

Passt dieselbe Ausschreibung auf mehrere Berater, erkennt das Dashboard sie am Link (Tracking-Parameter werden ignoriert, ersatzweise Plattform + `referenz`) und zeigt sie einmal mit allen passenden Beratern. Die Mail an den Ansprechpartner bietet dann alle gewählten Kandidaten namentlich in einer Nachricht an. Der Recherche-Claude muss den Link deshalb vollständig und unverändert übernehmen.

---

## Weg A: direkt schreiben

```
## Ablage im Dashboard „SAP Projektradar“
Schreibe nach jeder Recherche die Ergebnisse zusätzlich in das Artefakt
https://claude.ai/artifact/JDLESw8J8gVSgPWf1fww7d
mit dem Tool ArtifactData (action "batch", ein Aufruf mit allen Dokumenten).

1. Collection "recherchen", doc_id "JJJJ-MM-TT_hhmm_nachname" (nur a–z, 0–9, "-" und "_"). Felder:
   berater {name, rolle, module[], seniority, verfuegbarAb (JJJJ-MM-TT oder "sofort"), auslastung, remote, region},
   zeitraum, erstelltAm (ISO-Zeitstempel), plattformen[],
   zusammenfassung {fazit, top3[], hoheErfolgswahrscheinlichkeit[], risiken[]},
   alternativen[] (alternative Suchbegriffe, Module, Rollen – vor allem bei wenigen Treffern).
   top3 und hoheErfolgswahrscheinlichkeit enthalten die doc_ids der Projekte.
2. Je Projekt ein Dokument in Collection "projekte", doc_id "<recherche-doc_id>__01", "__02" … nach Score absteigend. Felder:
   rechercheId, titel, kunde, einsatzort, land (DE/AT/CH …), remoteAnteil (Zahl 0–100, null wenn nicht genannt),
   remoteHinweis, veroeffentlicht (JJJJ-MM-TT), laufzeit, start (JJJJ-MM-TT oder "ASAP"), passung, anforderungen[],
   ansprechpartner {anrede ("Herr"/"Frau", nur wenn genannt), name, email, telefon}, link, plattform,
   referenz (Projektnummer der Plattform, falls vorhanden), s4hana (true/false), score (0–10, eine Nachkommastelle).
3. Die Collection "status" nie anfassen. Dort pflegt der Ressourcenmanager den Bearbeitungsstand.

Den Link immer vollständig und unverändert aus der Ausschreibung übernehmen: Passt dieselbe Ausschreibung
auf mehrere Berater, fasst das Dashboard sie über den Link zu einer Ausschreibung mit einer gemeinsamen Mail zusammen.
Mehr als 49 Projekte auf mehrere batch-Aufrufe verteilen. Fehlende Angaben weglassen statt zu raten.
Die Textantwort mit Management-Zusammenfassung und Projektübersicht bleibt wie gehabt.
```

---

## Weg B: JSON-Block zum Einfügen

````
## Export für das Dashboard „SAP Projektradar“
Gib am Ende jeder Antwort zusätzlich einen JSON-Codeblock in genau diesem Aufbau aus.
Regeln: fehlende Angaben weglassen statt zu raten · remoteAnteil als Zahl 0–100 oder null (nicht genannt)
· Datumswerte als JJJJ-MM-TT · score 0–10 mit einer Nachkommastelle · Projekte nach Score absteigend
· top3 und hoheErfolgswahrscheinlichkeit verweisen über "nr" auf die Projekte
· link vollständig und unverändert aus der Ausschreibung übernehmen (darüber bündelt das Dashboard
  dieselbe Ausschreibung für mehrere Berater).

```json
{
  "recherche": {
    "berater": {
      "name": "Vorname Nachname",
      "rolle": "SAP FI/CO Consultant",
      "module": ["FI", "CO"],
      "seniority": "Senior",
      "verfuegbarAb": "2026-11-01",
      "auslastung": "100 %",
      "remote": "mind. 80 % remote",
      "region": "DACH, kein CH"
    },
    "zeitraum": "letzte 3 Tage",
    "erstelltAm": "2026-10-14T09:30:00+02:00",
    "plattformen": ["Freelancermap", "Hays", "Westhouse", "K2 Partnering", "Krongaard"],
    "zusammenfassung": {
      "fazit": "Ein bis zwei Sätze zur Lage.",
      "top3": [1, 2, 3],
      "hoheErfolgswahrscheinlichkeit": [1],
      "risiken": ["…"]
    },
    "alternativen": ["…"]
  },
  "projekte": [
    {
      "nr": 1,
      "titel": "…",
      "kunde": "…",
      "einsatzort": "…",
      "land": "DE",
      "remoteAnteil": 80,
      "remoteHinweis": "…",
      "veroeffentlicht": "2026-10-13",
      "laufzeit": "…",
      "start": "2026-11-01",
      "passung": "…",
      "anforderungen": ["…", "…"],
      "ansprechpartner": { "anrede": "Herr", "name": "…", "email": "…", "telefon": "…" },
      "link": "https://…",
      "plattform": "Freelancermap",
      "referenz": "…",
      "s4hana": true,
      "score": 8.5
    }
  ]
}
```
````
