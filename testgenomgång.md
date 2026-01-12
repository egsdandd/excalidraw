# Testgenomgång - Excalidraw Projekt

**Datum:** 12 januari 2026  
**Totalt antal testfiler:** ~89 filer  
**Totalt antal test cases:** 500+

---

## 1. Testnivåer

### 1.1 Regression Testing (~5%)

**Dedikerade filer:**
- [packages/excalidraw/tests/regressionTests.test.tsx](packages/excalidraw/tests/regressionTests.test.tsx) (1201 rader)

**Beskrivning:**
- Testar tidigare buggar för att säkerställa att de inte återkommer
- Använder snapshots för att verifiera rendering och state
- Verifierar antal renderingar: `expect(renderStaticScene.mock.calls.length)`
- Checkpoint-baserad testing för att spåra exakt var fel uppstår
- Fokuserar på specifika buggar som tidigare upptäckts i produktionen

**Exempel på testfall:**
- Verifiering av element-state efter specifika operationer
- Rendering-konsistens vid olika interaktioner
- Edge cases som tidigare orsakat problem

---

### 1.2 Unit Testing (~70%)

Detta är den dominerande testtypen i projektet.

#### **A. Matematik/Geometri (packages/math/tests/)**
- `curve.test.ts` - Kurvintersektioner, närmaste punkt på kurva
- `ellipse.test.ts` - Punkt-i-ellips, linje-ellips-intersektioner
- `line.test.ts` - Linje-linje-intersektioner
- `point.test.ts` - Punktrotation och transformationer
- `segment.test.ts` - Linjesegment-intersektioner
- `range.test.ts` - Range overlap och intersektioner
- `vector.test.ts` - Vektoroperationer

**Karaktär:** Ren matematik, testar enskilda funktioner isolerat.

#### **B. Utilities (packages/utils/tests/)**
- `geometry.test.ts` - Geometriska hjälpfunktioner
- `withinBounds.test.ts` - Boundingbox-beräkningar
- `export.test.ts` - Export-funktioner (canvas, blob, SVG)
- `utils.unmocked.test.ts` - Utilities utan mocking

**Karaktär:** Hjälpfunktioner som används av övriga systemet.

#### **C. Common Utilities (packages/common/src/)**
- `utils.test.ts` - `isTransparent()`, `reduceToCommonValue()`, `mapFind()`

#### **D. Element-operationer (packages/element/tests/)**
- `align.test.tsx` - Elementjustering (top, bottom, left, right, center)
- `binding.test.tsx` - Arrow-bindings till shapes

**Testfall exempel:**
- Justera två objekt till toppen
- Justera grupper med nested elements
- Bindings mellan pilar och shapes

#### **E. Actions (packages/excalidraw/actions/)**
- `actionDeleteSelected.test.tsx` - Radering med frames och children
- `actionDuplicateSelection.test.tsx` - Duplicering av frames, grupper, bindings
- `actionElementLock.test.tsx` - Låsning/upplåsning av element
- `actionProperties.test.tsx` - Element-egenskaper

#### **F. Data-hantering**
- `clipboard.test.ts` - Clipboard parsing (JSON, HTML, bilder, spreadsheets)
- `charts.test.ts` - Chart-data parsing och validering

---

### 1.3 System Testing (~20%)

Testar hela subsystem och integration mellan komponenter.

#### **Huvudsakliga system-tester:**

**App & Core:**
- `App.test.tsx` - Applikationsstate och konfiguration
- `excalidraw.test.tsx` - Huvudkomponent med props och children
- `appState.test.tsx` - Application state management

**History System (5243 rader!):**
- `history.test.tsx` - Omfattande tester av undo/redo-systemet
  - Element-manipulationer och historik
  - Multi-element operationer
  - State-återställning
  - History stack management

**Collaboration:**
- `collab.test.tsx` - Realtidssamarbete
  - Ephemeral updates
  - Force-deleted elements med undo/redo
  - Synkronisering mellan klienter

**Export Pipeline:**
- `export.test.tsx` - Hela export-flödet
  - Canvas export
  - Blob generation
  - SVG export
  - Format-konvertering

