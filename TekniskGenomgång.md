# Excalidraw - Teknisk Arkitektur & Presentation

**För**: Studiekompisar  
**Fokus**: Hur är Excalidraw byggt? Vilken teknik används?  
**Nivå**: Introduktion till avancerad

---
# 1 Teknisk genomgång

Följande täcks in i detta dokument:

- Övergripande arkitektur - Visuell diagram över lagerna
- Tech Stack - React, Vite, TypeScript osv
- Rendering Engine - Varför Canvas och hur det fungerar
- Element System - Datastruktur för former
- Interaction Handling - Event loop och drag/drop
- Undo/Redo System - Hur history fungerar
- Storage - localStorage, IndexedDB, Cloud
- Testing Infrastructure - Vitest, @testing-library
- Deployment - Web app, npm package, Docker
- Performance Optimizations - Caching, debouncing, virtual scrolling
- Architecture Decisions - Varför vissa val gjordes
- Development Workflow - Lokalt development setup
- Scaling Considerations - Vad händer med stora diagram?

## 1. Övergripande Arkitektur

```txt
┌─────────────────────────────────────────────────────────┐
│                    WEBBROWSER                           │
├─────────────────────────────────────────────────────────┤
│  React Frontend (UI Components)                         │
│  ├─ Toolbar (verktyg)                                  │
│  ├─ Canvas (rityta)                                    │
│  ├─ Panels (egenskaper, bibliotek)                     │
│  └─ Dialogs (modals, settings)                         │
├─────────────────────────────────────────────────────────┤
│  State Management (Redux/useReducer)                    │
│  ├─ Elements (former)                                  │
│  ├─ AppState (app-inställningar)                       │
│  ├─ History (undo/redo)                               │
│  └─ Scenes (låger)                                     │
├─────────────────────────────────────────────────────────┤
│  Rendering Engine                                       │
│  ├─ Interactive Scene (canvas rendering)               │
│  ├─ Static Scene (bakgrund)                            │
│  ├─ WebGL (GPU acceleration)                           │
│  └─ Canvas 2D API                                      │
├─────────────────────────────────────────────────────────┤
│  Core Logic                                             │
│  ├─ Element manipulation                               │
│  ├─ Transform calculations                             │
│  ├─ Collision detection                                │
│  └─ Export/Import                                      │
├─────────────────────────────────────────────────────────┤
│  Storage & Sync                                         │
│  ├─ localStorage (lokal lagring)                        │
│  ├─ IndexedDB (större data)                            │
│  ├─ CloudStorage (molnet)                              │
│  └─ Collaboration (realtid sync)                       │
└─────────────────────────────────────────────────────────┘
```

---

## 2. Tech Stack

### Frontend Framework

**React** (JavaScript UI-bibliotek)
- **Version**: React 19.0.10
- **Varför React?**
  - Komponentbaserad arkitektur (återanvändbar kod)
  - Virtual DOM (effektiv rendering)
  - Enkelt att hantera komplexa UI-stater
  - Stor community och många bibliotek

**Exempel React-komponenter i Excalidraw:**
```javascript
<Excalidraw />           // Huvudapp
<Toolbar />              // Verktygsrad
<Canvas />               // Rityta
<ExportDialog />         // Export-dialog
<ColorPicker />          // Färgväljare
```

### Build Tools & Development

**Vite** (Modern bundler)
- **Version**: 5.0.12
- **Varför Vite?**
  - Mycket snabbare än Webpack (instant server start)
  - Hot Module Replacement (HMR) - ändringar utan reload
  - Optimal production builds
  - Native ES modules support

**TypeScript** (Typed JavaScript)
- **Version**: 5.9.3
- **Varför TypeScript?**
  - Typ-säkerhet (fångar fel före körtid)
  - Bättre IDE-stöd (autocomplete, refactoring)
  - Self-documenting kod
  - Lättare att underhålla stort projekt

### State Management

**Redux-liknande pattern** (useReducer hook)
- **Varför inte Redux direkt?**
  - Lättare och mindre boilerplate
  - React hooks är moderna standard
  - Tillräckligt för denna app's komplexitet

**State struktur:**
```typescript
interface AppState {
  elements: Element[];           // Alla former på canvas
  selectedElementIds: Set<string>; // Vilka är valda?
  appState: {
    zoom: number;                // Zoom-nivå
    scrollX: number;             // Horisontell scroll
    scrollY: number;             // Vertikal scroll
    viewBackgroundColor: string;  // Bakgrundsfärg
    // ... 50+ properties
  };
  history: HistoryEntry[];       // För undo/redo
}
```

---

