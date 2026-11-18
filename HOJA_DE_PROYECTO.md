# Pop!Pads — Hoja de proyecto

**Versión actual:** v1.13 (18-11-2026) · https://sampledhead-dot.github.io/poppads/

## Descripción
Sampler y looper para niños, con estética hip hop infantil (iconos con carita, colores vivos, contornos gruesos).
Se graba la voz manteniendo pulsado un pad vacío; los golpes se graban en un looper de 4 pistas con ajuste a la
rejilla, swing, metrónomo y deshacer. Disponible en web y en las versiones de Windows, Mac y Android (SH-apps/Releases).

## Bancos
- **A ☀️** y **B 🌙**: 16 pads cada uno (voz grabada, archivo o sonidos de prueba).
- **C 🎹** (nuevo en v1.13): teclado de una octava (8 teclas blancas de colores con carita y 5 negras) con
  **4 sonidos**: Bajo, Piano, Teclado eléctrico y 8 bits, y botones Grave / Agudo (octavas 1–6; cada sonido recuerda la suya).
  Se puede **deslizar el dedo** por las teclas. Las notas se graban en el looper como cualquier pad y recuerdan su sonido.

## Arquitectura
Un solo archivo `index.html` (HTML + CSS + JS, iconos SVG generados en código), `manifest.webmanifest` y `sw.js`
(caché `poppads-v1.13`).
- Audio: Web Audio. Los sonidos del teclado se **sintetizan** al vuelo (no hay muestras):
  - Bajo: seno + 2.º y 3.er armónico, leve caída de tono, saturación suave.
  - Piano: parciales ligeramente inarmónicos, cada uno con su caída (más rápida en los agudos) y golpe de macillo.
  - Teclado eléctrico: FM con «campanita» en el ataque y trémolo.
  - 8 bits: onda cuadrada al 25 %, vibrato de videojuego y resolución reducida.
- Cada nota de cada sonido es un **pad virtual** (índices 32+: `32 + sonido·73 + (midi − 24)`); por eso el looper,
  el deshacer y el guardado de sesión funcionan sin cambios. Los pads virtuales no se guardan (se vuelven a sintetizar).
- Sesión en IndexedDB (pads grabados, looper, banco y sonido/octavas del teclado).
- **Informe de error (.txt)**: se descarga solo ante cualquier error (window.onerror y promesas) y desde
  ⚙️ → «📄 Descargar informe (.txt)». Incluye versión, fecha, dónde, etapa, mensaje y stack, navegador, plataforma,
  núcleos, RAM, pantalla, URL, contexto seguro, COOP/COEP, WebGPU, memoria, audio, opciones, archivo (sin contenido),
  resultado parcial, cronología y últimas 300 líneas de consola.

## Historial
- **v1.13** (18-11-2026): banco C con teclado y 4 sonidos; informe de error .txt; botón de informe en ⚙️.
- v1.12 y anteriores: sampler de voz, bancos A/B, looper de 4 pistas, editor de pads, ajuste de latencia.

## Reglas permanentes
- Siempre pensada para niños: botones grandes, textos cortos con emoji, nada que pueda romper la sesión sin confirmar.
- Estética hip hop infantil (iconos de `IK`, contorno `#2b1055`, colores vivos).
- Al cambiar la versión: `APP_VERSION` y el texto `#ver` en `index.html`, y la caché `C` en `sw.js`.
- Probar en navegador headless antes de entregar (teclado, looper con teclado, recarga de sesión, informe).

## Límites conocidos
- El teclado toca notas sueltas que se apagan solas (no se mantienen mientras se pulsa la tecla).
- Las notas del teclado grabadas en el looper suenan con el sonido con el que se tocaron; las octavas cambian solo lo que se toca después.
- En Android/escritorio funciona igual (misma web empaquetada); no probado en un móvil real en esta versión.
