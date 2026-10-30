# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Proyecto

Tetris en JavaScript vanilla con HTML5 Canvas. No hay `package.json`, build, linter ni tests: son solo `index.html`, `style.css` y `game.js`. La documentación (README) y los textos de la UI están en español.

## Ejecutar

Abrir `index.html` directamente (`start index.html` en Windows) o servir la carpeta con un servidor estático (`python -m http.server 8000`, `npx serve .`) y abrir `http://localhost:8000`. No existe comando de tests; se verifica jugando en el navegador.

## Arquitectura

Toda la lógica vive en `game.js` (script clásico, `'use strict'`, estado global a nivel de módulo, sin `import`). Puntos que requieren leer varias partes para entenderlos:

- **Acoplamiento con el HTML**: `game.js` obtiene elementos por id al cargar (`board`, `next-canvas`, `score`, `lines`, `level`, `overlay`, `overlay-title`, `overlay-score`, `restart-btn`). Renombrar un id en `index.html` rompe el juego. Las dimensiones del canvas `#board` (300×600) están fijadas en el HTML y deben coincidir con `COLS × BLOCK` y `ROWS × BLOCK`; el canvas de la siguiente pieza es 120×120 (cuadrícula 4×4 de bloques de 30px).
- **Modelo**: `board` es una matriz `ROWS×COLS` con `0` o un índice de pieza 1–7. Ese mismo índice se usa en `PIECES` (formas) y `COLORS` (paleta, índice 0 = `null`); añadir o reordenar piezas exige mantener ambos arrays alineados y actualizar `randomPiece` (que asume 7 tipos).
- **Ciclo de vida de una pieza**: `spawn()` promueve `next` a `current` y detecta game over si colisiona al aparecer; `lockPiece()` = `merge()` → `clearLines()` → `spawn()`. Tanto el bucle (`loop`), el soft drop como el hard drop terminan en `lockPiece()`.
- **Bucle**: `loop` con `requestAnimationFrame` acumula `dropAccum` contra `dropInterval`; `clearLines` recalcula nivel y `dropInterval` (`max(100, 1000 - (level-1)*90)`). Pausa y game over cancelan el frame con `cancelAnimationFrame(animId)`; reanudar y `init()` (también el botón Reiniciar) lo reprograman.
- **Rotación**: `rotateCW` genera la matriz rotada y `tryRotate` prueba kicks horizontales `[0, -1, 1, -2, 2]` (no es SRS completo).
- **Skins**: sección `// ---- Skins ----` en `game.js`. `SKINS = { retro, neon, pastel, pixel }`, cada una con `palette` (7 colores, índice 0 = `null`, alineada con `PIECES`) y `draw(...)`; `drawBlock` delega en `SKINS[currentSkin]`. Fondo y cuadrícula por CSS vía `[data-skin]` en `<html>` (Neon fuerza negro); `applySkin` repinta, guarda en `localStorage['skin']` y el `<select id="skin-select">` está en `.top-controls` junto a `#theme-toggle`. El script inline del `<head>` aplica la skin guardada sin parpadeo.
- **Puntuación**: soft drop +1 por fila, hard drop +2 por celda, líneas según `LINE_SCORES` × nivel.

## CI / Triage de issues

`.github/workflows/claude-issue-triage.yml` analiza con Claude cada issue nuevo y cada edición de su título o cuerpo: asigna labels (tipo + `area:*` + `priority:*`, y `needs-info` si falta información) y publica/actualiza un único comentario de diagnóstico (marcador `<!-- claude-issue-triage -->`) con archivos implicados y propuesta de solución. Usa el secret `CLAUDE_CODE_OAUTH_TOKEN`, igual que `claude.yml` y `claude-code-review.yml`.