**Feature Integration:**
- `selection.test.tsx` (559 rader) - Selektionssystem
- `tool.test.tsx` - Verktygshantering
- `shortcuts.test.tsx` - Keyboard shortcuts
- `dragCreate.test.tsx` - Drag-and-drop skapande
- `rotate.test.tsx` - Rotationsfunktionalitet
- `scroll.test.tsx` - Scrolling och viewport
- `search.test.tsx` - Sökfunktionalitet
- `contextmenu.test.tsx` - Context menu
- `viewMode.test.tsx` - View modes
- `elementLocking.test.tsx` - Element locking system
- `lasso.test.tsx` - Lasso selection
- `library.test.tsx` - Element library
- `image.test.tsx` - Bildhantering
- `multiPointCreate.test.tsx` - Multi-point elements
- `move.test.tsx` - Element movement
- `flip.test.tsx` - Element flipping
- `fitToContent.test.tsx` - Auto-resize
- `actionStyles.test.tsx` - Style actions
- `MermaidToExcalidraw.test.tsx` - Mermaid diagram conversion

---

### 1.4 Acceptance Testing (~5%)

Testar användarupplevelse och UI från användarens perspektiv.

**UI Komponenter:**
- `MobileMenu.test.tsx` - Mobil meny-funktionalitet
  - Form factor detection (phone)
  - Welcome screen interaktion
  - UI state efter användarinteraktion
  
- `LanguageList.test.tsx` - Språkväxling
  - UI re-rendering vid språkbyte
  
- `DropdownMenu.test.tsx` - Dropdown-komponenten
  - Meny-öppning/stängning
  - Item selection

**Karaktär:**
- Testar från slutanvändarens perspektiv
- Verifierar UI-rendering och användarflöden
- Snapshot testing för UI-konsistens

---

## 2. Testmetodiker

### 2.1 Random Testing (~0%)

**Förekomst:** Ingen explicit random testing.

**Observation:**
- Projektet använder `reseed(7)` för **deterministisk** slumpmässighet
- Detta ger reproducerbara tester istället för random testing
- Fokus ligger på förutsägbara test cases

**Varför ingen random testing?**
- Grafiska element kräver exakta positioner
- UI-interaktioner måste vara reproducerbara
- Debugging skulle bli svårare med random inputs

---

### 2.2 Black-box Testing (~35%)

Testar funktionalitet utan kunskap om intern implementation.

#### **UI/Component Tests:**
- `MobileMenu.test.tsx` - Testar UI-beteende
- `LanguageList.test.tsx` - Testar språkbyte-effekt
- `DropdownMenu.test.tsx` - Testar meny-interaktioner

#### **Feature Tests:**
- `selection.test.tsx` - Testar selektionsbeteende utan att känna till intern algoritm
- `tool.test.tsx` - Testar verktyg från användarens perspektiv
- `shortcuts.test.tsx` - Testar keyboard shortcuts (input → output)

#### **Export Tests:**
- `export.test.ts` - Testar export-format utan att känna till rendering-detaljer
  ```typescript
  it("should change image/jpg to image/jpeg", async () => {
    // Input: image/jpg
    // Expected output: image/jpeg
  })
  ```

#### **Data Tests:**
- `clipboard.test.ts` - Testar clipboard parsing
  ```typescript
  it("should parse JSON as plaintext if not excalidraw data")
  it("should parse <image> src urls out of text/html")
  ```

**Kännetecken:**
- Testar input → output
- Verifierar specifikationer
- Ingen insyn i intern logik
- Fokus på användarbeteende

---

### 2.3 White-box Testing (~60%)

Testar intern logik och kodvägar med full kunskap om implementationen.

#### **Matematik Tests (fullständig täckning):**
```typescript
// curve.test.ts
it("point is found when control points are the same", () => {
  const c = curve(
    pointFrom(100, 0),
    pointFrom(100, 100),
    pointFrom(100, 100),
    pointFrom(0, 100),
  );
  const l = lineSegment(pointFrom(0, 0), pointFrom(200, 200));
  expect(curveIntersectLineSegment(c, l)).toCloselyEqualPoints([[87.5, 87.5]]);
});
```

