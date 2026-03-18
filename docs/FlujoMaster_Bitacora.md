# FlujoMaster — Bitácora de Trazabilidad
**Autor:** Daniel Arauz Sanjuan· Practicas empresariales Foscal. 
Floridablanca, Santander, CO
**Propósito:** Registro cronológico de interacciones, decisiones, errores y aprendizajes. Memoria de entrenamiento del proyecto para ser leída por Claude en cualquier sesión futura.

## AVISO DE LECTURA — PARA CLAUDE

Esta bitácora es un registro cronológico documental. Registra el proceso
de toma de decisiones, errores cometidos, correcciones aplicadas y la
evolución del pensamiento a lo largo del proyecto.

**No extraer reglas ni fórmulas de aquí.** Las versiones anteriores de
fórmulas y decisiones que aparecen en entradas antiguas pueden haber sido
superadas por entradas posteriores. La fuente de verdad operativa siempre
es INSTRUCCIONES_PARA_CLAUDE.md y FlujoMaster_EstiloVisual.md.

**Sí leer esta bitácora completa** antes de cada sesión. Su valor está en
transmitir el razonamiento detrás de cada decisión, la forma de pensar y del razonamiento interno, y los errores ya cometidos — para no repetirlos. Entender el
"por qué" de cada regla es tan importante como la regla misma.

---

## [2026-03-13] Sesión 1 — Construcción inicial del sistema

**Contexto:** Se construyó FlujoMaster desde cero como sistema de análisis de procedimientos para generar diagramas draw.io en formato XML.

**Decisiones tomadas:**
- Los procedimientos clínicos son siempre lineales y secuenciales — nunca inferir paralelismos
- Las columnas swimlane deben derivarse literalmente del campo "Persona Responsable" — nunca fusionar por similitud
- El XML no usa swimlanes nativos de draw.io — se construye con elementos independientes y coordenadas absolutas
- Se definieron reglas R1–R11 como base del sistema

---

## [2026-03-13] Sesión 2 — Primer documento procesado

**Documento:** Gestión del Riesgo Operacional V3 
**Estado:** XML generado y validado ✅
**Aprendizaje:** El flujo del primer documento confirmó que la lectura literal de responsables funciona — no hubo ambigüedades en la asignación de columnas.

---

## [2026-03-13] Sesión 3 — Detección de fallas estructurales en el XML

**Documento:** Ajuste y Adecuación de Medicamentos No Oncológicos V5

**Problema detectado por Daniel:** El XML generado tenía errores estructurales al no ver el flujograma real.
**Errores encontrados:**
- Nodos omitidos: pasos de rama sin número no se detectaban
- Act.5 representada como rombo siendo una acción
- R13 declarada pero no ejecutada en coordenadas
- Shape incorrecto para conectores de paginación

**Causa raíz:** No existía regla para detectar rombos implícitos dentro de la descripción de una actividad.

**Decisión:** Crear R16 — un numeral puede generar dos nodos: rectángulo siempre, y rombo adicional si la descripción contiene una pregunta (¿...?)

---

## [2026-03-13] Sesión 4 — Corrección de medidas visuales

**Problema:** XML desbordaba los límites de página carta en draw.io.
**Intento 1:** Reducir ancho de columna de 400px a 350px → seguía desbordando con 5 cols (1750px > 1169px).
**Intento 2:** Medir directamente desde imagen de referencia con PIL.

**Aprendizaje:** Las medidas no se estiman — se miden. El error principal era la separación entre nodos: se usaba 20px cuando la real era 60px. Eso causaba que los nodos se apilaran demasiado arriba y el último rombo cayera fuera del margen inferior.

**Decisión:** Derivar todas las medidas por píxeles desde el flujograma de referencia. Carta landscape real en draw.io = 1056×816px (11×8.5 pulgadas a 96dpi).

---

## [2026-03-13] Sesión 5 — Rediseño completo del sistema de paginación

**Problema:** El sistema de paginación por conteo de nodos generaba páginas desiguales con espacio vacío enorme.

**Intento 1 (Claude):** Paginar por px acumulados por columna → desperdiciaba espacio en columnas vacías.
**Intento 2 (Claude):** Paginar por conteo de nodos → nodos de distinta altura generaban páginas desiguales.

**Observación de Daniel:** "es mejor verlo con coordenadas" — propuso el modelo de grid tipo "batalla naval" donde cada nodo ocupa una celda (fila × columna).

**Aprendizaje:** El flujo swimlane es una matriz. La paginación debe calcularse por filas lógicas, no por nodos individuales ni por px de columna. Cada fila ocupa la misma Y independientemente de en qué columna esté el nodo.

