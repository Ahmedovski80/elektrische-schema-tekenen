# Rapport: Starter-app voor elektrische schema's tekenen

Datum: 2025-10-19  
Auteur: Ahmedovski80 (projectstarter) / Opgesteld door Copilot

## 1. Samenvatting
Dit document beschrijft het starterproject "Schematic Starter" — een eenvoudige webapp (React + TypeScript + react‑konva) waarmee je elektrische schema's kunt tekenen. Het bevat doel, scope, MVP‑features, gebruikte technologieën, architectuur, data‑model, hoe je het lokaal draait, en voorgestelde vervolgstappen.

## 2. Doel en scope
Doel: Een werkend prototype leveren waarmee gebruikers componenten (bv. weerstand, batterij) naar een canvas kunnen slepen, pins met draden kunnen verbinden, het schema kunnen opslaan als JSON en kunnen exporteren als PNG.

Scope MVP:
- Sidebar met minimaal 2 componenten (weerstand, batterij)
- Drag & drop naar canvas
- Pins zichtbaar en klik/drag om draad te tekenen
- Save / Load JSON
- Export PNG

Niet in scope (MVP): simulatie, cloud-sync, uitgebreide componentbibliotheek, orthogonaal routeren (kan later toegevoegd worden).

## 3. Gebruikte technologieën
- Front-end: React + TypeScript
- Canvas/tekenlaag: react-konva (Konva)
- UUID: uuid
- Build tooling: Vite
- Data opslag (MVP): local JSON (download/upload)

Reden van keuze:
- Web (React) is snel te delen en eenvoudig te starten.
- Konva biedt objectmodel voor shapes, drag/drop en export functionaliteit.

## 4. Architectuur & belangrijke onderdelen
- UI: drie hoofdzones
  - Linkerzijbalk: componentbibliotheek (drag sources)
  - Canvas: Stage/Layer (react-konva) — hier staan componenten en wires
  - Rechterpaneel: korte instructie / properties (basis)
- State: lokale React-state in CanvasEditor voor components[] en wires[].
- Serialisatie: JSON-structuur met components en wires (zie 6).
- Export: stage.toDataURL() -> PNG

Component lifecycle (kort):
- Sleep item vanaf Sidebar -> drop op canvas -> nieuw ComponentInstance toegevoegd
- Klik/drag vanaf een pin -> tekenen van tijdelijke lijn -> loslaten op pin -> persistente Wire maken
- Verplaatsen component -> component coords updaten; bestaande wires blijven met absolute punten (vereist later updaten van connected wire punten als component verplaatst wordt)

## 5. Belangrijke ontwerpkeuzes en beperkingen
- Pins zijn gedefinieerd relatief aan het component origin (x,y offsets) — handig voor berekening absolute pinposities.
- Wires worden voorlopig als simpele polyline (array met punten) opgeslagen; geen automatische routering.
- Bij verplaatsen van een component worden connecties niet automatisch herberekend in dit MVP (verbetering: bij drag update alle wire punten die naar de pin verwijzen).
- Undo/redo niet geïmplementeerd in MVP; kan eenvoudig toegevoegd worden met history snapshots of command pattern.

## 6. Data‑model (voorbeeld)
Voorbeeld component-template (componentTemplates):
```json
{
  "resistor": {
    "width": 80,
    "height": 24,
    "renderName": "Weerstand",
    "pins": [
      {"name":"A","x":0,"y":12},
      {"name":"B","x":80,"y":12}
    ]
  }
}
```

Voorbeeld opgeslagen schema (JSON):
```json
{
  "components": [
    {"uid":"c1","type":"resistor","x":100,"y":80,"rotation":0,"properties":{"value":"4.7k"}},
    {"uid":"c2","type":"resistor","x":300,"y":80,"rotation":0,"properties":{"value":"1k"}}
  ],
  "wires": [
    {"uid":"w1","from":{"component":"c1","pin":"B"},"to":{"component":"c2","pin":"A"},"points":[200,80]}
  ]
}
```

Notitie: In het starterproject worden wire.points als een flat number-array (x1,y1,x2,y2,...) opgeslagen.

