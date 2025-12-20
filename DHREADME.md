# Projektbeskrivning - Excalidraw

## Vad är Excalidraw?

En webbaserad ritapplikation / whiteboard-tool
Designad för att skapa enkla, handritade diagram och skisser
Körs helt i webläsaren, ingen installation behövs
Open-source och gratis att använda
Fokuserad på snabbhet och enkelhet

## Vad är dess syfte/mål?

Möjliggöra snabb visualisering av idéer och koncept
Låta användare enkelt skapa diagram, flödesscheman och mockups
Främja samarbete genom möjlighet att dela diagram i realtid
Tillhandahålla ett enkelt alternativ till komplicerade design-verktyg
Ge utvecklare och designers ett verktyg för snabb brainstorming och planering

## Vad syftar det till att åstadkomma?

- Minska tiden det tar att visualisera idéer från koncept till skiss
- Göra det enkelt för icke-designers att skapa professionella diagram
- Möjliggöra teamsamarbete och idéutbyte visuellt
- Tillhandahålla export-funktioner (PNG, SVG) för användning i andra verktyg

## Huvudfunktioner inkluderar

- Rita grundläggande former (rektanglar, cirklar, linjer, pilar)
- Lägga till text och etiketter
- Färgväljare för former och text
- Ångra/göra om-funktionalitet
- Spara och ladda diagram
- Exportera som PNG eller SVG
- Dela diagram via URL
- Lagra lokalt eller i molnet
- Stöd för flera språk
- Keyboard-shortcuts för snabb arbetsflöde

## Test-körning Resultat

### Sammanfattning
- **Totalt test-filer**: 89 (22 failed, 67 passed)
- **Totalt tests**: 1079
  - Passar: 709
  - Misslyckas: 323
  - Skippade: 46
  - Todo: 1
- **Körtid**: 15.78 sekunder

### Huvudproblem hittad
**Fel**: `localStorage.clear is not a function`
- Uppstår i många test-filer
- Orsak: localStorage är inte rätt konfigurerat i test-miljön
- Detta är ett setup-problem, inte ett problem med själva appen

### Test-filer som misslyckas
- rotate.test.tsx
- selection.test.tsx
- stats.test.tsx
- (och flera andra)

### Analys
Trots felen visar detta att:
- Projektet HÄR omfattande test-coverage (709 tests passar)
- Utvecklarna testar kritiska funktioner (rotation, selection, statistik)
- Problemet är konfiguration, inte kodkvalitet


## Test-körning Resultat - EFTER FIX

### Sammanfattning
- **Test Files**: 2 failed | 87 passed (89 totalt)
- **Tests**: 68 failed | 964 passed | 46 skipped | 1 todo (1079 totalt)
- **Körtid**: 32.11 sekunder

### Vad vi fixade
- **Problem**: localStorage.clear() var inte tillgänglig i test-miljön
- **Lösning**: La till en localStorage-mock i setupTests.ts
- **Resultat**: Gick från 323 misslyckade till 68 misslyckade tester!

### Återstående problem
- De 68 kvarvarande felen är snapshot-test mismatches
- Dessa är **inte kritiska kodfel**, utan ändrade output från tests
- De 39 nya snapshots som skrevs är normalt vid uppdateringar


## Val av filer att titta närmre på blev

- selection.test.tsx - Hur man väljer former
- flip.test.tsx - Hur man vänder/flippar former
- history.test.tsx - Ångra/göra om funktionalitet
- export.test.tsx - Exportera diagram

eftersom de täcker olika funktioner

# Test-analys: selection.test.tsx

## Filöversikt
- **Filnamn**: `packages/excalidraw/tests/selection.test.tsx`
- **Syfte**: Testa hur användaren kan välja/markera former på canvas
- **Omfattning**: ~600+ rader, många test-fall

---

## Test-grupper (describe-block)

### 1. **box-selection**
**Vad testas**: Rektangelval - användaren drar en ruta för att välja flera element samtidigt

**Tests i denna grupp:**
- `should allow adding to selection via box-select when holding shift`
  - **Vad**: Kan man lägga till element till selection genom att hålla shift och dra
  - **Edge case**: Shift-modifier kombinerat med drag
  - **Problem det löser**: Säkerställer att multi-select fungerar korrekt

- `should (de)select element when box-selecting over and out while not holding shift`
  - **Vad**: Avmarkera/markera element utan shift
  - **Edge case**: Dra utan modifier-tangent
  - **Problem det löser**: Enkelt val ska ersätta tidigare val

---

### 2. **inner box-selection**
**Vad testas**: Välja element som är visuellt nästlade inuti andra

**Tests i denna grupp:**
- `selecting elements visually nested inside another`
  - **Problem det löser**: Kan man välja små element inuti större element?
  - **Edge case**: Element med överlappande gränser

