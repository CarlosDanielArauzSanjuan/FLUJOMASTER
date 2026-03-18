# FlujoMaster — Estilo Visual Obligatorio para XML draw.io

> Este archivo es la **única fuente de verdad visual** del proyecto. Todo lo que define la apariencia, dimensiones y posicionamiento del diagrama está aquí. Las INSTRUCCIONES_PARA_CLAUDE.md referencian este archivo — no duplican su contenido.

---

## 1. Dimensiones de Página

Draw.io carta landscape a 96dpi:

```
PAGE_W   = 1056px   (11 pulgadas × 96dpi)
PAGE_H   = 816px    (8.5 pulgadas × 96dpi)
MARGIN   = 20px     (implícito en draw.io)
HEADER_H = 50px     (fila de encabezados de columna)
HLINE    = 8px      (línea horizontal divisora)
USABLE_W = 1016px   (PAGE_W - MARGIN×2)
USABLE_H = 718px    (PAGE_H - MARGIN×2 - HEADER_H - HLINE)
```

---

## 2. Grid — Sistema de Coordenadas

El diagrama es una **matriz de celdas (fila × columna)**. Cada nodo ocupa exactamente una celda. Las dimensiones se calculan en cascada:

```
ancho_celda  = floor(USABLE_W / cols_activas_página)
alto_celda   = floor(USABLE_H / filas_página)
ancho_nodo   = floor(ancho_celda × 0.85)
alto_nodo    = floor(alto_celda  × 0.55)
font_size    = max(6, min(10, floor(alto_nodo × 0.25)))
```

**Coordenada X de nodo en columna c (0-based):**
```
x = MARGIN + c × ancho_celda + (ancho_celda - ancho_nodo) / 2
```

**Coordenada Y de nodo en fila r (0-based):**
```
y = MARGIN + HEADER_H + HLINE + r × alto_celda + (alto_celda - alto_nodo) / 2
```

**Separación entre nodos:**
```
separacion = alto_celda - alto_nodo
```

**Tabla de referencia rápida — Columnas:**
| Cols activas | Ancho celda | Ancho nodo |
|-------------|-------------|------------|
| 3 | 338px | 287px |
| 4 | 254px | 215px |
| 5 | 203px | 172px |
| 6 | 169px | 143px |

**Tabla de referencia rápida — Filas:**
| Filas página | Alto celda | Alto nodo | Separación |
|-------------|------------|-----------|------------|
| 8 | 89px | 48px | 41px |
| 9 | 79px | 43px | 36px |
| 10 | 71px | 39px | 32px |

---

## 3. Paginación por Filas

```
límite_base        = 9 filas
límite_tolerancia  = 10 filas
umbral_redistribución: sobrante ≤ 3

PASO 1: páginas_min  = ceil(total_filas / límite_tolerancia)
PASO 2: páginas_base = ceil(total_filas / límite_base)
PASO 3: sobrante = total_filas - ((páginas_base - 1) × límite_base)

    si sobrante ≤ umbral_redistribución:
        páginas = páginas_min
        filas_por_página = ceil(total_filas / páginas)
    sino:
        páginas = páginas_base
        → primeras (páginas-1) con límite_base filas
        → última absorbe sobrante (máx límite_tolerancia)
```

| Total filas | páginas_min | páginas_base | Sobrante | Resultado |
|-------------|-------------|--------------|----------|-----------|
| 10          | 1           | 2            | —        | 1 × 10    |
| 11          | 2           | 2            | 11       | 6 + 5     |
| 15          | 2           | 2            | 6        | 9 + 6     |
| 19          | 2           | 3            | 1 ≤ 3    | 10 + 9    |
| 24          | 3           | 3            | 6        | 9 + 9 + 6 |
| 28          | 3           | 4            | 1 ≤ 3    | 10 + 9 + 9|

---

## 4. Distribución de Columnas en Bloques

```
límite_base        = 5 cols
límite_tolerancia  = 6 cols
umbral_redistribución: sobrante ≤ 2

PASO 1: bloques_min  = ceil(total_cols / límite_tolerancia)
PASO 2: bloques_base = ceil(total_cols / límite_base)
PASO 3: sobrante = total_cols - ((bloques_base - 1) × límite_base)

    si sobrante ≤ umbral_redistribución:
        bloques = bloques_min
        cols_por_bloque = ceil(total_cols / bloques)
    sino:
        bloques = bloques_base
        → primeros (bloques-1) con límite_base cols
        → último absorbe sobrante (máx límite_tolerancia)
```

