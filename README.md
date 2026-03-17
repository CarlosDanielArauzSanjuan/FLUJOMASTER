# ⬡ FlujoMaster v2.0

> Generador automático de diagramas de flujo swimlane en formato XML draw.io, impulsado por Claude AI.

**Autor:** Daniel Arauz Sanjuan ·Foscal Floridablanca, Santander, CO  
**Versión:** 2.0 · Actualizada 2026-03-17  
**Dominio:** Procedimientos institucionales Hospitalarios

---

## ¿Qué es FlujoMaster?

FlujoMaster es una herramienta que convierte procedimientos escritos en texto plano (actividades, responsables, numerales) en diagramas de flujo swimlane listos para importar en [draw.io](https://app.diagrams.net). Utiliza la API de Claude como motor de inteligencia para aplicar un sistema de reglas lógicas, algoritmo de posicionamiento en grid y escala visual en cascada.

---

## Archivos del proyecto

| Archivo | Descripción |
|---|---|
| index.html` | Aplicación web completa (autocontenida) |
| `INSTRUCCIONES_PARA_CLAUDE.md` | Reglas lógicas R1–R16, algoritmo de posicionamiento, paginación y formato de salida |
| `FlujoMaster_EstiloVisual.md` | Estilos gráficos XML, dimensiones de página y escala en cascada |
| `FlujoMaster_Bitacora.md` | Registro cronológico de decisiones, errores y aprendizajes (sesiones 1–8) |
| `README.md` | Este archivo |

---

## Uso rápido

1. Abre `index.html` en cualquier navegador moderno.
2. Pega el texto del procedimiento en el área de entrada.
3. Haz clic en **⚡ Generar FlujoMaster**.
4. Revisa las 4 pestañas de resultado:
   - **📊 Matriz** — tabla de responsables y tipos de nodo
   - **📄 XML draw.io** — código XML listo para copiar e importar
   - **💡 Optimizaciones** — sugerencias de mejora al proceso
   - **📥 Importar** — instrucciones paso a paso para draw.io
5. Copia el XML y en draw.io: `Extras → Editar Diagrama → Pegar XML → Aceptar`.

---

## Cómo funciona

FlujoMaster inyecta el sistema completo de reglas como prompt de sistema en cada llamada a la API de Claude. El modelo ejecuta el siguiente pipeline:

```
Texto del procedimiento
        ↓
  Identificación de responsables (R4, R5)
        ↓
  Asignación de columnas swimlane
        ↓
  Algoritmo de posicionamiento en grid (filas × columnas)
        ↓
  Lógica de paginación y distribución
        ↓
  Escala visual en cascada
        ↓
  Generación de XML draw.io
```

---

## Jerarquía de reglas

El sistema tiene 5 niveles de prioridad. Ningún nivel inferior puede invalidar uno superior.

```
1. Reglas lógicas R1–R16          ← máxima prioridad
2. Algoritmo de posicionamiento
3. Lógica de paginación y distribución
4. Escala visual en cascada
5. Estilo gráfico                 ← mínima prioridad
```

---

## Reglas lógicas principales

| Regla | Descripción |
|---|---|
| R1 | No inferir nada que no esté declarado explícitamente |
| R2 | El texto de cada nodo es el título exacto del documento |
| R3 | Flujo siempre lineal y secuencial |
| R4 | Solo personas o roles humanos generan columna swimlane |
| R5 | Cada responsable único genera su propia columna — nunca fusionar |
| R8 | El rombo de decisión va al final del ciclo operativo |
| R9 | Toda decisión produce exactamente dos ramas Sí/No |
| R13 | Sí = vértice inferior del rombo (↓) · No = vértice lateral |
| R16 | Si la descripción contiene ¿...? → genera rectángulo + rombo |

---

## Límites y tolerancias

| Dimensión | Límite base | Tolerancia |
|---|---|---|
| Columnas por bloque | 5 | 6 |
| Filas por página | 9 | 10 |

La tolerancia evita abrir un nuevo bloque o página por un solo elemento sobrante. El sistema estira antes de paginar.

---

## Dimensiones de página

Carta landscape a 96 dpi — compatible con draw.io por defecto.

```
PAGE_W = 1056px   PAGE_H = 816px
MARGIN = 20px     HEADER_H = 50px    HLINE = 8px
USABLE_W = 1016px USABLE_H = 718px
Y base del grid = 78px
```

---

## Tipos de nodo

| Tipo | Forma | Color de borde |
|---|---|---|
| Acción | Rectángulo redondeado | Azul `#3333CC` |
| Decisión | Rombo | Naranja `#E8860A` |
| INICIO / FIN | Elipse | Verde `#00AA00` |
| Referencia A/B/C | Elipse pequeña (1/4 celda) | Negro `#000000` |
| Paginación | Pentágono casa invertida (1/4 celda) | Negro `#000000` |

---

## Requisitos

- Navegador moderno con soporte para `fetch` y `navigator.clipboard`
- Acceso a Claude.ai (la llamada a la API usa la sesión activa — sin clave adicional)
- [draw.io](https://app.diagrams.net) para visualizar e imprimir el diagrama generado

---

## Historial de versiones

| Versión | Fecha | Cambios principales |
|---|---|---|
| 1.0 | 2026-03-13 | Reglas R1–R11, primer XML generado |
| 1.5 | 2026-03-13 | R16, corrección de medidas, sistema de grid |
| 2.0 | 2026-03-17 | Paginación por filas lógicas, columnas activas dinámicas, escala en cascada, tolerancias, app HTML completa |

---

## Notas de diseño

- El XML **no usa swimlanes nativos** de draw.io. Se construye con elementos independientes y coordenadas absolutas, lo que garantiza compatibilidad total y control preciso del layout.
- Todas las medidas se derivan en cascada desde las dimensiones de página. Nunca se usan valores fijos independientes.
- Los nodos de referencia A/B/C y los pentágonos de paginación **nunca ocupan una celda completa solos** — siempre comparten celda en distribución horizontal 1/4 + 3/4.

---

*FlujoMaster es un proyecto interno de mejora de procesos. Desarrollado con Claude (Anthropic).*