- `selecting grouped elements visually nested inside another`
  - **Problem det löser**: Fungerar det när element är grupperade?

- `selecting & deselecting grouped elements visually nested inside another`
  - **Problem det löser**: Kan man både välja och avmarkera grupperade element?

---

### 3. **selection element**
**Vad testas**: Den visuella "selection box" som visas när man drar för att välja

**Tests i denna grupp:**
- `create selection element on pointer down`
  - **Vad**: Skapas en selection-ruta när man klickar?
  - **Problem det löser**: Visuell feedback när man börjar välja
  - **Edge case**: Kontrollerar position och dimensioner

- `resize selection element on pointer move`
  - **Vad**: Växer selection-rutan när man drar musen?
  - **Problem det löser**: Interaktiv visuell feedback

- `remove selection element on pointer up`
  - **Vad**: Försvinner selection-rutan när man släpper musen?
  - **Problem det löser**: Rengöring efter val

---

### 4. **select single element on the scene**
**Vad testas**: Välja enskilda element (inte drag-selection)

**Tests för olika former:**
- `rectangle`, `diamond`, `ellipse`, `arrow`, `arrow escape`
  - **Vad**: Kan man klicka på varje form och välja den?
  - **Problem det löser**: Grundläggande selection för alla formtyper
  - **Edge case**: "arrow escape" - specialfall för pilar

---

### 5. **tool locking & selection**
**Vad testas**: Vad händer när "lock"-verktyget är aktivt

**Test:**
- `should not select newly created element while tool is locked`
  - **Vad**: Nya element ska INTE väljas automatiskt när tool är låst
  - **Problem det löser**: Låter användare skapa många element utan att välja dem en efter en
  - **Edge case**: Testar alla formtyper

---

### 6. **selectedElementIds stability**
**Vad testas**: Att selection-state är stabil och konsistent

**Test:**
- `box-selection should be stable when not changing selection`
  - **Vad**: Om inget ändras, ska selectedElementIds inte ändras
  - **Problem det löser**: Förhindrar onödiga re-renders eller state-ändringar
  - **Varför viktigt**: Performance och stabilitet

---

## Testningsmetodologi

### Verktyg som används:
- **API.createElement()**: Skapar test-element programmatiskt
- **API.setElements()**: Lägger elementen på canvas
- **Pointer (mouse)**: Simulerar musrörelser och klick
  - `mouse.downAt(x, y)` - Börja dra
  - `mouse.move(x, y)` - Flytta under drag
  - `mouse.moveTo(x, y)` - Flytta till exakt position
  - `mouse.up()` - Släpp
- **Keyboard.withModifierKeys()**: Testar shift, ctrl, alt, etc.
- **assertSelectedElements()**: Verifiera att rätt element är valda

### Exempel på test-mönster:
```javascript
// 1. Skapa element
const rect = API.createElement({ type: "rectangle", x: 0, y: 0, ... });

// 2. Lägg på canvas
API.setElements([rect]);

// 3. Simulera användarinteraktion
mouse.downAt(100, 100);
mouse.moveTo(200, 200);
mouse.up();

// 4. Verifiera resultat
assertSelectedElements([rect.id]);
```

---

## Edge Cases som täcks

| Edge Case | Test | Varför viktigt |
|-----------|------|-----------------|
| Multi-select med shift | `box-selection with shift` | Måste kunna välja flera element |
| Nästlade element | `inner box-selection` | Små element inuti större måste gå att välja |
| Låst tool | `tool locking` | Låsning ska förhindra automatisk selection |
| Selection-visualisering | `selection element` | Användare måste se selection-ruta |
| Alla formtyper | `select single element` | Alla former måste kunna väljas |
| State-stabilitet | `selectedElementIds stability` | Ingen onödig re-rendering |

---

## Requirements som uppfylls

✅ **Grundläggande välfunktionalitet**: Element kan väljas genom klick
✅ **Multi-select**: Flera element kan väljas samtidigt
✅ **Box-select**: Dra-baserat val fungerar
✅ **Modifierare**: Shift/Ctrl-kombinationer fungerar
✅ **Visuell feedback**: Selection-ruta visas under drag
✅ **Gruppering**: Grupperade element fungerar
✅ **Performance**: State uppdateras effektivt

---

## Problem/Bugs som löses

1. **Problem**: Användare kan inte välja flera element samtidigt
   - **Lösning**: Multi-select test säkerställer detta fungerar

2. **Problem**: Små element inuti större kan inte väljas
   - **Lösning**: Inner box-selection test validerar detta

3. **Problem**: Låsning av tool fungerar inte
   - **Lösning**: Tool locking test kontrollerar detta

4. **Problem**: Onödiga renders när inget ändras
   - **Lösning**: Stability test förhindrar detta

