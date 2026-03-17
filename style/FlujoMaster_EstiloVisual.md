# FlujoMaster — Estilo Visual Obligatorio para XML draw.io
**Versión:** 2.0 · Actualizada 2026-03-17

> Este archivo es la fuente única de verdad para estilos gráficos. Las reglas de este archivo están subordinadas a las reglas lógicas R1-R16 y al algoritmo de posicionamiento definidos en `INSTRUCCIONES_PARA_CLAUDE.md`.

---

## Dimensiones de Página

```
Tipo: carta landscape a 96dpi
PAGE_W   = 1056px  (11 pulgadas × 96dpi)
PAGE_H   = 816px   (8.5 pulgadas × 96dpi)
MARGIN   = 20px    (implícito en draw.io)
HEADER_H = 50px
HLINE    = 8px
USABLE_W = 1016px
USABLE_H = 718px
```

---

## Escala en Cascada

Todas las medidas se derivan en cascada desde la página. Nunca usar valores fijos independientes.

```
ancho_celda      = floor(1016 / cols_activas_esa_página)
alto_celda       = floor(718  / filas_esa_página)
ancho_nodo       = floor(ancho_celda * 0.85)
alto_nodo        = floor(alto_celda  * 0.55)
font_size        = max(6, min(10, floor(alto_nodo * 0.25)))
ancho_parcial    = floor(ancho_celda * 0.25)      ← para pentágono y ref A/B/C
ancho_compartido = floor(ancho_celda * 0.75 * 0.85) ← nodo que acompaña al parcial
```

### Tabla de referencia rápida

| Cols activas | Ancho celda | Ancho nodo |
|-------------|-------------|------------|
| 3 | 338px | 287px |
| 4 | 254px | 215px |
| 5 | 203px | 172px |
| 6 | 169px | 143px |

| Filas/página | Alto celda | Alto nodo | Font |
|-------------|------------|-----------|------|
| 8 | 89px | 48px | 6px |
| 9 | 79px | 43px | 6px |
| 10 | 71px | 39px | 6px |

---

## Estructura General — Matriz con Divisores Manuales

- Encabezados de columna como celdas `text` independientes (no swimlanes nativos).
- Línea horizontal divisora (`shape=line`) entre encabezados y área del diagrama.
- Líneas verticales entre columnas: rectángulos de 1px, `fillColor=#000000`.
- Nodos con coordenadas absolutas respecto a `parent="1"`.
- No se usan swimlanes nativos (`shape=pool` / `shape=swimlane`).

---

## Nodos de Acción — Rectángulo

```xml
style="rounded=1;whiteSpace=wrap;html=1;overflow=hidden;fillColor=none;
       strokeColor=#3333CC;strokeWidth=2;fontColor=#000000;
       fontFamily=Verdana;fontSize=FONT_SIZE;"
```

- Borde azul oscuro `#3333CC`, grosor 2, esquinas redondeadas.
- Fondo transparente. Texto negro, Verdana, tamaño dinámico.
- El texto comienza con el número de paso: `1. Texto`, `2. Texto`.

---

## Nodos de Decisión — Rombo

```xml
style="rhombus;whiteSpace=wrap;html=1;overflow=hidden;fillColor=none;
       strokeColor=#E8860A;strokeWidth=2;fontColor=#000000;
       fontFamily=Verdana;fontSize=FONT_SIZE;"
```

- Borde naranja `#E8860A`, grosor 2.
- SIEMPRE solo en su celda. Nunca comparte.
- Las etiquetas Sí/No son nodos `text` independientes junto a la flecha.

---

## Etiquetas Sí / No

```xml
style="text;html=1;strokeColor=none;fillColor=none;align=center;
       verticalAlign=middle;fontSize=FONT_SIZE;fontColor=#000000;
       fontFamily=Verdana;fontStyle=1;"
```

- Nodos `text` independientes posicionados manualmente junto a la flecha de salida.
- Nunca embebidos dentro del conector.

---

## Nodos INICIO / FIN — Elipse

```xml
style="ellipse;whiteSpace=wrap;html=1;overflow=hidden;fillColor=none;
       strokeColor=#00AA00;strokeWidth=2;fontStyle=1;fontColor=#000000;
       fontFamily=Verdana;fontSize=FONT_SIZE;"
```

- Borde verde `#00AA00`, grosor 2.
- Texto en negrita. INICIO y FIN siempre en fila propia.

---

## Nodo de Referencia A / B / C

```xml
style="ellipse;whiteSpace=wrap;html=1;fillColor=none;strokeColor=#000000;
       strokeWidth=1.5;fontStyle=1;fontColor=#000000;fontFamily=Verdana;
       fontSize=FONT_SIZE;"
```

