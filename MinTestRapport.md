# Excalidraw - Testning och Analys Rapport

**Författare**: Dan-Håkan Davall
**Datum**: December 2025  
**Projekt**: Undersökning av Excalidraw open-source projekt  
**Fokus**: Testing med Vitest/Jest

---

## 1. Projektbeskrivning

### Vad är Excalidraw?

Excalidraw är en webbaserad ritapplikation (whiteboard-tool) designad för att skapa enkla, handritade diagram och skisser direkt i webläsaren. Appen är helt gratis, open-source och kräver ingen installation.

### Syfte och mål

Projektets syfte är att:
- Möjliggöra snabb visualisering av idéer och koncept utan komplicerade designverktyg
- Tillhandahålla ett enkelt verktyg för brainstorming och planering
- Främja samarbete genom möjlighet att dela diagram i realtid
- Ge utvecklare och designers ett tillgängligt alternativ för snabb skissning

### Huvudfunktioner

- Rita grundläggande former (rektanglar, cirklar, linjer, pilar)
- Lägga till text och etiketter
- Färgväljare för former och text
- Ångra/göra om-funktionalitet
- Spara och ladda diagram
- Exportera som PNG eller SVG
- Dela diagram via URL
- Lagring lokalt eller i molnet
- Stöd för flera språk
- Keyboard-shortcuts för effektiv arbetsflöde

---

## 2. Köra befintliga tests

### Problem upptäckt

Vid första test-körning mot:
```bash
yarn test:app --run
```

**Resultat innan fix:**
- **Test Files**: 22 failed | 67 passed (89 totalt)
- **Tests**: 323 failed | 709 passed | 46 skipped | 1 todo (1079 totalt)
- **Körtid**: 15.78 sekunder

**Huvudproblem**: `TypeError: localStorage.clear is not a function`

Detta fel uppstod i många test-filer:
- `rotate.test.tsx`
- `selection.test.tsx`
- `stats.test.tsx`
- Och flera andra

**Orsak**: Vitest-miljön hade inte localStorage korrekt konfigurerat i setupfilen.

### Lösning implementerad

Lägg till localStorage-mock i `setupTests.ts`:

```typescript
const localStorageMock = (() => {
  let store: Record<string, string> = {};

  return {
    getItem: (key: string) => store[key] || null,
    setItem: (key: string, value: string) => {
      store[key] = value.toString();
    },
    removeItem: (key: string) => {
      delete store[key];
    },
    clear: () => {
      store = {};
    },
  };
})();

Object.defineProperty(window, "localStorage", {
  value: localStorageMock,
});
```

### Resultat efter fix

```bash
yarn test:app --run
```

**Resultat efter fix:**
- **Test Files**: 2 failed | 87 passed (89 totalt) ✅
- **Tests**: 68 failed | 964 passed | 46 skipped | 1 todo (1079 totalt) ✅
- **Körtid**: 32.11 sekunder

**Framsteg**: 301 tests fixades! 🎉

**Kvarvarande problem**: 68 snapshot-test mismatches (inte kritiska kodfel, utan ändrad output från tester)

---

## 3. Analys av befintliga tests

### Fokus: selection.test.tsx

Denna test-fil testar en av Excalidraw's mest kritiska funktioner: **hur användare väljer/markerar former**.

#### Test-grupper

**box-selection**: Testa rektangelval (drag för att välja flera element)
- Test: "should allow adding to selection via box-select when holding shift"
  - Verifiera att multi-select fungerar med shift-modifier
  
- Test: "should (de)select element when box-selecting over and out while not holding shift"
  - Verifiera att enkelt val ersätter tidigare val

**inner box-selection**: Testa väljning av nästlade element
- Välja element som ligger visuellt inuti andra element
- Välja grupperade element inuti större element

**selection element**: Testa visualiseringen av selection-ruta
- Skapas selection-ruta när man klickar?
- Växer den när man drar musen?
- Försvinner den när man släpper?

**select single element on the scene**: Testa väljning av enskilda element
- Testar alla formtyper: rectangle, diamond, ellipse, arrow

