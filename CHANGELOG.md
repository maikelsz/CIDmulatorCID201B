# Registro de cambios - CID 201-B

Un archivo por hito. Nunca se sobrescribe el hito anterior.

## v0.17.0 - `CID201B_17.html` (2026-11-16)

**Direccionamiento corregido, ayuda, contador de visitas y REC**

- Nuevo `CID201B_17.html`; `CID201B_16.html` queda intacto.
- **Puente indirecto corregido a 5 + 7**: página (5 bits, 0–31) + offset
  (7 bits). El campo (3 bits) no viaja en el puente: se toma de `RCD` para
  datos, de `RCI` para `VEA(I)` y de `RNI` para `VAS(I)`. La dirección absoluta
  de 15 bits se forma fuera del puente. El manual de 1978 (p. 11) dice «Puente
  RP + RI2»; la partición 3+2+7 era una inferencia editorial errónea. Se toma
  como referencia el esquema de la PDP-8/I.
- **`VAS(I)` es la única instrucción que consume `RNI`**: guarda el campo de
  origen en `RVI` (`RVI ← RCI`) y pasa `RNI` a `RCI`. **`VEA(I)`** salta dentro
  del campo actual sin tocar `RNI` ni `RVI`. **`VPP`** (`7700`) restaura `RCI`
  desde `RVI` y ejecuta `VEA(I) 0000`. El ciclo normal de instrucción no
  modifica `RNI`; `CCIN` lo prepara para la próxima `VAS(I)`.
- **Protección ternaria y solo indirecta**: `CAMPO` y `ZONA` son patrones de
  3 bits (`1` exige 1, `0` exige 0, `~` no importa). Se aplica únicamente a
  `ISC`, `DEL` y `VAS` indirectos, como indica el manual (p. 14). La zona del
  puente es `(b >> 9) & 7`; el campo de `VAS(I)` es el nuevo `RNI`/`RCI`. Se
  elimina la comprobación directa de `ISC`/`DEL`.
- **Ayuda interna**: orden **Uso, Repertorio, Protección, UAB, Periféricos**.
  Se corrigieron las explicaciones del puente, `CCIN`, `VAS(I)`, `VEA(I)`,
  `VPP` y la protección. La UAB sigue documentada como **no implementada**.
- **Contador de visitas y visitantes** Vercount en el pie de la página, con el
  mismo patrón que el emulador de la CID 201-A. Es la única petición externa;
  sin red, el contador queda en «Loading» y el emulador funciona igual.
- **Demo «Última Página»** corregida: `7751` pasa de `5202` (`VEA 7602`) a
  `5712` (`VEA I 7712`). El slot `7712` lo escribe cada `VAS display` con la
  dirección de retorno; no es un puente fijo.
- **REC corregido**: antes terminaba con `VEA 7714`, sin ejecutar `VAS 7712`, y
  `end` borraba `RECstate`. La primera pulsación leía `[7600] = 7140`, pero la
  segunda tomaba `RegNum = 7140` como dirección y leía `[7140] = 7500`. Ahora
  REC prepara `AC = RegNum`, llama al display con `VAS 7712` y vuelve a `init`
  (no a `end`), conservando `RECstate`. Secuencia verificada:
  `7600 → 7140 → 5210 → 7140`.
- **Variables de ALM**: `ALMdir` y `ALMstate` pasan de `7663`/`7664` a
  `7772`/`7773` para dejar sitio a la llamada `VAS display` de REC.
- Fuente, `.lst`, `.len1`, `.lstb` y listado regenerados con `asm201b.py`; las
  128 palabras del `.lstb` coinciden con la demo precargada.

**Pruebas y documentación**

- Banco ampliado a **123/123 pruebas, 518 comprobaciones**, sin errores de
  consola. Grupos reescritos o ampliados: `02-puente.js` (14 pruebas, 46
  comprobaciones), `04-proteccion.js` (12, 36), `05-limites-errores.js`
  (9, 56), `06-estado-entrada-salida.js` (10, 21) y `07-codec.js` (15, 97).
- Nuevas pruebas de REC: continuación desde `7600`, REC como primera operación
  sin `[7712]` previo y reinicio de la secuencia con `BOR`.
- En `_16` y `_15` pasan 98/123: fallan las 25 pruebas del modelo 5+7, la
  protección ternaria, el límite de 32 páginas, el cambio de módulo con
  `CCIN`/`VAS(I)` y cuatro del monitor «Última Página».
- `TEST/emulador-core.js` regenerado con `_17` y `_16`; el archivo por defecto
  del banco es `_17`.