**Decisión:** Sistema de grid con filas lógicas. Máximo 9 filas por página, tolerancia 10, con redistribución equitativa cuando el sobrante es ≤ 3 filas.

**Observación de Daniel sobre tolerancia:** "no tiene sentido que sean 10 nodos y que justo se hagan 2 páginas porque por los límites puestos sobraría uno y lo usaríamos para toda la página completa, lo cual es absurdo". La lógica correcta es estirar antes de abrir página nueva.

**Fórmula aprobada:**
```
si total_filas <= 10 → 1 página
sino:
    páginas = ceil(total_filas / 9)
    sobrante = total_filas - ((páginas-1) * 9)
    si sobrante <= 3 → redistribuir equitativamente
    sino → mantener 9 por página, última absorbe sobrante (máx 10)
```

---

## [2026-03-13] Sesión 5 cont. — Columnas vacías en páginas

**Problema:** Columnas sin ningún nodo en una página seguían dibujándose y desperdiciando espacio horizontal.

**Decisión:** Por cada página, solo se dibujan las columnas activas (con al menos 1 nodo). El ancho de columna se recalcula dinámicamente: `floor(1016 / cols_activas)`.

---

## [2026-03-13] Sesión 6 — Lógica de columnas y distribución en bloques

**Observación de Daniel:** Mismo razonamiento de tolerancia aplica para columnas — si son 7 mejor distribuir 4+3 que forzar en 5.

**Fórmula aprobada:**
```
si total_cols <= 6 → 1 bloque
sino:
    bloques = ceil(total_cols / 5)
    sobrante = total_cols - ((bloques-1) * 5)
    si sobrante <= 2 → redistribuir equitativamente
    sino → 5 por bloque, último absorbe sobrante (máx 6)
```

---

## [2026-03-13] Sesión 6 cont. — Mini-grid interno y reglas de celdas

**Decisiones tomadas:**
- Cada celda se divide en 4 unidades horizontales internas
- Pentágono (paginación) y nodo referencia A/B/C ocupan 1/4 horizontal — nunca una celda completa
- Los elementos parciales se distribuyen horizontalmente (no verticalmente) dentro de la celda
- Rombo de decisión SIEMPRE solo en su celda — nunca comparte
- Pentágono NUNCA solo, salvo que filas_página ≤ 7 (límite_base - 2)
- Nodo referencia A/B/C NUNCA solo, sin excepción

**Observación de Daniel:** "los nodos de salto de letras tampoco deberían estar solos jamás"

**Forma del pentágono:** Casa invertida — vértice apuntando hacia abajo (no flecha horizontal, no casa normal).

---

## [2026-03-13] Sesión 6 cont. — Escala proporcional en cascada

**Problema previo:** Font size fijo de 12pt causaba texto desbordando nodos en celdas pequeñas.

**Observación de Daniel:** "no solamente el texto sino también la figura" — todo debe escalar proporcionalmente.

**Decisión:** Todo en cascada desde las dimensiones de página:
```
ancho_celda  = floor(1016 / cols_activas)
alto_celda   = floor(718  / filas_página)
ancho_nodo   = floor(ancho_celda * 0.85)
alto_nodo    = floor(alto_celda  * 0.55)
font_size    = max(6, min(10, floor(alto_nodo * 0.25)))
```
Figura y texto nunca salen de su celda. whiteSpace=wrap + overflow=hidden obligatorio.

---

## [2026-03-13] Sesión 7 — Algoritmo de posicionamiento de nodos en el grid

**Problema:** Los nodos se colocaban una fila abajo cada vez que cambiaban de columna, aunque debían ir en paralelo.

**Observación de Daniel:** "hay otro error, en la división de celdas, cuando un proceso va de la columna 1 a la columna 2, se colocó el nodo una fila verticalmente más abajo, cuando deberían ser paralelas".

**Iteración:** Claude interpretó que todos los nodos directamente conectados van en la misma fila. Daniel corrigió: "las filas deberían ser espacios de memoria" — la fila solo avanza cuando la columna del siguiente nodo es igual o menor a la columna actual.

**Regla aprobada:**
```
→ misma fila: si col_actual > col_anterior Y celda libre
↓ baja fila:  si col_actual <= col_anterior O celda ocupada
```

**Reglas adicionales aprobadas:**
- INICIO siempre ocupa fila propia — el siguiente nodo SIEMPRE baja (↓)
- FIN siempre ocupa fila propia y sola

**Salidas del rombo:**
- Sí → vértice inferior (↓), continúa flujo principal
- No → vértice lateral derecho o izquierdo, según hacia dónde retroceda el flujo