**tool locking & selection**: Testa interaktion med låst verktyg
- Nya element ska INTE väljas automatiskt när tool är låst

**selectedElementIds stability**: Testa state-stabilitet
- Selection-state ska inte ändras om inget förändras

#### Styrkor i denna test-fil

✅ Omfattande coverage av vanliga use-cases  
✅ Realistisk mussimulering (down, move, up)  
✅ Testar modifier-tangenter (shift)  
✅ Testar flera formtyper, inte bara en  
✅ Edge cases som nästlade element och gruppering  
✅ Kontrollerar både selection OCH render-cykler  

#### Svagheter och förbättringsmöjligheter

**Saknad keyboard-testning**
- Problem: Inga tester för Tab, Enter, Escape navigation
- Påverkan: Keyboard-användare får sämre erfarenhet
- Prioritet: Högt

**Saknad touch/mobile-testning**
- Problem: Pekskärm-användare inte täckta
- Påverkan: Mobile-användare får sämre erfarenhet
- Prioritet: Högt

**Saknad cross-platform testning**
- Problem: Bara shift testas, inte Ctrl (Windows) eller Cmd (Mac)
- Påverkan: Windows och Mac-användare använder olika tangenter
- Prioritet: Medel

**Svaga assertions**
- Problem: Tester kontrollerar bara vilka element som är valda, inte visuell feedback
- Saknar: Färger, UI-uppdateringar, funktionstillgänglighet
- Prioritet: Medel

**Saknad performance-testning**
- Problem: Vet inte vad som händer med 1000+ element
- Påverkan: Skalbarhet är okänd
- Prioritet: Låg

**Bristande boundary condition-testning**
- Saknas: Mycket små selection-rutor, negativa koordinater, dubbla element på samma position
- Prioritet: Låg

**Saknad undo/redo-testning med selection**
- Problem: Vad händer med selection när man angår/gör om?
- Prioritet: Medel

#### Rekommendationer för förbättring

**Högsta prioritet** (påverkar många användare):
1. Lägg till keyboard-baserad selection (Tab, Enter, Escape)
2. Lägg till Ctrl/Cmd-click testning (cross-platform)
3. Lägg till touch/pekskärm-testning

**Medel prioritet** (edge cases):
4. Lägg till visuell verifikation (färger, stilar)
5. Lägg till undo/redo med selection
6. Lägg till negativa koordinater testning

**Låg prioritet** (optimering):
7. Performance-testning med många element
8. Bättre error-reporting
9. Dokumentation av varför varje test är viktigt

---

## 4. Exploratory Testing

### Testningsmetodologi