- Documentación corregida: `README.md`, `DOChistorico/CID201B.md`,
  `DOChistorico/CID201B.extract.txt`, `DOCwork/ANALISIS-direccionamiento-201B.md`,
  `DOCwork/TODO.md`, `TEST/README.md` y `DOCwork/HITO-17.md`.
- Título: v0.17.0.
- `CIDmulatorCID201B.html` (copia publicada) actualizado a la v0.17.0; es
  idéntico por SHA-256 a `CID201B_17.html`.

**PENDIENTE-DOC**

- Sigue sin confirmar si `RFD` debe desplazarse con `RDP` al ejecutar `TAR`.
- La UAB y los periféricos de alta velocidad siguen sin implementarse.

## v0.16.0 - `CID201B_16.html` (2026-11-16)

**Ayuda interna con pestañas y protección visible**

- Nuevo `CID201B_16.html`; `CID201B_15.html` queda intacto.
- La ayuda interna se organiza en cinco pestañas: **Uso**, **Protección**,
  **UAB**, **Periféricos** y **Repertorio**. Solo una sección está visible a la
  vez; cada pestaña tiene `role="tab"` y su panel `role="tabpanel"`.
- Las secciones de protección, periféricos y repertorio se movieron a su
  pestaña, sin duplicar contenido.
- La pestaña **Protección** documenta el alcance (`ISC`, `DEL`, `VAS`
  indirecto), la tabla de posiciones `1 / ~ / 0` (solo `0` prohíbe; `~` es la
  intermedia) y el campo `PROT` de la barra de estado:
  `PROT: PERM CAM: 1~0 ZON: 111`.
- Nueva pestaña **UAB**: describe el bloque `65xx`, el Registro de Estado y los
  formatos, con el aviso destacado de que **no está implementada**.
- Nueva pestaña **Repertorio**: conserva las tablas de IRM,
  microinstrucciones, puente y E/S.

**Pruebas y documentación**

- Nuevo grupo `12-ayuda-proteccion.js`: 2 pruebas y 6 comprobaciones sobre
  `protPosChar()` y el resumen `CAMPO`/`ZONA`.
- Resultado en `_16`: **115/115 pruebas, 460 comprobaciones**, sin errores de
  consola. `_15` también pasa 115/115 porque la función probada ya existía; en
  `_14` pasan 103/115 (442); en `_13`, 94/115 (372); en `_12`, 84/115 (340);
  en `_11`, 77/115 (318).
- `TEST/emulador-core.js` regenerado con `_16` y `_15`; el archivo por defecto
  del banco es `_16`.
- `TEST/README.md`, `DOCwork/TODO.md` y `DOCwork/HITO-16.md` actualizados.
- `README.md` actualizado: badges v0.16.0 y 115/115, archivo activo `_16`,
  descripción de las pestañas, campo `PROT` y decisiones del hito.
- Título: v0.16.0.
- `CIDmulatorCID201B.html` (copia publicada) actualizado a la v0.16.0; es
  idéntico por SHA-256 a `CID201B_16.html`.

**PENDIENTE-DOC**

- Sigue sin confirmar si `RFD` debe desplazarse con `RDP` al ejecutar `TAR`.

## v0.15.0 - `CID201B_15.html` (2026-11-16)

**Saltos combinados, `SEC`/`PUE` y mapa de memoria editable**

- Nuevo `CID201B_15.html`; `CID201B_14.html` queda intacto.
- **`SEC` (`7206`)** ahora salta con `EN == 0`. La tabla anterior decía
  `EN <> 0`, una errata que repetía la condición de `SED`. Corregido también en
  `DOChistorico/CID201B.md`.
- **`PUE` (`7520`)** pone `EN = 1`. Antes caía en la rama de `CEN` porque el
  código contiene el bit `0o020`; la comprobación exacta se movió antes.
- **Saltos combinados.** El grupo 1 (`72xx`/`73xx`) usa un decodificador por
  bits: `SED`/`SAC`/`SAN` se combinan por **OR** y `SEC`/`SAD`/`SAP` por
  **AND**. El bit `0o002` selecciona el grupo AND; `SLT` (`7202`) es el AND
  vacío y salta siempre. Los grupos OR y AND no se mezclan. Nuevos ejemplos
  documentados: `7230` (AC ≤ 0), `7214` (EN ≠ 0 o AC = 0), `7216` (EN = 0 y
  AC ≠ 0) y `7330` (LAC + salto).
- **`ALT`**: sin cambios de código. Se documenta que la tecla del CODEC produce
  **Alto Dinámico (`AD`)** y la instrucción `ALT` (`7201`) produce **Alto de
  Programa (`AP`)**.
