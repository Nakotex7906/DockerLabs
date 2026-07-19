# Comandos ExifTool

Guía de referencia rápida con los comandos de `exiftool` más usados en retos de esteganografía/forense de archivos. Cada sección explica qué hace cada flag, cuándo conviene usarla y trae el comando listo para copiar (reemplaza `<archivo>` por la ruta del archivo a analizar, ej. `imagen.jpg`).

---

## 1. Lectura básica de metadatos

El uso más simple: ver todos los metadatos que trae un archivo (imagen, PDF, audio, video, documento de Office, etc). En CTF es lo primero que corres sobre cualquier archivo adjunto o descargado.

```bash
exiftool <archivo>
```

Sin flags, muestra campos como cámara/dispositivo, fecha de creación, software usado, coordenadas GPS, autor, comentarios — cualquiera de estos puede esconder la flag o una pista.

---

## 2. Flags de nivel de detalle

- `-a` → muestra **todos** los tags, incluyendo los duplicados (por defecto exiftool oculta tags repetidos entre distintos grupos).
  ```bash
  exiftool -a <archivo>
  ```
- `-G` → agrupa la salida mostrando a qué **grupo**/segmento pertenece cada tag (ej. `EXIF`, `IPTC`, `XMP`, `File`), útil para ubicar dónde está escondido un dato raro.
  ```bash
  exiftool -a -G <archivo>
  ```
- `-v` (o `-v1`, `-v2`, `-v3`) → modo verboso con distintos niveles de profundidad, muestra la estructura binaria interna del archivo (offsets, tamaños). Usar `-v3` cuando sospechas de datos ocultos a bajo nivel.
  ```bash
  exiftool -v3 <archivo>
  ```
- `-u` → incluye tags "unknown" (no reconocidos por exiftool), a veces ahí es donde el reto esconde algo custom.
  ```bash
  exiftool -a -G -u <archivo>
  ```
- `-b` → saca el valor de un tag en formato binario crudo (raw), útil cuando un campo de texto en realidad contiene bytes de otro tipo de archivo (ej. una imagen embebida dentro de un thumbnail EXIF).

---

## 3. Extraer un tag específico

Cuando ya sabes qué campo te interesa (por ejemplo, tras ver la salida completa), puedes pedir solo ese tag.

- `-<NombreDelTag>` → muestra solo ese tag puntual (ej. `-Comment`, `-GPS*`, `-Artist`).
  ```bash
  exiftool -Comment <archivo>
  ```
- `-GPS*` (con comodín `*`) → muestra todos los tags relacionados a geolocalización.
  ```bash
  exiftool -GPS* <archivo>
  ```
- `-s` → muestra los nombres de los tags en formato "corto" (nombre técnico en vez de descripción larga), útil para después usar ese mismo nombre al escribir con `-TAG=valor`.

---

## 4. Extraer miniaturas e imágenes embebidas

Muchos archivos (JPG, RAW de cámara, PDF) llevan una miniatura embebida en los metadatos, y a veces ahí se esconde algo distinto a la imagen principal.

- `-ThumbnailImage` → extrae el thumbnail embebido.
  ```bash
  exiftool -b -ThumbnailImage <archivo> > thumbnail.jpg
  ```
- `-PreviewImage` → extrae una vista previa embebida de mayor resolución (común en RAW/algunos JPG).
  ```bash
  exiftool -b -PreviewImage <archivo> > preview.jpg
  ```
- `-a -b -W %d%f_%t%-c.%s` → extrae **todas** las imágenes embebidas de un archivo, nombrando cada salida automáticamente según el tag de origen.
  ```bash
  exiftool -a -b -W %d%f_%t%-c.%s -ee -FileTypeExtension <archivo>
  ```

---

## 5. Escribir / modificar metadatos

En algunos retos no solo hay que leer, sino también ocultar o limpiar datos.

- `-TAG=valor` → escribe un valor en un tag específico.
  ```bash
  exiftool -Comment="mensaje secreto" <archivo>
  ```
- `-TAG=` (sin valor) → borra ese tag específico.
  ```bash
  exiftool -Comment= <archivo>
  ```
- `-all=` → borra **todos** los metadatos del archivo de una vez (útil para limpiar antes de subir un archivo a otro reto, o para comparar tamaño antes/después).
  ```bash
  exiftool -all= <archivo>
  ```
- `-overwrite_original` → aplica los cambios directamente sobre el archivo original, sin crear una copia de respaldo `_original` (que es el comportamiento por defecto).
  ```bash
  exiftool -all= -overwrite_original <archivo>
  ```

---

## 6. Procesamiento en lote (varios archivos/carpetas)

- `-r` → modo recursivo, procesa también subcarpetas.
  ```bash
  exiftool -r -a -G /ruta/carpeta/
  ```
- Pasar varios archivos o un directorio directamente también funciona sin flags extra:
  ```bash
  exiftool *.jpg
  ```
- `-csv` → exporta los metadatos de todos los archivos procesados en formato CSV, útil para comparar muchos archivos a la vez.
  ```bash
  exiftool -csv -r /ruta/carpeta/ > metadatos.csv
  ```

---

## 7. Combo recomendado para DockerLabs

Flujo típico cuando te entregan un archivo (imagen, PDF, audio) como parte de un reto:

```bash
# Paso 1: vistazo completo, agrupado y con tags desconocidos, para no perderte nada raro
exiftool -a -G -u <archivo>

# Paso 2: si algo llamó la atención (ej. un thumbnail o preview), extráelo
exiftool -b -ThumbnailImage <archivo> > extraido.jpg

# Paso 3: si sospechas de datos binarios ocultos a bajo nivel, sube el nivel de detalle
exiftool -v3 <archivo>
```

---

### Notas rápidas
- ExifTool detecta el tipo de archivo por su contenido, no por la extensión — sirve para confirmar si un `.jpg` es en realidad otro formato (típico truco de esteganografía).
- Si `exiftool` no muestra nada llamativo, complementa con `file <archivo>`, `binwalk <archivo>` y `strings <archivo>` — cada herramienta mira el archivo desde un ángulo distinto.
- Después de extraer un archivo embebido, siempre revísalo también con `exiftool` — puede traer sus propios metadatos escondidos.