Manual testning av själva appen genom webben (http://localhost:3000). Testad:
- Selection-funktionalitet
- Ritning och redigering
- Undo/Redo
- Färg och styling
- Delete/radera
- Drag och resize

### Test-resultat

#### Test 1: Single Element Selection ✅
- **Vad**: Välja en enskild form
- **Steg**: Rita rektangel → Klicka på den
- **Resultat**: Rektangeln blev markerad med blå gräns
- **Status**: Fungerar som förväntat

#### Test 2: Box Selection ✅
- **Vad**: Välja flera element genom att dra en ruta
- **Steg**: Rita två rektanglar → Dra selection-ruta omkring båda
- **Resultat**: Båda markerades, selection-rutan var synlig under drag
- **Status**: Fungerar som förväntat

#### Test 3: Shift+Click Multi Select ✅
- **Vad**: Lägga till element till selection
- **Steg**: Välj första rektangel → Shift+klick på andra
- **Resultat**: Båda blev markerade
- **Status**: Fungerar som förväntat

#### Test 4: Undo/Redo ✅
- **Vad**: Ångra och gör om ändringar
- **Steg**: Rita rektangel → Ctrl+Z → Ctrl+Y
- **Resultat**: Rektangeln försvann, kom tillbaka
- **Status**: Fungerar som förväntat

#### Test 5: Färgändring ✅
- **Vad**: Ändra färg på form
- **Steg**: Rita rektangel → Välj den → Ändra färg
- **Resultat**: Rektangeln ändrade färg direkt
- **Status**: Fungerar som förväntat

#### Test 6: Radera element ✅
- **Vad**: Ta bort en form
- **Steg**: Rita rektangel → Välj den → Delete
- **Resultat**: Rektangeln försvann, kunde angras med Ctrl+Z
- **Status**: Fungerar som förväntat

#### Test 7: Drag/Flytta element ✅
- **Vad**: Flytta form genom att dra
- **Steg**: Rita rektangel → Välj → Dra till ny position
- **Resultat**: Följde musen, hamnade på rätt plats
- **Status**: Fungerar som förväntat

#### Test 8: Resize/Storleksändra ✅
- **Vad**: Ändra storlek på form
- **Steg**: Rita rektangel → Välj → Dra på handlens
- **Resultat**: Storleken ändrades enligt drag
- **Status**: Fungerar som förväntat

### Sammanfattning exploratory testing

- **Totalt tester**: 8
- **Alla fungerade**: ✅
- **Buggar hittade**: 0
- **UI-element som saknades**: 0
- **Övergripande intryck**: Appen fungerar väl och intuitivt

---

## 5. Slutsatser

### Vad lärde jag mig

1. **Test-setup är kritiskt**: Ett litet problem (localStorage-mock) blockerade 301 tester. Rätt konfiguration är essentiell.

2. **Coverage vs Quality**: Antalet tester (1079) är imponerande, men det finns gap (keyboard-testning, touch, cross-platform).

3. **Exploratory testing är värdefullt**: Manuell testning bekräftade att testerna täcker det som faktiskt är viktigt för användaren.

4. **Snapshot-tests kan vara problematiska**: 68 snapshot-mismatches visar att snapshot-baserade tester kräver underhåll.

5. **Open-source projekt är vältestat**: Excalidraw's 1079 tester visar ett allvarligt förhållningssätt till kodkvalitet.

### Reflektioner

**Styrkor i Excalidraw's testing:**
- Omfattande test-coverage
- Flera test-typer (unit, integration, regression)
- Automatiserad testning från dag ett
- Fokus på användarinteraktion, inte bara funktioner

**Möjligheter att förbättra:**
- Keyboard-navigation testning (för accessibility)
- Touch/mobile testning
- Performance-testning med många element
- Bättre error-reporting i tester
- Ångra/göra om-funktionalitet med selection

### Rekommendationer för framtida testing

1. Implementera keyboard-navigation tester (accessibility)
2. Lägg till touch-event simulering för mobile
3. Lägg till cross-platform modifier-key testning
4. Implementera visual regression testing (för UI-ändringar)
5. Lägg till performance benchmarks

---

## 6. Bilaga: Tekniska detaljer

### Miljö
- **Node.js**: v25.2.1
- **Package Manager**: yarn 1.22.22
- **Test Runner**: Vitest 3.0.6
- **Test Framework**: @testing-library/react
- **Canvas Mock**: vitest-canvas-mock

### Filer analyserade
- `setupTests.ts` - Test-miljö konfiguration
- `vitest.config.mts` - Vitest konfiguration
- `packages/excalidraw/tests/selection.test.tsx` - Selection-tester

### Kommando för att köra tester
```bash
# Alla tester
yarn test:app --run

# Enkel test-fil
yarn test:app packages/excalidraw/tests/selection.test.tsx --run

# Med coverage
yarn test:app --coverage

# Watch mode
yarn test:app --watch
```

---

## Slutord

Excalidraw är ett excellentexempel på ett välutvecklat open-source projekt med seriös approach till testning. Genom att analysera dess test-suite lärde jag mig värdefulla lärdomar om:

- Hur man strukturerar tester för användarinteraktion
- Vikten av rätt test-setup
- Gap mellan test-coverage och faktisk kval
- Värdet av både automatiserad och manuell testning

**Projekt-status**: Väl underhållet, aktivt utvecklat, högt kodkvalitet.