**Karaktär:**
- Testar exakta matematiska beräkningar
- Verifierar edge cases i algoritmer
- Täcker specifika kodvägar

#### **State Management:**
- `history.test.tsx` - Testar intern state-hantering
  - Undo/redo stack manipulation
  - State transitions
  - History branching

#### **Internal Logic:**
- `binding.test.tsx` - Testar binding-algoritmer
- `align.test.tsx` - Testar alignment-beräkningar
- `clipboard.test.ts` - Testar parsing-logik för olika format

#### **Mocking av interna moduler:**
```typescript
const renderStaticScene = vi.spyOn(StaticScene, "renderStaticScene");
const renderInteractiveScene = vi.spyOn(InteractiveCanvas, "renderInteractiveScene");
```

**Kännetecken:**
- Testar specifika funktioner och metoder
- Verifierar intern state
- Använder mocking för isolation
- Testar edge cases och error paths
- Code coverage focus

---

### 2.4 Glass-box Testing (~5%)

Glass-box = White-box med fokus på strukturell täckning och intern state-inspektion.

#### **Snapshot Testing:**
```typescript
const checkpoint = (name: string) => {
  expect(renderStaticScene.mock.calls.length).toMatchSnapshot(
    `[${name}] number of renders`,
  );
  expect(h.state).toMatchSnapshot(`[${name}] appState`);
  expect(h.elements.length).toMatchSnapshot(`[${name}] number of elements`);
};
```

#### **Render Count Verification:**
```typescript
expect(renderStaticScene.mock.calls.length).toBe(expectedCount);
```

#### **Detailed State Inspection:**
```typescript
expect(h.state.zenModeEnabled).toBe(true);
expect(h.elements[0].x).toBe(100);
expect(h.app.editorInterface.formFactor).toBe("phone");
```

**Kännetecken:**
- Verifierar exakt antal renderingar
- Inspekterar djup state
- Snapshot-baserad verifiering
- Strukturell täckning av kod

---

## 3. Statistisk Sammanfattning

### Testnivåer

| Testnivå | Andel | Antal filer | Beskrivning |
|----------|-------|-------------|-------------|
| **Unit Testing** | ~70% | 60-65 | Enskilda funktioner och metoder |
| **System Testing** | ~20% | 15-20 | Integration och subsystem |
| **Acceptance Testing** | ~5% | ~5 | UI och användarupplevelse |
| **Regression Testing** | ~5% | 1-2 | Tidigare buggar |

### Testmetodiker

| Metodik | Andel | Antal filer | Karaktär |
|---------|-------|-------------|----------|
| **White-box** | ~60% | ~50 | Intern logik, matematik, state |
| **Black-box** | ~35% | ~30 | UI, features, export |
| **Glass-box** | ~5% | ~5 | Snapshots, render counts |
| **Random Testing** | ~0% | 0 | Används ej |

---

## 4. Testverktyg och Framework

**Test Runner:**
- Vitest (modern, snabb Vite-baserad test runner)

**Testing Libraries:**
- `@testing-library/react` - React component testing
- `@testing-library/user-event` - Användarinteraktioner
- `vitest` - Mocking och assertions

**Custom Test Utilities:**
- `test-utils.ts` - Projektspecifika hjälpfunktioner
- `API.ts` (helpers) - Element creation helpers
- `ui.ts` (helpers) - UI interaction helpers (Pointer, Keyboard, UI)

---

## 5. Testmönster och Best Practices

### 5.1 Test Organization
```typescript
describe("Feature Name", () => {
  describe("Sub-feature", () => {
    it("should do specific thing", () => {
      // Test implementation
    });
  });
});
```

### 5.2 Setup/Teardown
```typescript
beforeEach(() => {
  localStorage.clear();
  renderInteractiveScene.mockClear();
  reseed(7); // Deterministisk slumpmässighet
});

afterAll(() => {
  restoreOriginalGetBoundingClientRect();
});
```