- **Mapa de memoria editable.** Clic en el valor octal para editar; se aceptan
  de 1 a 4 dígitos del 0 al 7. `Enter` guarda y avanza; `Tab` guarda y avanza;
  `Esc` cancela. Un valor inválido se marca en rojo y no se escribe. Al final
  de un módulo el editor pasa al siguiente. Exige alimentación y, si la máquina
  ejecuta, la detiene en `AD`. Nuevas funciones `parseMMValue()` y
  `mmCommitCell()`.
- **Ayuda interna** corregida: `ENTRADA A/B/C` = *Shift*, *Hola Mundo* y
  monitor; *Eco* fuera de la tecla C; explicación de `AD`/`AP`; tabla con
  `SEC`, `SAD`, `SAP`, `LEN`, `PUE` y `POE`; combinaciones OR/AND; mapa
  editable y texto de `ALTO`.

**Pruebas y documentación**

- Nuevos grupos `10-saltos-combinados.js` (10 pruebas) y
  `11-mapa-editable.js` (6 pruebas): 16 pruebas y 54 comprobaciones nuevas.
- Resultado en `_15`: **113/113 pruebas, 454 comprobaciones**, sin errores de
  consola. En `_14` pasan 101/113 (436); en `_13`, 92/113 (366); en `_12`,
  82/113 (334); en `_11`, 75/113 (312).
- `TEST/emulador-core.js` regenerado con `_15` y `_14`; el archivo por defecto
  del banco es `_15`.
- `TEST/README.md`, `DOCwork/TODO.md` y `DOCwork/HITO-15.md` actualizados.
- `README.md` actualizado: badges v0.15.0 y 113/113, tabla de saltos con grupos
  OR/AND, sección de control del `EN`, mapa editable, limitaciones y
  decisiones del hito.
- Título: v0.15.0.
- `CIDmulatorCID201B.html` (copia publicada) actualizado a la v0.15.0; es
  idéntico por SHA-256 a `CID201B_15.html`.

**PENDIENTE-DOC**

- Sigue sin confirmar si `RFD` debe desplazarse con `RDP` al ejecutar `TAR`.

## v0.14.0 - `CID201B_14.html` (2026-11-15)

**Memoria de ferrita, reset y barra de estado**

- Nuevo `CID201B_14.html`; `CID201B_13.html` queda intacto.
- La memoria se comporta como ferrita: apagar y volver a encender **no la
  borra**. `setPower(true)` ya no llama a `loadPrograms()`; las demos de fábrica
  solo se cargan al abrir el emulador.
- Botón de **reset ⟲** en la **extrema izquierda de la barra de estado**, con
  id `btn-soft-reset`. Se deshabilita cuando la máquina está apagada.
- El reset solo actúa con la máquina encendida. Detiene la ejecución, inicializa
  la memoria con las demos de fábrica, limpia todos los registros de CPU,
  punteros, autoíndices, estado de ejecución y estado de la Unidad de Enlace, y
  deja la máquina en **ALTO DINÁMICO (AD)**.
- El reset **conserva la posición y el estado de los interruptores** (SW, DATO e
  INSTRUCCIÓN), la llave, el modo de protección, el modo de ejecución, la
  alimentación de la Unidad de Enlace y los interruptores de falta de
  alimentación e interrupción. No toca el CODEC, el Ricoh ni el Consul.
- Un programa `.lstb` cargado se pierde al pulsar reset; el apagado y encendido,
  en cambio, lo conservan.
- En la barra de estado, a la izquierda de `SW`, aparecen `DATO:` e `INSTR:`
  con el valor de sus tres interruptores (`getSwitchValue3()`), además de los
  campos ya existentes.
- El cargador `.lstb` mantiene el rechazo de la máquina apagada, pero el mensaje
  ya no afirma que el encendido borra la memoria.
- La ayuda interna incluye «Memoria de Ferrita, Encendido y Reset».

**Pruebas y documentación**

- Nuevo grupo `09-ferrita-softreset.js`: 9 pruebas y 89 comprobaciones.
- Resultado en `_14`: **97/97 pruebas, 400 comprobaciones**, sin errores de
  consola. En `_13` fallan solo las 9 pruebas del hito 14 (88/97); en `_12`
  fallan además las 11 del cargador (77/97); en `_11`, las 7 del CODEC (70/97).
- Nuevo `TEST/emulador-core.js` con copias de `_14` y `_13`, generado por
  `DOCwork/tools/build_test_core.py`. `CID201B-tests.html` funciona ahora al
  abrirse directamente como `file://`, sin `NetworkError`, y muestra la etiqueta
  «copia incluida». El botón **Ejecutar** se verificó también en ese modo.