---

## [2026-03-17] Sesión 8 — Prueba con documento real: Ventilación No Invasiva

**Documento:** PR Ventilación No Invasiva UCI Adultos 
**Responsables detectados:** 6 únicos
1. Médico(a)
2. Enfermero(a)
3. Auxiliar de Enfermería
4. Terapeuta Respiratorio
5. Auxiliar de Laboratorio Clínico
6. Camillero

**Error cometido por Claude:** Se agruparon Auxiliar de Laboratorio Clínico y Camillero en una sola columna "Aux. Lab. / Camillero" para no superar 5 columnas.

**Por qué fue un error — observación de Daniel:**
- Viola R5: cada descripción única de "Persona Responsable" genera su propia columna
- Ignora la tolerancia ya definida: límite_base=5, tolerancia=6. 6 columnas es perfectamente válido
- La tolerancia existe exactamente para estos casos — no había ninguna necesidad de fusionar

**Aprendizaje registrado:** Antes de fusionar cualquier responsable, verificar si el total cabe dentro de la tolerancia (≤6). Solo si supera 6 se aplica distribución en bloques. Nunca fusionar para evitar superar el límite base.

**Distribución final correcta del documento:**
| Nodo | Col | Fila | Razón |
|------|-----|------|-------|
| INICIO | 1 | 1 | fila propia obligatoria |
| Act1 Médico | 1 | 2 | ↓ siempre baja después de INICIO |
| Act2 Enfermero | 2 | 2 | → col aumenta, libre |
| Act3 Aux.Enf | 3 | 2 | → col aumenta, libre |
| Act4 Terapeuta | 4 | 2 | → col aumenta, libre |
| Act5 Médico | 1 | 3 | ↓ col retrocede |
| Act6 Enfermero | 2 | 3 | → col aumenta, libre |
| Act7 Aux.Enf | 3 | 3 | → col aumenta, libre |
| Act8 Aux.Lab | 5 | 3 | → col aumenta, libre |
| Act9 Camillero | 6 | 4 | ↓ col retrocede desde 5 |
| FIN | 1 | 5 | ↓ fila propia obligatoria |

**XML generado:** ventilacion_vni_6cols.xml ✅

---

## [2026-03-18] Sesión 9 — Corrección de fórmula de paginación

**Error detectado:** La fórmula anterior calculaba páginas con ceil(total/límite_base)
y redistribuía entre ese mismo número de páginas, sin considerar que el sobrante
pequeño permite reducir el número de páginas usando la tolerancia.

**Ejemplo del error:** 19 filas → ceil(19/9)=3 páginas → sobrante=1 → redistribuía
en 3 páginas (7+6+6) en lugar de reducir a 2 páginas (10+9).

**Corrección:** Agregar PASO 1 con páginas_min = ceil(total/límite_tolerancia).
Cuando sobrante ≤ umbral, usar páginas_min como número final de páginas.
La misma lógica aplica simétricamente para columnas en bloques.

## [2026-03-18] Sesión 10 — Reducción de verbosidad y protocolo de lectura obligatorio

**Problema detectado por Daniel:** El formato de salida era excesivamente extenso,
consumiendo límite de chat con secciones innecesarias para el usuario final.

**Secciones eliminadas del formato de salida:**
- Algoritmo Perfecto — no aporta valor al usuario, solo consume tokens
- Grid de nodos — era información interna de construcción, no un entregable
- Nodos de referencia requeridos — información interna, no un entregable

**Cambios al formato de salida aprobados:**
1. Matriz de Responsables — descripción del paso debe ser resumida (no literal extensa)
2. Confirmaciones interactivas — una por turno, en este orden:
   a. ¿Las columnas detectadas son correctas?
   b. ¿El número de actividades es correcto?
   c. ¿La distribución de páginas es correcta? (SOLO si páginas > 2)
3. XML draw.io — completo dentro de bloque de código
4. Optimizaciones Opcionales — máximo 3

**Problema adicional detectado:** Claude repitió errores ya registrados en bitácora
porque no existe un protocolo formal de lectura de archivos del proyecto antes
de procesar el documento del usuario.

**Protocolo de ejecución obligatorio aprobado:**
Antes de leer cualquier documento del usuario, leer en este orden:
1. FlujoMaster_Bitacora.md
2. INSTRUCCIONES_PARA_CLAUDE.md
3. FlujoMaster_EstiloVisual.md
Solo después de completar estos 3 pasos, procesar el documento del usuario.

**Aprendizaje:** Los errores que ya están en bitácora son inadmisibles. La bitácora
solo tiene valor si se lee antes de ejecutar — no después de cometer el error.