### 5.3 Mocking
```typescript
const renderStaticScene = vi.spyOn(StaticScene, "renderStaticScene");
mockBoundingClientRect(dimensions);
```

### 5.4 Assertions
```typescript
expect(result).toCloselyEqualPoints([[87.5, 87.5]]);
expect(h.state).toMatchSnapshot();
expect(element).toMatchInlineSnapshot();
```

---

## 6. Testfiler per Kategori

### Matematik (packages/math/tests/)
- `curve.test.ts`
- `ellipse.test.ts`
- `line.test.ts`
- `point.test.ts`
- `range.test.ts`
- `segment.test.ts`
- `vector.test.ts`

### Element (packages/element/tests/)
- `align.test.tsx`
- `binding.test.tsx`

### Actions (packages/excalidraw/actions/)
- `actionDeleteSelected.test.tsx`
- `actionDuplicateSelection.test.tsx`
- `actionElementLock.test.tsx`
- `actionProperties.test.tsx`

### Core Features (packages/excalidraw/tests/)
- `App.test.tsx`
- `appState.test.tsx`
- `charts.test.ts`
- `clipboard.test.tsx`
- `contextmenu.test.tsx`
- `dragCreate.test.tsx`
- `elementLocking.test.tsx`
- `excalidraw.test.tsx`
- `export.test.tsx`
- `fitToContent.test.tsx`
- `flip.test.tsx`
- `history.test.tsx` (5243 rader!)
- `image.test.tsx`
- `lasso.test.tsx`
- `library.test.tsx`
- `MermaidToExcalidraw.test.tsx`
- `move.test.tsx`
- `multiPointCreate.test.tsx`
- `regressionTests.test.tsx`
- `rotate.test.tsx`
- `scroll.test.tsx`
- `search.test.tsx`
- `selection.test.tsx`
- `shortcuts.test.tsx`
- `tool.test.tsx`
- `viewMode.test.tsx`

### UI Components (excalidraw-app/tests/)
- `collab.test.tsx`
- `LanguageList.test.tsx`
- `MobileMenu.test.tsx`

### Utilities
- `packages/utils/tests/export.test.ts`
- `packages/utils/tests/geometry.test.ts`
- `packages/utils/tests/withinBounds.test.ts`
- `packages/utils/tests/utils.unmocked.test.ts`
- `packages/common/src/utils.test.ts`

---

## 7. Observationer och Insikter

### Styrkor:
1. **Omfattande unit testing** - Särskilt matematik och geometri har excellent täckning
2. **Deterministiska tester** - `reseed(7)` gör alla tester reproducerbara
3. **History testing** - 5243 rader dedikerade till undo/redo visar hög kvalitetsfokus
4. **Regression focus** - Dedikerad fil för att förhindra återkommande buggar
5. **Custom helpers** - Välutvecklade test utilities (API, Pointer, Keyboard, UI)

### Förbättringsområden:
1. **Ingen E2E testing** - Saknas end-to-end tester
2. **Begränsad acceptance testing** - Endast 5% av testerna
3. **Ingen random/fuzz testing** - Kunde hitta edge cases
4. **Integration testing** - Kunde vara mer omfattande (20% idag)

### Testkultur:
- **Matematiskt rigorös** - Exakta assertions för geometri
- **UI-fokuserad** - Snapshot testing för UI-konsistens
- **Developer-friendly** - Tydliga beskrivningar och hjälpfunktioner
- **Continuous** - Tester körs vid varje förändring

---

## 8. Slutsats

Excalidraw-projektet har en **mycket robust testsvit** med fokus på:
- ✅ Unit testing (70%)
- ✅ White-box testing (60%)
- ✅ Deterministiska, reproducerbara tester
- ✅ Matematisk precision
- ✅ Regression prevention

Projektet skulle kunna förbättras med:
- ❌ E2E testing
- ❌ Mer acceptance testing
- ❌ Performance testing
- ❌ Random/fuzz testing för edge case discovery

**Totalt: 89 testfiler med 500+ individuella test cases**

Testsviten ger hög konfidans i kodkvaliteten och förhindrar regressioner effektivt.