## 7. Bestanden in het starterproject (kort overzicht)
- package.json — dependencies + scripts (vite)
- tsconfig.json — TypeScript config
- index.html — app container
- src/main.tsx — React entry
- src/App.tsx — hoofd-app, sidebar, save/load UI
- src/components/Sidebar.tsx — drag sources
- src/components/CanvasEditor.tsx — Stage/Layer, component rendering, wire tekenen, export PNG, import/export schema
- src/componentTemplates.ts — templates met pinposities voor componenttypes
- src/types.ts — Types/Interfaces (ComponentInstance, Wire, SavedSchema)
- src/utils/serialization.ts — exportJSON / importJSON
- src/styles.css — basis opmaak
- README.md — korte projectuitleg

(Deze bestanden zijn bedoeld als kopieerbare starterbestanden die lokaal kunnen draaien.)

## 8. Hoe lokaal draaien (kort)
1. Node.js installeren (v18+ aanbevolen)
2. Projectmap aanmaken en bestanden plaatsen
3. In projectmap uitvoeren:
   - npm install
   - npm run dev
4. Open de URL die Vite toont (bv. http://localhost:5173)

## 9. Testen en acceptatiecriteria (voor een eenvoudig cursusrapport)
- Functionaliteit:
  - Componenten slepen naar canvas werkt.
  - Klik/drag vanaf een pin maakt een stijgende tijdelijke lijn.
  - Loslaten op andere pin creëert een blijvende draad.
  - Save JSON produceert valide JSON dat opnieuw kan worden geladen.
  - Export PNG downloadt een afbeelding.
- Usability:
  - Pins zijn zichtbaar en klikbaar.
  - Componenten zijn verplaatsbaar.
- Codekwaliteit (basis):
  - Types gebruikt (TypeScript).
  - Logische bestandsscheiding (components, utils, types).

## 10. Vervolgstappen en uitbreidingen (prioriteit)
1. Wire‑update bij component‑drag (verbindende wires automatisch verschuiven) — hoge prioriteit
2. Orthogonaal/Manhattan routering voor nette rechte draden — hoge prioriteit
3. Undo/Redo — middelmatig
4. Grid en snap‑to‑grid, rotatie van componenten — middelmatig
5. Eigenschappenpaneel per component (waarde, footprint, identifier) — middelmatig
6. Export naar SVG en netlist (voor simulatie/SPICE) — laag
7. Cloud‑sync / realtime delen (Firebase/Firestore) — optioneel
8. Grotere IEC symbolen‑bibliotheek en import van SVG symbolen — optioneel

## 11. Mogelijke verbeteringen voor een beoordeelbaar project (voor school/portfolio)
- Voeg user stories en acceptance tests toe.
- Maak een korte video/demonstratie van de app.
- Host het prototype op Netlify / Vercel en link naar live demo.
- Maak automatische unit tests voor serialisatie en component-helpers.

## 12. Bijlagen — korte codevoorbeelden
- Start van een draad vanaf pin (concept, React/Konva):
```ts
// startWire(compId, pin, x, y)
const uid = uuidv4();
setDrawingWire({ uid, from: { component: compId, pin }, points: [x, y, x, y] });
```

- Export JSON (utils):
```ts
export function exportJSON(schema: SavedSchema) {
  return JSON.stringify(schema, null, 2);
}
```

## 13. Conclusie
Het starterproject is een compact en uitvoerbaar prototype om elektrische schema's te tekenen. Het biedt de kernfunctionaliteit (drag/drop, pins, wires, save/load, export) en vormt een stabiele basis om functies als orthogonaal routeren, undo/redo, en cloud sync toe te voegen. Dit rapport kan gebruikt worden als onderdeel van een projectverslag of als README uitgebreid document.

---

Wil je dat ik:
- dit rapport omzet naar een PDF (ik kan Markdown genereren; conversie naar PDF zul je lokaal doen of ik kan instructies geven), of
- dit rapport direct in een README.md in een GitHub‑repo zet en de bestanden push (ik kan een repo opzetten en de code toevoegen)?

Laat me weten welke van deze je wilt en ik maak het meteen klaar.