- `TEST/README.md`, `DOCwork/TODO.md` y `DOCwork/HITO-14.md` actualizados.
- Título: v0.14.0.
- `README.md` reescrito por completo con la estructura y profundidad del README
  de la CID 201-A, usando solo documentación, decisiones e implementación de la
  201-B.
- `CIDmulatorCID201B.html` (copia publicada) actualizado a la v0.14.0; es
  idéntico por SHA-256 a `CID201B_14.html`.

**PENDIENTE-DOC**

- Los pendientes del CODEC siguen sin confirmar: desplazamiento de `RFD` con
  `TAR` y tipo de alto de `ALT` (`AD`, `AP` o ambos).

## v0.13.0 - `CID201B_13.html` (2026-11-15)

**Cargador de programas compilados `.lstb`**

- Nuevo `CID201B_13.html`; `CID201B_12.html` queda intacto.
- El botón de carpeta de la barra inferior abre el selector de archivos y carga
  el programa en memoria. Solo se acepta la extensión `.lstb`; cualquier otra
  se rechaza con un aviso.
- Formato por líneas: dirección y palabra en octal, separadas por coma,
  tabulador o espacios (`7600, 7140`). Se ignoran comentarios que empiezan por
  `;`, `#`, `//` o `/*`.
- Validación completa antes de escribir: dirección octal de hasta 15 bits
  (`77777`), palabra octal de hasta 12 bits (`7777`), duplicados (gana la
  última), página 0 y líneas sin datos. Un archivo inválido no deja la memoria a
  medio cargar.
- Si la máquina está ejecutando, el cargador la detiene primero con alto `AD`;
  después deposita la memoria y la deja en `ALTO`, sin arrancarla.
- El `.lstb` contiene **solo memoria**: no define la dirección de inicio. El
  cargador no modifica `CP`, `RCI`, `RCD`, `RNI` ni `RVI`. Tras cargar, el
  usuario fija el vector con los interruptores, pulsa `CONTINUAR` y luego
  `COMENZAR`.
- Aviso flotante con el número de palabras, el alto automático y las
  instrucciones para fijar el vector.
- Se expone `window.cargarLstbEnMemoria(texto)` para automatización y pruebas.

**Ejemplo y herramienta**

- `EJEMPLOS/MONITOR_Ultima_Pagina_201B.lstb`: monitor «Última Página» con sus
  128 palabras, listo para cargar.
- `CIDcoder/CODE/LEN CID201B/MONITOR_Ultima_Pagina_201B.lstb`: el mismo
  programa junto al fuente y al listado.
- `DOCwork/tools/asm201b.py`: nueva opción `--lstb-out` para generar el
  programa compilado.

**Pruebas y documentación**

- Nuevo grupo `08-lstb.js`: 11 pruebas y 51 comprobaciones.
- Resultado en `_13`: **88/88 pruebas, 311 comprobaciones**, sin errores de
  consola. En `_12` pasan los grupos 00–07 y fallan las 11 pruebas del
  cargador (77/88).
- `TEST/README.md`, `DOCwork/TODO.md` y `DOCwork/HITO-13.md` actualizados.
- Título: v0.13.0.

**PENDIENTE-DOC**

- Los pendientes del CODEC siguen sin confirmar: desplazamiento de `RFD` con
  `TAR` y tipo de alto de `ALT` (`AD`, `AP` o ambos).

## Entregables de código (2026-11-14)

Sin cambios en `CID201B_12.html`.

- `CIDcoder/CODE/LEN CID201B/MONITOR_Ultima_Pagina_201B.len1`: fuente
  ensamblador del monitor, con la codificación de la 201-B.
- `CIDcoder/CODE/LEN CID201B/MONITOR_Ultima_Pagina_201B.lst`: listado con
  dirección, palabra, nemotécnico/operando y comentario, al estilo de la
  201-A.
- `CIDcoder/CODE/LEN CID201B/README.md`: alcance, codificación, formato y
  comprobación.
- `DOCwork/tools/asm201b.py`: ensamblador mínimo para regenerar el `.lst` y
  compararlo con el `.len1` extraído.
- Verificación: **128/128 palabras** coinciden con `CID201B_12.html` y con
  `EJEMPLOS/MONITOR_Ultima_Pagina_201B.len1`. El listado de tres columnas
  generado con `--len1-out` es idéntico byte por byte al `.len1` extraído.

## v0.12.0 - `CID201B_12.html` (2026-11-14)

**Consola CODEC**

- `RE` de 4 bits: el teclado guarda el dígito completo (`num & 0xF`), incluidos
  8 y 9; ya no los convierte en 0 y 1.
