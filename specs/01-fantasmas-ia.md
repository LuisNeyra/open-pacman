# SPEC 01 — Cuatro fantasmas con IA propia

> **Estado:** Approved
> **Depende de:** ninguno
> **Fecha:** 2026-08-11
> **Objetivo:** Añadir 4 fantasmas, cada uno con un comportamiento distinto, de los cuales uno persigue agresivamente a Pac-Man.

## Scope

**In:**

- Pasar de 2 a 4 fantasmas en `GHOST_STARTS` (`maze.js`), los 4 arrancando dentro de la pen.
- 4 comportamientos (`kind`) inspirados en el clásico: `hunter`, `ambusher`, `flanker`, `erratic`.
- Color por `kind` en `render.js` para respetar la identidad clásica (rojo perseguidor, rosa emboscador, cian flanqueador, naranja errático).

**Out of scope (specs futuros):**

- Power pellets / modo vulnerable (fantasmas azules comestibles).
- Fases scatter/chase cronometradas.
- Velocidades distintas por fantasma.
- Salidas escalonadas de la pen con temporizador.

## Data model

```js
// maze.js — GHOST_STARTS pasa a 4 entradas
const GHOST_STARTS = [
  { x: 13, y: 14, kind: 'hunter' },   // rojo — perseguidor agresivo
  { x: 14, y: 14, kind: 'ambusher' }, // rosa — emboscador
  { x: 12, y: 13, kind: 'flanker' },  // cian — flanqueador
  { x: 15, y: 13, kind: 'erratic' },  // naranja — errático
];
```

```js
// game.js — constantes nuevas
const AMBUSH_AHEAD = 4;            // celdas delante de Pac-Man (rosa)
const FLANK_AHEAD = 2;             // base del objetivo del cian
const FLANK_REF_KIND = 'hunter';   // fantasma de referencia del flanqueo
const ERRATIC_CHASE_RANGE = 8;     // distancia Manhattan (naranja)
```

```js
// render.js — color por kind, no por índice
const GHOST_COLOR_BY_KIND = {
  hunter:   '#ff0000',
  ambusher: '#ffb8ff',
  flanker:  '#00ffff',
  erratic:  '#ffb852',
};
```

Lógica de objetivo por `kind` (greedy de Manhattan existente en `decideGhost`, generalizado a un objetivo por fantasma):

- `hunter`: objetivo = posición actual de Pac-Man.
- `ambusher`: objetivo = Pac-Man + `AMBUSH_AHEAD` × dirección de Pac-Man.
- `flanker`: punto A = Pac-Man + `FLANK_AHEAD` × dir de Pac-Man; objetivo = A + (A − posición del `hunter`).
- `erratic`: si dist(Pac-Man, fantasma) > `ERRATIC_CHASE_RANGE` → objetivo = Pac-Man; si no → sin objetivo, elección aleatoria (reemplaza al `kind: 'random'` actual).

## Implementation plan

1. `maze.js`: extender `GHOST_STARTS` a las 4 entradas con los nuevos kinds. Prueba: recargar, se ven 4 fantasmas en la pen.
2. `game.js`: refactorizar `decideGhost` para calcular un objetivo por `kind` y hacer greedy hacia él (o elección aleatoria cuando el `erratic` está cerca). El estado de partida, `resetPositions` (ya indexa `GHOST_STARTS`) y colisiones no cambian. Prueba: cada fantasma sigue un patrón observablemente distinto.
3. `render.js`: dibujar cada fantasma con `GHOST_COLOR_BY_KIND[ g.kind ]` en vez del color por índice.
4. Comprobación final: partida completa (comer dots, perder vidas, ganar) con los 4 fantasmas.

## Acceptance criteria

- [ ] El juego carga sin errores en la consola y muestra 4 fantasmas.
- [ ] El rojo (`hunter`) se mueve siempre en la dirección que minimiza la distancia a Pac-Man.
- [ ] El rosa (`ambusher`) persigue la celda 4 pasos delante de Pac-Man según su dirección.
- [ ] El cian (`flanker`) calcula su objetivo desde la posición del rojo y un punto delante de Pac-Man (no persigue a Pac-Man directamente).
- [ ] El naranja (`erratic`) persigue a Pac-Man a más de 8 celdas y se mueve aleatoriamente a menos de 8.
- [ ] Los 4 arrancan dentro de la pen y pueden cruzar la puerta.
- [ ] Los 4 comparten `speed = 0.1` (uniforme).
- [ ] Chocar con un fantasma sigue quitando una vida y reseteando posiciones.
- [ ] Comer todos los puntos sigue ganando la partida.

## Decisions

- **Sí:** 4 kinds inspirados en el clásico reutilizando el greedy de `decideGhost` con objetivo por `kind`; es el cambio más pequeño que cumple "cada uno se comporta distinto".
- **Sí:** los 4 empiezan en la pen (los 2 existentes + esquinas `(12,13)` y `(15,13)`); la puerta ya es cruzable solo por fantasmas, no hace falta mecanismo de salida.
- **Sí:** velocidad uniforme `0.1`; la diferenciación recae solo en la IA.
- **Sí:** color por `kind` para que la identidad visual coincida con el comportamiento (rojo siempre persigue).
- **No:** power pellets / modo vulnerable — spec futuro, agrandaría demasiado este.
- **No:** fases scatter/chase, velocidades por fantasma ni salidas escalonadas — complejidad de balance sin pedido.

## Risks

| Riesgo | Mitigación |
| --- | --- |
| El flanqueador depende de la posición del `hunter` | Fallback: si no hay referencia, usa solo el punto delante de Pac-Man |
| Objetivos de `ambusher`/`flanker` caen en paredes o fuera del laberinto | El greedy elige la mejor opción entre celdas transitables; no rompe |
| Umbral del `erratic` (8) es de balance y puede sentirse mal | Constante única `ERRATIC_CHASE_RANGE` ajustable en `game.js` |

## What is **not** in this spec

- Power pellets / modo vulnerable.
- Fases scatter/chase.
- Velocidades distintas por fantasma.
- Salidas escalonadas de la pen.

Cada uno de esos, si llega, va en su propio spec.