---

## Styrkor

✅ **Omfattande coverage**: Testar många olika scenarios (single, multi, nested, grouped)
✅ **Realistisk interaktion**: Använder verklig muspekare-simulering (down, move, up)
✅ **Modifierare-tester**: Testar shift, ctrl och andra tangenter
✅ **Multiple formtyper**: Testar rectangle, diamond, ellipse, arrow - inte bara en typ
✅ **Edge cases**: Nästlade element, gruppering, låsta verktyg
✅ **State-validering**: Kontrollerar både selection OCH render-cykler
✅ **Visuell feedback**: Testar att selection-rutan visas korrekt

---

## Svagheter & Förbättringsförslag

### 1. **Saknade testscenarier**

**Problem**: Tester för **tangentbords-baserad selection** saknas helt
- **Varför det är viktigt**: Många användare använder Tab, Enter, Escape för navigation
- **Exempel**: Tab genom elements, Enter för att välja, Escape för att avvälja
- **Påverkan**: Keyboard-användare får sämre erfarenhet utan dessa tester

**Problem**: Ingen testning av **drag-och-drop med flera element**
- **Scenario**: Välj ett element, sedan drag det - ska det förbli valt?
- **Varför viktigt**: Vanligt användarscenario

**Problem**: Saknas tester för **performance med många element**
- **Varför**: Vad händer när canvas har 1000+ element? Blir det långsamt?
- **Påverkan**: Skalbarhet är okänd

---

### 2. **Ofullständig testning av modifier-tangenter**

**Problem**: Bara shift testas, inte ctrl/cmd eller alt
- **Windows**: Ctrl+click för multi-select
- **Mac**: Cmd+click för multi-select
- **Problem**: Cross-platform bugs kan missa
- **Varför viktigt**: Windows och Mac-användare använder olika tangenter

**Problem**: Ingen testning av **kombinerad shift+ctrl**
- **Scenario**: Shift+Ctrl+click - vad ska hända?
- **Varför viktigt**: Komplexa interaktioner kan bli oväntade

---

### 3. **Svaga assertions (verifikationer)**

**Problem**: Tester använder ofta bara `assertSelectedElements()`
- **Vad det visar**: Vilka element är valda
- **Vad det INTE visar**: 
  - Är renderingen uppdaterad?
  - Är UI-elementen (toolbars) uppdaterade?
  - Är selection-färger korrekt?
  - Fungerar delete-knapp nu?

**Problem**: Ingen kontroll av **visual state**
- **Exempel**: Selection-ruta har rätt färg, tjocklek, streck-stil?
- **Varför viktigt**: Användare ser om något är fel

**Problem**: Inte alltid klart **varför** ett test misslyckas
- **Feedback**: "Expected rect1 to be selected" säger inte varför det inte är det
- **Förbättring**: Kunde visa position, z-index, visibility, etc.

---

### 4. **Bristande testning av boundary conditions**

**Problem**: Ingen testning när **selection-rutan är mycket liten**
- **Scenario**: Klicka nästan på samma plats (x, y är nästan samma)
- **Problem**: Kan det skapa en invisibel selection-ruta?

**Problem**: Ingen testning av **negativa koordinater**
- **Scenario**: Dra UTANFÖR canvas (negative x, negative y)
- **Varför viktigt**: Canvas kan ha element långt bort

**Problem**: Saknas test för **dubbla element på samma position**
- **Scenario**: Två element exakt på samma plats - vilken väljs?
- **Varför viktigt**: Vilket element får prioritet? (z-index?)

---

### 5. **Orealistiska test-scenarier**

**Problem**: Ingen testning av **snabb dubbelklick**
- **Varför viktigt**: Användare dubbelklickar ofta för att redigera text

**Problem**: Saknas test för **långsamt drag** (långsammare än normal)
- **Scenario**: Användare drar mycket långsamt - renderas det rätt?
- **Varför viktigt**: Performance optimering kan ta genvägar för snabba drag

**Problem**: Ingen testning av **touch/pekskärm**
- **Varför viktigt**: Mobila användare (iPad, Android) använder touch
- **Påverkan**: Touch-interaktion kan fungera helt olika än mus

---

### 6. **Saknad testning av **undo/redo** med selection**

**Problem**: Ingen test för:
- Välj element → Ångra → var är selection?
- Välj element → Gör om → är selection tillbaka?

**Varför viktigt**: State-hantering mellan selection och history kan krocka

---

### 7. **Saknad documentation/kommentarer**

**Problem**: Många tester har **ingen förklaring av varför**
- **Exempel**: "should (de)select element when..." - vad är use-case?
- **Påverkan**: Ny utvecklare förstår inte varför testet är viktigt

