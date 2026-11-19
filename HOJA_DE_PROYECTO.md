# Pop!Pads — Hoja de proyecto

**Versión actual:** v1.17 (19-11-2026) · https://sampledhead-dot.github.io/poppads/

## Descripción
Sampler y looper para niños, con estética hip hop infantil (iconos con carita, colores vivos, contornos gruesos).
Se graba la voz manteniendo pulsado un pad vacío; los golpes se graban en un looper de 4 pistas con ajuste a la
rejilla, swing, metrónomo y deshacer. Independiente de las demás apps (sin bandeja compartida). Pensada para web y Android.

## Bancos
- **A ☀️** y **B 🌙**: 16 pads cada uno (voz grabada, archivo o sonidos de prueba).
- **C 🎹** (nuevo en v1.13): teclado de una octava (8 teclas blancas de colores con carita y 5 negras) con
  **4 sonidos**: Bajo, Piano, Teclado eléctrico y 8 bits, y botones Grave / Agudo (octavas 1–6; cada sonido recuerda la suya).
  El **Bajo** (v1.15) es **monofónico** y **suena solo mientras se mantiene la tecla**; en el looper cada nota del bajo guarda su duración.
  Se puede **deslizar el dedo** por las teclas. Las notas se graban en el looper como cualquier pad y recuerdan su sonido.

## Mezclador 🎚️ (v1.14)
Botón 🎚️ en la cabecera (al tocar A, B o C se vuelve a los pads). Mientras está abierto, el looper sigue funcionando debajo.
- **4 canales**: A ☀️, B 🌙, C 🎹 (teclado) y Máster ⭐. Cada uno tiene un fader grande con cara que cambia con el volumen (🤫 🙂 😊 😃 🤩), de 0 a 130 %. **Por defecto (v1.15): canales al 70 % y máster al 80 %**, para que no sature. Doble toque en el fader lo devuelve a su punto inicial. Cada canal tiene medidor de nivel (verde, amarillo y rojo), 🔊/🔇 (silencio) y 🎧 (solo; se pueden juntar varios). En el máster, ↺ devuelve todo a los valores iniciales.
- **3 pads de efectos táctiles** (como los de SampledHead App, simplificados): suenan mientras se mantienen pulsados; con 📌 se quedan puestos.
  - 🌊 **Filtro**: a la izquierda «🐻 Grave» (paso bajo), a la derecha «🐭 Agudo» (paso alto); hacia arriba, más «silbido» (resonancia).
  - 🏔️ **Eco** (delay estéreo de ida y vuelta al tempo): de izquierda a derecha lento 🐢 1/4, saltarín 🐸 1/8 con puntillo, rápido 🐇 1/8 y súper rápido ⚡ 1/16; hacia arriba, más eco. Al soltar, el eco se apaga solo.
  - 🔁 **Repetir** (beat repeat): repite el último trozo con sonido en 1/4, 1/8, 1/16 o 1/32; hacia arriba, más cortito. Usa un AudioWorklet; sin él, avisa de que no está disponible.
  - El tempo es el del looper, o el del click (⏱) si aún no hay ritmo.
- Cadena: canales → suma → Repetir → Filtro (+ envío al Eco) → Máster → limitador suave. El click del metrónomo va directo a la salida, sin mezclador ni efectos.
- La mezcla se guarda en la sesión y en `localStorage` (`pp_mix`, con `ver:2`); 🏠 «Empezar de nuevo» vuelve a los valores iniciales y apaga los efectos. Las mezclas guardadas antes de la v1.15 (sin `ver:2`) pasan una vez a los valores nuevos.

## Rendimiento (v1.14)
- Los golpes de los pads y el pulso del tempo se animan con la Web Animations API. Antes se quitaba una clase, se forzaba un recálculo de la página (`offsetWidth`) y se volvía a poner, en cada golpe.
- El cabezal del looper se mueve con `transform` (el navegador no recoloca la página en cada fotograma) y el contador de grabación solo se reescribe cuando cambia.
- Las mini-ondas de los pads solo se redibujan si cambia su sonido o sus puntos.
- Piano del teclado: los parciales se calculan girando fasores en vez de llamar a `Math.sin`/`Math.exp` en cada muestra. Es de 3 a 10 veces más rápido y la señal es idéntica (diferencia de −100 dB).
- Las teclas visibles se preparan una a una en ratos libres (`requestIdleCallback`), empezando por el centro, en vez de todas de golpe. En un móvil modesto simulado (CPU ×4 más lenta), elegir el piano bloqueaba la app 5,3 s; ahora el bloqueo más largo es de 90 ms.
- El procesador de Repetir solo existe con el mezclador abierto (o con Repetir 📌). Los medidores solo se leen con el mezclador a la vista.
- `contain` en los pads, `will-change` en lo que se anima y sin estrellas flotantes si el sistema pide «reducir movimiento».