Tabla de referencia:
| Total cols | Bloques | Resultado   |
|------------|---------|-------------|
| 6          | 1       | 1 × 6       |
| 7          | 2       | 4 + 3       |
| 8          | 2       | 4 + 4       |
| 10         | 2       | 5 + 5       |
| 11         | 2       | 6 + 5       |
| 12         | 2       | 6 + 6       |
| 13         | 3       | 5 + 4 + 4   |

---

## 5. Columnas Activas por Página

Por cada página, solo se dibujan las columnas que tienen al menos 1 nodo en esa página. Las columnas sin nodos no se renderizan y su espacio se redistribuye.

```
cols_activas = columnas con ≥ 1 nodo en esa página
ancho_celda  = floor(USABLE_W / len(cols_activas))
```

---

## 6. Algoritmo de Posicionamiento de Nodos

Los nodos solo se mueven → (derecha) o ↓ (abajo). Nunca izquierda ni arriba.

```
para cada nodo en secuencia:
    col = columna del responsable

    si col > col_anterior Y celda(fila_actual, col) está libre:
        → misma fila
    sino:
        ↓ buscar primera fila >= fila_actual donde celda(fila, col) esté libre
```

Reglas especiales:
- INICIO → siempre fila propia. El siguiente nodo SIEMPRE baja (↓).
- FIN → siempre fila propia y exclusiva. Ningún otro nodo comparte su fila.

---

## 7. Reglas de Celdas — Mini-grid Interno (4 unidades horizontales)

```
Celda completa (4/4):
├── Rectángulo de acción     → siempre 4/4, nunca comparte
├── Rombo de decisión        → siempre 4/4, NUNCA comparte celda
└── Elipse INICIO / FIN      → siempre 4/4, nunca comparte

Elemento parcial (1/4 horizontal):
├── Pentágono paginación     → 1/4 + nodo adyacente 3/4 horizontal
└── Nodo referencia A/B/C   → 1/4 + nodo adyacente 3/4 horizontal

Elemento flotante (sin fila propia):
└── Etiqueta Sí / No         → flota junto al rombo, nunca ocupa fila propia
```

Reglas de acompañamiento:
- Pentágono NUNCA solo, salvo filas_página ≤ 7
- Nodo referencia A/B/C NUNCA solo, sin excepción
- Rombo NUNCA comparte celda
- Etiqueta Sí/No NUNCA ocupa fila propia — siempre flotante adyacente al rombo
- Pentágono siempre en fila 1 o última fila de la página

**Fórmulas de escala en cascada (todo deriva de dimensiones de página):**
```
ancho_celda           = floor(USABLE_W / cols_activas)
alto_celda            = floor(USABLE_H / filas_página)
ancho_nodo            = floor(ancho_celda × 0.85)
alto_nodo             = floor(alto_celda  × 0.55)
separacion            = alto_celda - alto_nodo
font_size             = max(6, min(10, floor(alto_nodo × 0.25)))
ancho_parcial         = floor(ancho_celda × 0.25)
ancho_nodo_compartido = floor(ancho_celda × 0.75 × 0.85)
diametro_ref          = floor(ancho_celda × 0.25 × 0.7)
```

Límites de contenido:
- Figura nunca desborda la celda
- Texto nunca desborda la figura
- Si el texto no cabe, reducir font_size hasta mínimo 6px
- `whiteSpace=wrap` + `overflow=hidden` obligatorio en todos los nodos

---

## 8. Salidas del Rombo de Decisión

```
Sí → vértice inferior  (↓) — continúa flujo principal hacia abajo
No → vértice lateral   (→ o ←) — según dirección del retroceso
```

Las etiquetas Sí / No son nodos text independientes. Nunca embebidas en el conector.

---

## 9. Estilos de Nodos

### Encabezados de Columna
```
style="text;html=1;strokeColor=none;fillColor=none;align=center;verticalAlign=middle;
       fontStyle=1;fontSize=10;fontFamily=Verdana;fontColor=#000000;"
height=50px
```

### Nodos de Acción (rectángulos)
```
style="rounded=1;whiteSpace=wrap;html=1;fillColor=none;strokeColor=#3333CC;
       strokeWidth=2;fontColor=#000000;fontFamily=Verdana;fontSize=FONT_SIZE;overflow=hidden;"
```