- Elipse pequeña, borde negro.
- Ancho = `ancho_parcial` (1/4 de celda). Alto = `alto_nodo`.
- NUNCA solo en celda — siempre acompañado horizontalmente (1/4 + 3/4).

---

## Pentágono de Paginación — Casa Invertida

```xml
style="shape=mxgraph.flowchart.start2;whiteSpace=wrap;html=1;
       fillColor=none;strokeColor=#000000;strokeWidth=1.5;
       fontColor=#000000;fontFamily=Verdana;fontSize=FONT_SIZE;
       direction=south;"
```

- Forma de casa invertida: vértice apuntando hacia **abajo**.
- Ancho = `ancho_parcial` (1/4 de celda).
- NUNCA solo salvo `filas_página ≤ 7`. Siempre en fila 1 o última fila.
- Número de página (1, 2, 3...) como texto centrado dentro.

---

## Flechas / Conectores

```xml
style="edgeStyle=orthogonalEdgeStyle;html=1;strokeColor=#000000;
       strokeWidth=1.5;fillColor=#000000;"
```

- Color negro, grosor 1.5, punta sólida, estilo ortogonal.
- **Salida horizontal (→):** `exitX=1;exitY=0.5;entryX=0;entryY=0.5`
- **Salida vertical (↓):** `exitX=0.5;exitY=1;entryX=0.5;entryY=0`
- Flechas que cruzan columnas usan puntos intermedios (`Array as="points"`) para trayectoria ortogonal limpia.

---

## Encabezados de Columna

```xml
style="text;html=1;strokeColor=none;fillColor=none;align=center;
       verticalAlign=middle;fontStyle=1;fontSize=10;fontFamily=Verdana;
       fontColor=#000000;"
```

- Texto en MAYÚSCULAS, negrita, centrado, Verdana 10pt.
- Alto fijo: `HEADER_H = 50px`.
- Uno por columna, posicionado en `y = MARGIN`.

---

## Estructura Base XML Obligatoria

```xml
<mxGraphModel>
  <root>
    <mxCell id="0"/>
    <mxCell id="1" parent="0"/>

    <!-- Borde exterior -->
    <mxCell id="border" value=""
      style="rounded=0;whiteSpace=wrap;html=1;fillColor=none;
             strokeColor=#000000;strokeWidth=2;pointerEvents=0;"
      vertex="1" parent="1">
      <mxGeometry x="20" y="20" width="TOTAL_W" height="TOTAL_H" as="geometry"/>
    </mxCell>

    <!-- Encabezados (uno por columna activa) -->
    <mxCell id="hdr0" value="RESPONSABLE 1"
      style="text;html=1;strokeColor=none;fillColor=none;align=center;
             verticalAlign=middle;fontStyle=1;fontSize=10;fontFamily=Verdana;
             fontColor=#000000;"
      vertex="1" parent="1">
      <mxGeometry x="20" y="20" width="ANCHO_COLUMNA" height="50" as="geometry"/>
    </mxCell>
    <!-- repetir para cada columna activa con x += ANCHO_COLUMNA -->

    <!-- Línea horizontal divisora -->
    <mxCell id="hline" value=""
      style="shape=line;strokeColor=#000000;strokeWidth=2;fillColor=none;horizontal=1;"
      vertex="1" parent="1">
      <mxGeometry x="20" y="70" width="TOTAL_W" height="8" as="geometry"/>
    </mxCell>

    <!-- Líneas verticales (una entre cada columna activa) -->
    <mxCell id="vl1" value=""
      style="rounded=0;whiteSpace=wrap;html=1;fillColor=#000000;
             strokeColor=#000000;strokeWidth=1;pointerEvents=0;"
      vertex="1" parent="1">
      <mxGeometry x="ANCHO_COL1_FIN" y="20" width="1" height="TOTAL_H" as="geometry"/>
    </mxCell>
    <!-- repetir con x += ANCHO_COLUMNA -->

    <!-- Nodos y flechas con coordenadas absolutas -->

  </root>
</mxGraphModel>
```

---

## Posicionamiento de Nodos

```
Y base del grid   = MARGIN + HEADER_H + HLINE = 20 + 50 + 8 = 78px
Centro X columna  = MARGIN + (col * ancho_celda) + (ancho_celda / 2)
X nodo            = MARGIN + (col * ancho_celda) + (ancho_celda - ancho_nodo) / 2
Y nodo            = 78 + (fila * alto_celda) + (alto_celda - alto_nodo) / 2
```

- `col` y `fila` son 0-based.
- Los nodos dentro de cada columna nunca salen de los límites de su celda.
- Las flechas entre nodos de distinta fila usan puntos intermedios para trayectoria ortogonal.