## Mis canciones y grabaciones (v1.17)
- Base IndexedDB aparte, `poppads-canciones`: almacén `songs` (copia completa de la sesión: el `meta` y los `b<i>` de `poppads`) y `recs` (WAV 16 bits estéreo).
- Abrir una canción = vaciar la sesión (como 🏠) + copiar sus datos a `poppads` + `loadSession()`. Si hay algo hecho, «¿Abrir?» pide un segundo toque.
- Archivo `.poppads`: `PPAD1` + u32 largo del JSON + JSON (nombre, animalito, claves) + audio float32 seguido.
- Grabar: `ScriptProcessor` estéreo colgado de la salida del limitador (`clipN`); máximo 10 min.
- 📤 usa «Compartir» del sistema si puede enviar archivos (Android, iPhone); si no, descarga el archivo.

## Arquitectura
Un solo archivo `index.html` (HTML + CSS + JS, iconos SVG generados en código), `manifest.webmanifest` y `sw.js`
(caché `poppads-v1.15`).
- Audio: Web Audio. Los sonidos del teclado se **sintetizan** al vuelo (no hay muestras):
  - Bajo: seno + 2.º y 3.er armónico, leve caída de tono, saturación suave; 6 s casi sostenidos. `bassPlay` lleva la lista de voces (`BV`): cada nota nueva corta la anterior (25 ms) y, si hay otra programada después, acaba ahí; al soltar la tecla, `bassStop`. En el looper la nota guarda `d` (duración; `null` mientras se mantiene, y entonces no se repite).
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

## Pantalla (v1.15)
- Margen superior con `env(safe-area-inset-top)` (variables `--sat`/`--sab`) en la app y en el editor: la cabecera ya no queda bajo la hora y la batería.
- Respaldo para la app Android 1.1.1 o anterior (no deja el hueco de las barras): si el sistema no da margen, existe `SHAndroid` sin `insets()` y es Android 15+ (o la web ocupa toda la pantalla), se añaden 30 px arriba. Desde los paquetes 1.1.2 el propio programa deja el hueco.
- Pads casi cuadrados: `#pads` tiene altura máxima `(ancho − 20 px) / 0,92`; lo que sobra en móviles alargados va a las pistas del looper (de 30 a 50 px de alto, más fáciles de tocar).

## Historial
- **v1.17** (19-11-2026): botón 💾 «Mis canciones» (también en la pantalla de inicio): guardar varias canciones (sonidos, ritmo y mezcla) con un animalito y nombre, abrirlas, cambiarles el nombre y borrarlas; 🔴 «Grabar lo que suena» (todo lo que sale por el máster con efectos, sin el click; barra roja con el tiempo y ■ Parar; se quita el silencio del principio y del final; se guardan las 20 últimas); 📤 compartir (móvil) o guardar el WAV y la canción (.poppads) y 📥 abrir un .poppads de otro móvil u ordenador. Se quita la conexión con las otras apps (1.16): Pop!Pads es independiente.
- 1.16 (19-11-2026, retirada en 1.17): conexión con las otras apps de Sampled Head.
- **v1.15** (19-11-2026): bajo monofónico que suena mientras se pulsa (también en el looper); mezclador más bajo por defecto (70 % / máster 80 %) para que no sature; la cabecera ya no queda bajo la barra de estado; pads casi cuadrados y pistas del looper más altas.
- **v1.14** (18-11-2026): mezclador 🎚️ con canales A, B, teclado y máster (fader, medidor, silencio, solo) y 3 pads de efectos táctiles (Filtro, Eco, Repetir); mejoras de rendimiento (teclado sin bloqueos, animaciones sin recálculos de página).
- **v1.13** (18-11-2026): banco C con teclado y 4 sonidos; informe de error .txt; botón de informe en ⚙️.
- v1.12 y anteriores: sampler de voz, bancos A/B, looper de 4 pistas, editor de pads, ajuste de latencia.

## Reglas permanentes
- Siempre pensada para niños: botones grandes, textos cortos con emoji, nada que pueda romper la sesión sin confirmar.
- Estética hip hop infantil (iconos de `IK`, contorno `#2b1055`, colores vivos).
- Al cambiar la versión: `APP_VERSION` y el texto `#ver` en `index.html`, y la caché `C` en `sw.js`.
- Probar en navegador headless antes de entregar (teclado, looper con teclado, recarga de sesión, informe).

## Límites conocidos
- Mezclador y efectos probados en Chrome sin pantalla, midiendo el audio de salida; no se han probado en un móvil real ni en Safari de iPhone (Repetir necesita AudioWorklet, disponible en iOS 14.5 o posterior).
- Los efectos afectan a lo que suena en directo. Pop!Pads no exporta audio, así que no hay nada más a lo que aplicarlos.
- Piano, teclado eléctrico y 8 bits tocan notas que se apagan solas (no se mantienen mientras se pulsa); solo el Bajo se sostiene, hasta 6 s.
- Las notas del teclado grabadas en el looper suenan con el sonido con el que se tocaron; las octavas cambian solo lo que se toca después.
- En Android/escritorio funciona igual (misma web empaquetada); no probado en un móvil real en esta versión. El margen de la barra de estado no se ha podido ver en un móvil real (probado en Chrome sin pantalla, simulando el margen).