### Nodos de Decisión (rombos)
```
style="rhombus;whiteSpace=wrap;html=1;fillColor=none;strokeColor=#E8860A;
       strokeWidth=2;fontColor=#000000;fontFamily=Verdana;fontSize=FONT_SIZE;overflow=hidden;"
```

### Etiquetas Sí / No
```
style="text;html=1;strokeColor=none;fillColor=none;align=center;verticalAlign=middle;
       fontStyle=1;fontSize=FONT_SIZE;fontColor=#000000;fontFamily=Verdana;"
```

### Nodos INICIO / FIN (elipses)
```
style="ellipse;whiteSpace=wrap;html=1;fillColor=none;strokeColor=#00AA00;
       strokeWidth=2;fontStyle=1;fontColor=#000000;fontFamily=Verdana;fontSize=FONT_SIZE;"
```

### Pentágono de Paginación (casa invertida — vértice abajo)
```
style="shape=mxgraph.flowchart.start2;fillColor=none;strokeColor=#000000;
       strokeWidth=1.5;fontColor=#000000;fontFamily=Verdana;fontSize=FONT_SIZE;direction=south;"
```

### Nodo Referencia A / B / C
```
style="ellipse;whiteSpace=wrap;html=1;fillColor=none;strokeColor=#000000;
       strokeWidth=1.5;fontStyle=1;fontColor=#000000;fontFamily=Verdana;fontSize=FONT_SIZE;"
ancho = alto = diametro_ref   (ver fórmula en sección 7)
```

### Flechas / Conectores
```
style="edgeStyle=orthogonalEdgeStyle;html=1;strokeColor=#000000;strokeWidth=1.5;
       fillColor=#000000;exitX=EX;exitY=EY;exitDx=0;exitDy=0;
       entryX=NX;entryY=NY;entryDx=0;entryDy=0;"
```

Puntos de conexión según movimiento:
```
→ horizontal: exitX=1  exitY=0.5 | entryX=0   entryY=0.5
↓ vertical:   exitX=0.5 exitY=1  | entryX=0.5 entryY=0
Sí (rombo↓):  exitX=0.5 exitY=1  | entryX=0.5 entryY=0
No (lateral): exitX=1  exitY=0.5 | entryX según destino
```

---

## 10. Estructura Base XML Obligatoria

```xml
<mxGraphModel>
  <root>
    <mxCell id="0"/>
    <mxCell id="1" parent="0"/>

    <!-- Borde exterior -->
    <mxCell id="border" value=""
      style="rounded=0;whiteSpace=wrap;html=1;fillColor=none;strokeColor=#000000;strokeWidth=2;pointerEvents=0;"
      vertex="1" parent="1">
      <mxGeometry x="20" y="20" width="TOTAL_W" height="TOTAL_H" as="geometry"/>
    </mxCell>

    <!-- Encabezados (uno por columna activa, x += ancho_celda) -->
    <mxCell id="hdr0" value="RESPONSABLE 1"
      style="text;html=1;strokeColor=none;fillColor=none;align=center;verticalAlign=middle;
             fontStyle=1;fontSize=10;fontFamily=Verdana;fontColor=#000000;"
      vertex="1" parent="1">
      <mxGeometry x="20" y="20" width="ANCHO_CELDA" height="50" as="geometry"/>
    </mxCell>

    <!-- Línea horizontal divisora -->
    <mxCell id="hline" value=""
      style="shape=line;strokeColor=#000000;strokeWidth=2;fillColor=none;horizontal=1;"
      vertex="1" parent="1">
      <mxGeometry x="20" y="70" width="TOTAL_W" height="8" as="geometry"/>
    </mxCell>

    <!-- Líneas verticales divisoras (x = 20 + i × ancho_celda) -->
    <mxCell id="vl1" value=""
      style="rounded=0;whiteSpace=wrap;html=1;fillColor=#000000;strokeColor=#000000;strokeWidth=1;pointerEvents=0;"
      vertex="1" parent="1">
      <mxGeometry x="20+ANCHO_CELDA" y="20" width="1" height="TOTAL_H" as="geometry"/>
    </mxCell>

    <!-- Nodos con coordenadas absolutas del grid -->
    <!-- Flechas con exitX/exitY/entryX/entryY según dirección -->

  </root>
</mxGraphModel>
```