- `RDP` de 8 posiciones de 4 bits y `RFD` de 8 bits (un punto por posición).
- `TAR`: desplaza `RDP` una posición a la izquierda, inserta `AC & 0xF` por la
  derecha y limpia `AC`. No desplaza `RFD`.
- `TAF`: carga los 8 bits bajos del `AC` en `RFD` (bit 7 → posición 0) y limpia
  el `AC`.
- `POS` pone el signo a 0 (más, `+`) y `PUS` lo pone a 1 (menos, `−`); ninguno
  limpia el `AC`. Se añadió `codecDisplaySign()` para separar el bit del
  carácter que se muestra.
- Ayuda interna y tabla de instrucciones corregidas para la nueva polaridad.

**Demo «Última Página»**

- El monitor octal vuelve a ser funcional: `5`, `7` → `00000057`; el 8 se
  descarta a nivel de programa; `BOR` → `00000005`; `ALM` escribe y avanza;
  `REC` lee `00000006` y luego `00007500`.
- El display usa máscaras de 3 bits más `RD3` en lugar de `RI3`. `RI3`/`RD3`
  rotan a través de `EN`, y la extracción por `RI3` arrastraba bits de acarreo.
- La validación octal comprueba el bit 3 de `RE` (dígito nuevo), no el
  candidato acumulado.
- El display vuelve fijo a `7602`: el puente 3+2+7 no puede direccionar la
  página 31, donde vive la demo. No se cambió la semántica del núcleo.

**Extracción**

- `EJEMPLOS/MONITOR_Ultima_Pagina_201B.len1`: 128 líneas de la página 31
  completa, con tres columnas separadas por tabulador (dirección octal,
  contenido octal y nemotécnico o valor octal).
- La demo sigue precargada en el emulador; el `.len1` es una extracción.

**Pruebas y documentación**

- Nuevo grupo `07-codec.js`: 12 pruebas y 22 comprobaciones.
- Resultado en `_12`: **77/77 pruebas, 260 comprobaciones**, sin errores de
  consola. En `_11` pasan los grupos 00–06 y fallan las 7 pruebas nuevas del
  CODEC (70/77).
- `TEST/README.md`, `DOCwork/TODO.md` y `DOCwork/HITO-12.md` actualizados.
- Título: v0.12.0.

**PENDIENTE-DOC**

- Si `RFD` debe desplazarse junto con `RDP` al ejecutar `TAR`.
- Si la tecla `ALT` del CODEC debe dar alto `AD`, `AP` o ambos; el código
  actual usa `AD`.

## v0.11.0 — `CID201B_11.html` (2026-11-14)

**Núcleo**

- Puente indirecto con la partición documentada **campo(3) + página(2) + offset(7)**.
  El campo del puente manda en la dirección efectiva.
- `RNI`/`CCIN`: `RCI ← RNI` en cada frontera de instrucción; `RNI` por defecto
  sigue el campo actual. `762N` cambia de módulo de verdad.
- `VAS(I)`/`VEA(I)`: `RVI ← RCI` (origen) y `RCI = RNI = campo del puente`.
- `INT1`: `RVI` guarda el campo de origen.
- Protección: zona = 3 bits altos del puente; `ISC`/`DEL` contra `RCD`,
  `VAS(I)` contra `RCI`; zona directa por página destino.
- `doCD()` usa el código de operación de la instrucción efectiva (`VPP`).
- `doCI`/`doCM`: las microinstrucciones y la E/S (`op >= 6`) se ejecutan en `CM`
  sobre la misma instrucción; se retrocede el `CP` y no se decodifican como
  indirectas.
- Eliminada `cidSelfTest` y sus accesos (`?selftest=1`, `Ctrl+Alt+T`, botón y ayuda).
- Banco de pruebas externo en `TEST/`: 65/65 pruebas, 181 aserciones en `_11`;
  `_10` falla 22, lo que detecta las regresiones.
- Terminología: se distingue el **Registro de Enlace (EN)** de la **Unidad de
  Enlace (ENLACE)** en la ayuda, los comentarios y las pruebas.
- Documentación: `DOChistorico/CID201B.md` validado (987 líneas, 44 tablas,
  0 errores) con seis rejillas gráficas eliminadas a mano; cobertura sin
  diferencias inesperadas.
- Ayuda: apartado de direccionamiento indirecto.
- Título: v0.11.0.

**No incluido** (hitos siguientes): CODEC decimal, cinta del Ricoh, Unidad de
Enlace y editor de memoria, UAB 65xx, periféricos de alta velocidad.