**Problem**: Ingen **documentation för edge cases**
- **Varför**: Nästa utvecklare kan tro det är en bug, inte intention

---

### 8. **Begränsad error-reporting**

**Problem**: När ett test misslyckas, är det svårt att förstå **varför**
- **Exempel**: "Expected [rect1.id] but got []"
  - Var rätt rect? Vad var wrong position?
  - Var det en timing-issue?
  - Var selection-rutan inkonsekventa?

**Förbättring**: Kunde logga:
- Alla element på canvas
- Alla render-cykler
- Alla state-ändringar
- Muörelsernas exakta vägar

---

## Prioritering av förbättringar

**Högsta prioritet** (påverkar många användare):
1. Keyboard-baserad selection (Tab, Enter, Escape)
2. Ctrl/Cmd-click (cross-platform consistency)
3. Touch/pekskärm-testning (mobil)
4. Performance med många element

**Medel prioritet** (edge cases):
5. Undo/redo med selection
6. Negativa koordinater
7. Dubbla elements på samma plats

**Låg prioritet** (polish):
8. Visuell verifikation (färger, stil)
9. Bättre error-meddelanden
10. Dokumentation/kommentarer

---

## Sammanfattning

Denna test-fil är **bra och täcker mycket**, men har några **kritiska gap**:

**Styrka**: Täcker vanliga scenarios väl
**Svaghet**: Saknar keyboard, touch, och cross-platform testning
**Påverkan**: Vissa användargrupper (keyboard users, mobile) får sämre erfarenhet

**Rekommendation**: Lägg till tester för keyboard-baserad selection först - det påverkar mest användare.

## Exploratory Testing - Excalidraw

### Test 1: Single Element Selection
**Vad**: Välja en enskild form
**Steg**:
1. Klicka på rectangle-verktyget
2. Rita en rektangel på canvas
3. Klicka på rektangeln

**Resultat**: Rektangeln blir markerad (blå gräns)
**Förväntat**: Rektangeln blir markerad (blå gräns)
**Bug?**: Nej

### Test 2: Box Selection
**Vad**: Välja flera element genom att dra en ruta
**Steg**:
1. Rita två rektanglar
2. Dra en ruta som omfattar båda
3. Släpp musen

**Resultat**: Båda rektanglarna blir markerade
**Förväntat**: Båda rektanglarna blir markerade
**Bug?**: Nej

### Test 3: Shift-Click Multi Select
**Vad**: Lägga till element till selection med shift
**Steg**:
1. Välj en rektangel (klick)
2. Håll shift och klicka på en annan rektangel

**Resultat**: Båda är nu valda
**Förväntat**: Båda är nu valda
**Bug?**: Nej

### Test 4: Undo/Redo
**Vad**: Ångra och gör om ändringar
**Steg**:

1. Rita en rektangel
2. Tryck Ctrl+Z (eller Cmd+Z på Mac) för att ångra
3. Rektangeln bör försvinna
4. Tryck Ctrl+Y (eller Cmd+Shift+Z) för att göra om
5. Rektangeln bör komma tillbaka

Dokumentera:

Försvann rektangeln när du tryckte Ctrl-Z: Ja
Kom den tillbaka när du gjorde om?: Ja
Fungerade det smidigt?: Ja

### Test 5: Färgändring
Vad: Ändra färg på en form
Steg:

Rita en rektangel
Välj den (klicka på den)
Leta efter en färgväljare (vanligtvis ett färgblock i verktygsraden/panelen)
Klicka på det och välj en ny färg
Kontrolera att rektangeln ändrade färg

Dokumentera:

Såg du färgväljaren? ja
Ändrade rektangeln färg direkt? ja 
Försvann färgen när du avmarkerade? nej

#### Test 6: Radera element
Vad: Ta bort en form
Steg:

Rita en rektangel
Välj den
Tryck Delete eller Backspace
Rektangeln bör försvinna

Dokumentera:

Försvann rektangeln? ja
Försvann det direkt eller behövde du bekräfta? försvann direkt
Kan du ångra det (Ctrl+Z)? Ja

### Test 7: Drag/Flytta element
Vad: Flytta en form genom att dra den
Steg:

Rita en rektangel
Välj den
Dra den till en annan position på canvas
Släpp

Dokumentera:

Följde rektangeln musen när du drog? Ja
Blev den markerad under dragningen? Ja
Hamnade den på rätt plats när du släppte? ja

### Test 8: Resize/Storleksändra
Vad: Ändra storlek på en form
Steg:

Rita en rektangel
Välj den
Du bör se små handtag (prickar) i hörnen eller på sidorna
Dra ett av dessa handtag för att ändra storlek
Släpp

Dokumentera:

Såg du handtagen? Ja
Ändrade rektangeln storlek när du drog? Ja
Blev resultatet vettigt? Ja