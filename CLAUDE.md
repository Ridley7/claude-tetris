# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Proyecto

Tetris en JavaScript vanilla + HTML5 Canvas. Sin dependencias, sin `package.json`, sin build, sin tests, sin linter. UI y textos en español.

## Ejecutar

```bash
python3 -m http.server 8000   # luego abrir http://localhost:8000
# o simplemente: start index.html (Windows)
```

No hay suite de tests; verificar cambios jugando en el navegador.

## Arquitectura

Tres archivos: `index.html` (DOM: `#board` canvas, panel HUD, `#next-canvas`, `#overlay`), `style.css`, `game.js` (toda la lógica).

`game.js` es un script clásico (no módulo) con estado global mutable en variables `let` a nivel de archivo (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropAccum`, `dropInterval`, `animId`). `init()` resetea todo y también es el handler del botón reiniciar.

- **Modelo**: `board` es matriz `ROWS × COLS`; cada celda `0` o índice de color 1–7. Las piezas en `PIECES` son matrices cuadradas cuyas celdas ya contienen su índice de tipo, por lo que `merge()` copia el valor directamente y `COLORS[i]` da el color. `PIECES`/`COLORS` están indexados desde 1 (`[0]` es `null`) — mantener alineados.
- **Piezas**: `{ type, shape, x, y }`. Rotación = `rotateCW` + wall kicks `[0,-1,1,-2,2]` en `tryRotate`. Solo rotación horaria; no es SRS.
- **Colisión**: `collide(shape, ox, oy)` permite `ny < 0` (celdas sobre el tablero).
- **Loop**: `requestAnimationFrame` acumula `dt` en `dropAccum`; al superar `dropInterval` baja una fila o llama `lockPiece()` → `merge()` → `clearLines()` → `spawn()`. `spawn()` dispara `endGame()` si la pieza nueva colisiona.
- **Pausa/Game Over**: ambos cancelan `animId` y reutilizan el mismo `#overlay`. Nota: al reanudar, `togglePause` no vuelve a ocultar el overlay.
- **Puntuación/nivel**: `LINE_SCORES[cleared] * level`; hard drop +2/celda, soft drop +1/fila; nivel = `floor(lines/10)+1`; `dropInterval = max(100, 1000 - (level-1)*90)`.
- **Render**: `draw()` redibuja todo cada frame (grid → board → ghost con alpha 0.2 → pieza actual). `drawBlock` sirve para ambos canvas vía parámetro `context`/`size`.

## Restricciones

- Si cambian `COLS`, `ROWS` o `BLOCK`, ajustar `width`/`height` de `<canvas id="board">` en `index.html` (`COLS*BLOCK` × `ROWS*BLOCK`).
- `drawNext` asume un canvas de 4×4 celdas de 30px (`#next-canvas` 120×120).