## 3. Rendering Engine

### Canvas vs SVG vs WebGL

Excalidraw använder **HTML5 Canvas** (inte SVG eller WebGL enbart)

**Varför Canvas?**
- ✅ Högre performance än SVG för många element
- ✅ Perfekt för ritprogram (pixelbaserad)
- ✅ GPU acceleration möjlig
- ✅ Enkelt att implementera "handritad" känsla
- ❌ SVG vore för långsamt med många element
- ❌ WebGL vore för komplext för denna use-case

### Två Canvas-lager

Excalidraw använder **två canvas-element samtidigt:**

```
┌──────────────────────────────┐
│  Interactive Canvas (överst) │ ← Musklick, drag, selection
│  (renderInteractiveScene)    │
└──────────────────────────────┘
         ↓
┌──────────────────────────────┐
│  Static Canvas (under)       │ ← Det användaren rita
│  (renderStaticScene)         │
└──────────────────────────────┘
```

**Varför två canvas?**
- Interactive: Uppdateras ofta (mousemove, hover)
- Static: Uppdateras sällan (bara när något ändras)
- **Resultat**: Bättre performance

### Rendering Pipeline

```javascript
// 1. Hämta canvas context
const ctx = canvas.getContext('2d');

// 2. Rensa (clear)
ctx.clearRect(0, 0, width, height);

// 3. Applicera transformationer (zoom, scroll)
ctx.transform(...);

// 4. Rita alla element
elements.forEach(element => {
  ctx.fillStyle = element.backgroundColor;
  ctx.fillRect(element.x, element.y, element.width, element.height);
});

// 5. Rita selection-ruta om något är valt
if (selectedElements.length > 0) {
  ctx.strokeStyle = '#0066cc';
  ctx.strokeRect(...);
}
```

---

## 4. Element System

### Element Datastruktur

```typescript
interface Element {
  // Identitet
  id: string;
  type: 'rectangle' | 'diamond' | 'ellipse' | 'arrow' | 'text' | 'image' | 'frame';
  
  // Position & Storlek
  x: number;
  y: number;
  width: number;
  height: number;
  
  // Stil
  backgroundColor: string;
  strokeColor: string;
  fillStyle: 'hachure' | 'cross-hatch' | 'solid';
  strokeWidth: number;
  
  // Rotation & Transform
  angle: number;
  
  // Gruppering
  groupIds: string[];
  
  // Binding (för pilar)
  startBinding: {
    elementId: string;
    focus: number;
    gap: number;
  } | null;
  
  // Text (för text-element)
  text: string;
  fontSize: number;
  fontFamily: string;
  
  // Locking & Visibility
  locked: boolean;
  versionNonce: number; // För collaboration
}
```

### Element Operations

**CRUD (Create, Read, Update, Delete):**

```javascript
// Create
const rect = {
  id: generateIdForElement(),
  type: 'rectangle',
  x: 100, y: 100,
  width: 200, height: 100,
  // ...
};
elements.push(rect);

// Read
const rect = elements.find(el => el.id === rectId);

// Update
rect.backgroundColor = '#ff0000';
rect.width = 300;

// Delete
elements = elements.filter(el => el.id !== rectId);
```

---

## 5. Interaction Handling

### Event Loop

```
User Action (click, drag, key)
            ↓
Event Listener (onPointerDown, onPointerMove, onPointerUp)
            ↓
Calculate Action (what changed?)
            ↓
Update State (elements, appState)
            ↓
Re-render Canvas
            ↓
Visual Feedback to User
```

### Exempel: Drag an Element

```javascript
// 1. Pointer Down
onPointerDown = (event) => {
  const element = getElementAtPosition(event.x, event.y);
  if (element) {
    state.draggedElement = element;
    state.dragOffset = {
      x: event.x - element.x,
      y: event.y - element.y
    };
  }
}

// 2. Pointer Move
onPointerMove = (event) => {
  if (state.draggedElement) {
    state.draggedElement.x = event.x - state.dragOffset.x;
    state.draggedElement.y = event.y - state.dragOffset.y;
    render(); // Uppdatera canvas
  }
}

// 3. Pointer Up
onPointerUp = (event) => {
  if (state.draggedElement) {
    // Spara i history för undo
    addToHistory(state);
    state.draggedElement = null;
  }
}
```

### Collision Detection

