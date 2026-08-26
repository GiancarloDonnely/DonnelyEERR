# CLAUDE.md

Guía para Claude Code (o cualquier agente) al trabajar en este repositorio.

## Qué es este proyecto

Tablero financiero gerencial de **Donnely Ltda.** — un dashboard HTML de una sola página que
grafica el Estado de Resultados (EERR) 2026 en CLP, usando Chart.js. Los datos se leen **en
vivo desde una Google Sheet publicada como CSV**; no hay backend, build step ni framework: todo
es HTML + CSS + JS inline.

## Estructura del repo

- `index.html` — **único** dashboard vigente (~1737 líneas). Contiene HTML+CSS+JS embebido, la
  configuración de conexión al Sheet, un parser CSV propio, datos de respaldo y el renderizado
  con Chart.js.
- `Datos_Donnely_template.csv` — plantilla que refleja la estructura real y actual de la hoja
  "Estado de Resultados" publicada (fila de título, fila de meses, fila `Concepto`/`CLP`/`%`, y
  las filas de subtotal oficiales que lee el parser). Los valores son un snapshot real (no datos
  inventados) — al agregar meses nuevos al Sheet, conviene refrescar este archivo también.
- `README.md` — placeholder (`# DonnelyEERR`), sin contenido adicional todavía.
- `archive/Dashboard_Donnely_2026 (2).html` — versión anterior del dashboard, **archivada y sin
  uso**. Usaba un parser CSV pensado para un formato simple (Concepto + un valor por mes) que ya
  no coincide con la hoja real "Estado de Resultados" (columnas alternadas CLP/%, ítems
  detallados). Apuntaba a la misma URL de Sheet que `index.html`, por lo que en la práctica caía
  silenciosamente a sus datos de respaldo desactualizados. Se mantiene solo como referencia
  histórica — no editar ni servir este archivo.

## Cómo fluyen los datos (importante antes de tocar código)

1. En Google Sheets: **Archivo → Compartir → Publicar en la web** → se elige la hoja "Estado de
   Resultados" → formato **CSV** → **Publicar**.
2. La URL publicada (termina en `output=csv`) se pega en la constante `SHEET_CSV_URL` dentro de
   `index.html` (~línea 1586).
3. Al cargar la página, el script hace `fetch(SHEET_CSV_URL, {cache:'no-store'})`, parsea el CSV
   con `parseSheetCSV()` (parser propio, ~línea 1682) y renderiza los gráficos.
4. Si el fetch falla, se usan datos de respaldo embebidos en el código (~línea 615) y se muestra
   el aviso "⚠ Usando datos de respaldo" en el elemento `data_status` (~línea 219).
5. **El formato real de la hoja NO es "Concepto + un valor por mes"**: cada mes ocupa **dos
   columnas** (`CLP` y `%`), y el `Concepto` está en la columna 1. `detectMonthColumns()`
   detecta dinámicamente en qué columnas están los meses buscando los nombres de mes en las
   primeras 15 filas; `labelToKey()` mapea el texto exacto de las filas de subtotal oficiales del
   EERR (`VENTAS NETAS`, `COSTO DE VENTAS TOTAL`, `TOTAL GASTOS DE ADMINISTRACIÓN`, `TOTAL
   GASTOS DE VENTAS`, `(-) Gastos financieros...`, `(+/-) Diferencias de cambio...`, `(-)
   Depreciación...`, `(-) Impuesto a la Renta... PPM...`) a las claves internas (`ventas`, `cogs`,
   `adm`, `vta`, `fin`, `dif`, `dep`, `imp`). Solo se usan esos 8 subtotales; las decenas de
   líneas de detalle (sueldos, arriendos, materias primas, etc.) se ignoran. Ver
   `Datos_Donnely_template.csv` como referencia exacta de la estructura real.
6. El dashboard "se extiende solo al agregar meses" (según el propio comentario del código): para
   agregar un mes nuevo basta con agregar el par de columnas (mes/CLP/%) en el Sheet, no hay que
   tocar el código — `detectMonthColumns()` lo detecta automáticamente.

## Convenciones de trabajo

- Un solo archivo HTML por dashboard (HTML+CSS+JS inline), sin build step ni package manager: se
  abre directamente en el navegador.
- Librería externa: Chart.js vía CDN.
- Idioma de la interfaz: español. Moneda: CLP (montos netos).
- Al editar `index.html`, mantener el parser CSV (`splitCSVLine`, `parseSheetCSV`) tolerante al
  mismo formato que exporta Google Sheets (comillas, comas dentro de celdas, etc.) — un cambio
  de formato ahí puede romper silenciosamente la lectura en vivo.
- Si se modifica la lista de conceptos (filas) o el rango de meses, actualizar también
  `Datos_Donnely_template.csv` para que siga sirviendo como plantilla real.

## Antes de tocar el repo / conectar con GitHub

- Esta copia local **no tiene `.git` inicializado** (es una carpeta descargada como ZIP). Antes
  de asumir que hay historial previo o un remoto configurado, correr `git init`, agregar el
  remoto (`git remote add origin ...`) y hacer el primer commit si corresponde.
- En este entorno no hay `gh` (GitHub CLI) instalado — si una tarea lo necesita, verificar
  primero con `which gh` e instalarlo, o usar un token (`GITHUB_TOKEN`) como respaldo.

## Pendientes / puntos a decidir con el usuario

- `README.md` está prácticamente vacío; conviene documentar ahí el propósito del repo, cómo
  publicar el Sheet, y el link al dashboard publicado (por ejemplo si se sube a GitHub Pages).

## Skill relacionado

Este proyecto es un caso de uso natural para el skill **`gsheet-github-sync`**: lee datos en
vivo desde un Google Sheet y los sincroniza con archivos de un repo de GitHub (commit + push), o
genera/actualiza Issues de GitHub a partir de filas de una planilla. Útil, por ejemplo, para
automatizar la actualización del CSV de respaldo o de la data embebida en `index.html` cada vez
que cambie el Sheet, una vez que este repo esté conectado a GitHub.
