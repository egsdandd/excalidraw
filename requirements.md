---
title: Requirements Specification – Excalidraw
author: Web Programming Student
date: 2025-12-17
---

# Requirements Specification – Excalidraw

## Har Excalidraw en requirements specification?

**Kort svar:** Ja, på flera sätt – även om den inte är formellt dokumenterad.

---

## 1. Implicit i README och dokumentation

~~~ bash

cat README.md
~~~

**README** beskriver bland annat:

- Vad appen är (ett whiteboard-/diagramverktyg)
- Vad den kan göra (rita, dela, exportera)
- Hur man installerar och använder den

*(https://github.com/excalidraw/excalidraw)*

---

## 2. Implicit i själva applikationen

**UI** visar tydligt funktionalitet:

- Ritverktyg: linjer, rektanglar, cirklar, pilar  
- Grundläggande features: spara, exportera, dela, ångra, färgval m.m.  
- Användarbeteende visar vad som krävs: Vad kan användaren faktiskt göra?

*(https://excalidraw.com)*

---

## 3. I koden och testerna

- **Testfilerna** visar vad som förväntas fungera.  
- **Kodstrukturen** avslöjar krav och designbeslut.  
- **GitHub-issues och PRs** ger inblick i planerade och befintliga funktioner.

*(https://github.com/excalidraw/excalidraw)*

---

## Sammanfattande bedömning

**Ja – Excalidraw uppfyller kravet på requirements specification genom:**

✅ **Tolkbara requirements:** Appens syfte som ritverktyg är uppenbart.  
✅ **Självförklarande funktionalitet:** Genom att använda appen ser du vad den ska göra.  
✅ **Tydlig dokumentation:** README förklarar syfte och huvudfunktioner.  
✅ **Kod och tester som visar förväntat beteende:** Tests definierar vad som måste fungera.  

---

## Slutsats

Du behöver ingen separat, formell requirements specification för Excalidraw.  
Du kan förstå vad som ska fungera genom att:

1. Läsa **README** *(https://github.com/excalidraw/excalidraw)*
2. **Använda** applikationen *(https://excalidraw.com)*
3. Läsa **tester**, issues och PRs *(https://github.com/excalidraw/excalidraw)*