```javascript
// Kollidera två rektanglar?
function doRectanglesCollide(rect1, rect2) {
  return !(
    rect1.x + rect1.width < rect2.x ||
    rect2.x + rect2.width < rect1.x ||
    rect1.y + rect1.height < rect2.y ||
    rect2.y + rect2.height < rect1.y
  );
}

// Hit testing (klickade användaren på element?)
function getElementAtPosition(x, y) {
  // Iterera bakifrån (översta element först)
  for (let i = elements.length - 1; i >= 0; i--) {
    const element = elements[i];
    if (doRectanglesCollide(element, { x, y, width: 1, height: 1 })) {
      return element;
    }
  }
  return null;
}
```

---

## 6. Undo/Redo System

### History Management

```typescript
interface HistoryEntry {
  elements: Element[];
  appState: AppState;
  timestamp: number;
}

class History {
  entries: HistoryEntry[] = [];
  currentIndex: number = -1;
  
  addEntry(elements, appState) {
    // Ta bort framtida entries om användaren gör något nytt
    this.entries = this.entries.slice(0, this.currentIndex + 1);
    this.entries.push({ elements, appState, timestamp: Date.now() });
    this.currentIndex++;
  }
  
  undo() {
    if (this.currentIndex > 0) {
      this.currentIndex--;
      return this.entries[this.currentIndex];
    }
  }
  
  redo() {
    if (this.currentIndex < this.entries.length - 1) {
      this.currentIndex++;
      return this.entries[this.currentIndex];
    }
  }
}
```

**Varför denna design?**
- ✅ Enkelt att förstå
- ✅ Effektivt för små ändringar
- ✅ Kan ta mycket minne för stora ändringar
- ❌ Inte optimal för mycket stora diagram

---

## 7. Storage & Persistence

### localStorage (Webbrowser)

```javascript
// Spara
localStorage.setItem('excalidraw-app', JSON.stringify({
  elements: [...],
  appState: {...}
}));

// Ladda
const saved = JSON.parse(localStorage.getItem('excalidraw-app'));
```

**Begränsningar:**
- ~5-10MB per origin (webbsida)
- Synkron (blockerar UI vid stora filer)
- Plain text (ingen kompression)

### IndexedDB (För större data)

```javascript
// Större filhantering
const db = await openDB('excalidraw-db');
await db.put('drawings', {
  id: drawingId,
  data: hugeDrawingData, // Kan vara MB
  timestamp: Date.now()
});
```

**Fördelar:**
- ~50MB+ per origin
- Asynkron (blockerar inte UI)
- Kan lagra blob/binary data

### Cloud Storage (Molnet)

```javascript
// Spara till server
await fetch('/api/save', {
  method: 'POST',
  body: JSON.stringify({ elements, appState })
});

// Ladda från server
const data = await fetch('/api/load?id=123').json();
```

**Fördelar:**
- ✅ Obegränsad lagring
- ✅ Synkronisering mellan enheter
- ✅ Collaboration (flera användare)
- ❌ Kräver backend-server
- ❌ Nätverksberoende

---

## 8. Testing Infrastructure

### Test Stack

**Vitest** (Test Runner)
- Snabb test-körtid
- ESM support
- Jest-kompatibel

**@testing-library/react** (UI Testing)
- Testar komponenter som användare ser dem
- Simulerar klick, drag, keyboard
- Fokus på accessibility

**vitest-canvas-mock** (Canvas Mocking)
- Simulerar Canvas API i test-miljö
- Utan detta kan inte tester rita på canvas

### Test Organization

```
packages/excalidraw/tests/
├── selection.test.tsx         # Element selection
├── rotation.test.tsx          # Rotation
├── flip.test.tsx              # Flip/mirror
├── export.test.tsx            # Export functionality
├── history.test.tsx           # Undo/Redo
├── dragCreate.test.tsx        # Drag to create
├── lasso.test.tsx             # Lasso selection
├── library.test.tsx           # Element library
├── image.test.tsx             # Image handling
├── clipboard.test.tsx         # Copy/Paste
├── elementLocking.test.tsx     # Lock functionality
└── regressionTests.test.tsx    # Common bugs
```

### Test Pyramid

```
         /\
        /  \        E2E Tests (Selenium, Playwright)
       /____\       Few, slow, realistic
      /      \
     /        \     Integration Tests (Multi-component)
    /          \    Many, faster, realistic
   /____________\
  /              \   Unit Tests (Single function)
 /                \ Lots, fast, isolated
/__________________\
```

Excalidraw använder **Integration + Unit** (mest viktigt för UI-kod)

---

## 9. Deployment & Distribution

### Frontend Distribution

**Option 1: Standalone Webapp**
```bash
npm run build:app
# Producerar: excalidraw-app/build/
# Deploy till: GitHub Pages, Netlify, Vercel, etc.
```

**Option 2: NPM Package**
```bash
npm install @excalidraw/excalidraw
# Andra appar kan importera och använda det
import { Excalidraw } from '@excalidraw/excalidraw';
```

**Option 3: Docker Container**
```bash
docker build -t excalidraw .
docker run -p 3000:3000 excalidraw
```

### Monorepo Structure

```
excalidraw/ (monorepo root)
├── excalidraw-app/        # Web app
├── packages/
│   ├── common/            # Shared utilities
│   ├── element/           # Element types
│   ├── excalidraw/        # Core library
│   ├── math/              # Math utilities
│   └── utils/             # Helper functions
└── examples/              # Example implementations
```

**Varför monorepo?**
- ✅ Shared code mellan projekt
- ✅ Enkel versionering
- ✅ Coordinerad development
- ❌ Mer komplext setup

---

## 10. Performance Optimizations

### Rendering Optimization

```javascript
// Memoization - cache funktions-resultat
const memoizedGetBounds = useMemo(() => {
  return calculateElementBounds(elements);
}, [elements]); // Uppdatera bara om elements ändras

// Selective rendering - rita bara synliga element
const visibleElements = elements.filter(el => 
  isInViewport(el, viewport)
);
visibleElements.forEach(el => renderElement(el));
```

### Event Debouncing

```javascript
// Inte köra onPointerMove för VARJE musrörelse
const debouncedOnMove = debounce(onPointerMove, 16); // 60fps
canvas.addEventListener('pointermove', debouncedOnMove);
```

### Virtual Scrolling (för stora listor)

```javascript
// Om elementen lista är stor, rendera bara synliga
const visibleElements = elements.slice(startIndex, endIndex);
```

---

## 11. Key Technologies Summary

| Teknologi | Version | Syfte |
|-----------|---------|-------|
| **React** | 19.0.10 | UI Framework |
| **TypeScript** | 5.9.3 | Type Safety |
| **Vite** | 5.0.12 | Build Tool |
| **Canvas 2D** | - | Rendering |
| **IndexedDB** | - | Local Storage |
| **Vitest** | 3.0.6 | Testing |
| **@testing-library** | - | UI Testing |

---

## 12. Architecture Decisions & Tradeoffs

### Canvas vs SVG
- **Vald**: Canvas
- **Anledning**: Performance med många element
- **Tradeoff**: Svårare att implementera vissa features

### Redux vs useReducer
- **Vald**: useReducer (Redux-liknande)
- **Anledning**: Mindre boilerplate, modern React
- **Tradeoff**: Inte optimerad för mycket stora states

### localStorage vs IndexedDB
- **Vald**: Båda
- **Anledning**: localStorage för små, IndexedDB för stora
- **Tradeoff**: Mer komplext code

### Monorepo vs Separate Repos
- **Vald**: Monorepo
- **Anledning**: Shared code, easy versioning
- **Tradeoff**: Mer komplext setup, større install

---

## 13. Development Workflow

### Local Development

```bash
# 1. Installera dependencies
yarn install

# 2. Starta dev-server
yarn start

# 3. Gör ändringar (hot reload automatiskt)

# 4. Köra tester
yarn test:app

# 5. Build för production
yarn build
```

### Code Quality Tools

```bash
yarn test:code        # ESLint - kod stil och fel
yarn test:other       # Prettier - kodformatering
yarn test:typecheck   # TypeScript type checking
yarn test:all         # Alla tester
```

---

## 14. Scaling Considerations

### Vad händer när diagram blir mycket stort?

**Problem**: 10,000+ element → långsamt

**Lösningar Excalidraw använder:**
1. **Spatial Indexing** - Organisera element i grid
2. **Culling** - Rendera bara synliga element
3. **Throttling** - Minska render-frekvens vid drag
4. **Worker Threads** - Offload beräkningar

**Future improvements:**
- WebGL rendering för 100,000+ element
- Lazy loading av element
- Distributed collaboration

---

## Sammanfattning

**Excalidraw är byggt på:**

✅ Moderna web-teknologier (React, TypeScript, Vite)  
✅ Canvas för högt performance rendering  
✅ Redux-mönster för state management  
✅ Omfattande testning (Vitest, @testing-library)  
✅ Multiplafform distribution (web app, npm package, docker)  
✅ Smart caching och optimisering  

**Resultat**: En responsiv, snabb, och pålitlig ritapp som kan hantera komplexa diagram.

---

## För vidare läsning

- **React docs**: https://react.dev
- **Vite docs**: https://vitejs.dev
- **Canvas API**: https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API
- **Excalidraw repo**: https://github.com/excalidraw/excalidraw
- **Excalidraw docs**: https://docs.excalidraw.com