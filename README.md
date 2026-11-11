# Project Manager AI  ?  (generado)

App Android en **Java clasico** (sin AndroidX, sin dependencias externas) que
convierte **Qwen3-Max** en tu equipo de desarrollo: tu describes el proyecto,
la app genera el **prompt perfecto**, Qwen responde con el **Protocolo PM** y
la app **escribe / edita / borra los archivos** por ti.

Pensada para compilarse **dentro del propio telefono con AIDE 3.2.2**.

---

## 1. Compilar en AIDE 3.2.2

1. Copia la carpeta `ProjectManagerAI` completa al almacenamiento del telefono.
2. AIDE ? **Project** ? **Open** ? selecciona la carpeta `ProjectManagerAI`.
   (AIDE reconoce `AndroidManifest.xml` + `src/` + `res/`.)
3. Ajustes de AIDE recomendados:
   - SDK: **Android 8.1 (API 27)** o **API 28**.
   - Java: **1.7**.
   - Activa "Auto-build" si quieres.
4. Pulsa **Run**. El icono y el nombre (`Project Manager AI`) salen automaticos
   desde `res/mipmap-*/ic_launcher.png`.

> No hay `dependencies {}`: todo usa el SDK de Android y `org.json` (incluido).
> Si AIDE se queja de `build.gradle`, borralo: no es necesario.

Ruta de trabajo de la app en el telefono:
`/Android/data/com.pm.ai/files/PM_Projects/<tu-proyecto>/`

---

## 2. Flujo de trabajo (el importante)

```
 [1] Crear proyecto  ->  carpeta + ficha tecnica (descripcion + caracteristicas)
 [2] Prompt Qwen     ->  la app redacta el mega-prompt con el PROTOCOLO PM
 [3] Copiar / Abrir  ->  lo pegas en chat.qwen.ai con el modelo Qwen3-Max
 [4] Qwen responde   ->  SOLO TEXTO con marcadores @@PM_*  (no crea archivos)
 [5] Aplicar         ->  pegas la respuesta y la app escribe TODO en el proyecto
 [6] Ejecutar        ->  HTML en WebView interno, Python/Node/C con Termux,
                         Android con AIDE
 [7] Iterar          ->  MEJORAR / CORREGIR ERROR / CONTINUAR
```

---

## 3. Protocolo PM (lo que Qwen debe devolver)

```
@@PM_START
@@PM_PROJECT: mi-app
@@PM_MODE: CREATE
@@PM_PLAN
1. index.html - estructura
2. css/styles.css - tema oscuro
@@PM_END_PLAN

@@PM_FILE
@@PM_ACTION: CREATE
@@PM_PATH: index.html
@@PM_LANG: html
@@PM_NOTE: pagina principal
@@PM_CODE_START
<!DOCTYPE html>
...contenido completo...
@@PM_CODE_END
@@PM_STATUS: COMPLETE
@@PM_END_FILE

@@PM_FILE
@@PM_ACTION: EDIT
@@PM_PATH: js/app.js
@@PM_EDIT_START
@@PM_FIND_START
const version = "1.0";
@@PM_FIND_END
@@PM_REPLACE_START
const version = "2.0";
@@PM_REPLACE_END
@@PM_EDIT_END
@@PM_STATUS: COMPLETE
@@PM_END_FILE

@@PM_SUMMARY
Que se ha hecho y como probarlo
@@PM_NEXT
Siguiente iteracion sugerida
@@PM_END
```

Acciones soportadas: `CREATE`, `REPLACE`, `EDIT` (con pares FIND/REPLACE),
`DELETE`, `RENAME` (usa `@@PM_TARGET`), `CONTINUE`, `APPEND`.

El parser es **tolerante**: acepta los marcadores dentro de comentarios
(`<!-- @@PM_PATH: ... -->`), bloques ``` envolventes y variantes de nombre.
Si la respuesta no trae marcadores, intenta reconstruir las acciones desde los
bloques de codigo markdown buscando la ruta en las lineas cercanas.

---

## 4. Cuando Qwen corta el codigo (CONTINUAR)

1. Pegas la respuesta en **Aplicar**. La app detecta `@@PM_STATUS: INCOMPLETE`
   o un bloque `@@PM_CODE_START` sin cerrar.
2. Guarda el trozo recibido en `.pm/pending.json` (no escribe el archivo roto).
3. Aparece el banner amarillo ? **Continuar**.
4. La app genera un prompt con las **ultimas 30 lineas ya escritas** y la orden
   de seguir **exactamente** donde se quedo, sin repetir nada.
5. Pegas la continuacion en **Aplicar**: se concatena al parcial y, cuando el
   modelo marca `COMPLETE`, se escribe el archivo definitivo y se limpia el
   parcial. Repite tantas veces como haga falta.

---

## 5. Estructura del codigo generado

```
AndroidManifest.xml        minSdk 21 / targetSdk 28 / sin permisos peligrosos
project.properties         target=android-28
build.gradle               opcional (Android Studio)
res/values/                colors, strings, styles
res/drawable/              shapes, ripples, selectores e iconos vectoriales
res/mipmap-*/              icono PNG (48/72/96/144/192) generado por el script
res/layout/                6 pantallas + 4 dialogs + items
src/com/pm/ai/
   MainActivity            lista de proyectos, stats, buscador, ajustes, import
   ProjectActivity         arbol de archivos, acciones, ejecutar, exportar ZIP
   EditorActivity          editor con gutter, resaltado, buscar/reemplazar, tools
   PromptActivity          generador de prompts (CREATE/UPDATE/FIX/CONTINUE)
   ApplyActivity           parser + lista de acciones + aplicacion con backup
   PreviewActivity         WebView con CONSOLA JS / visor de texto
   core/Storage            proyectos, arbol, lectura/escritura, ZIP
   core/Project            metadatos (.pm/project.json)
   core/Types              reglas tecnicas por tipo (html/python/android/...)
   core/ProjectTemplates   plantillas iniciales por tipo
   core/PromptBuilder      PROTOCOLO PM y construccion del prompt
   core/ResponseParser     parser tolerante de la respuesta del modelo
   core/Action             modelo de accion (CREATE/EDIT/DELETE/...)
   core/ActionApplier      ejecucion sobre disco + backup + ediciones fuzzy
   core/PendingManager     archivos a medias para CONTINUAR
   core/Runner             ejecutar segun tipo (WebView/Pydroid/Termux/AIDE)
   ui/CodeEditor           EditText con numeros de linea y auto-indentado
   ui/Syntax               resaltado por familia de lenguaje
   ui/*Adapter             ProjectAdapter, TreeAdapter, ActionAdapter
   util/Utils              IO, ZIP, portapapeles, compartir, fechas
```

Todo el proyecto vive en `.pm/` del propio proyecto:
`project.json`, `prompts/`, `responses/`, `pending.json`, `backups/<fecha>/`.

---

## 6. Regenerar / personalizar

```bash
python3 generar_project_manager_ai.py            # genera ./ProjectManagerAI
python3 generar_project_manager_ai.py --zip      # ademas crea el .zip
python3 generar_project_manager_ai.py --out DIR  # carpeta de destino
```

Para cambiar el nombre o el paquete, edita arriba del script:
`APP_NAME`, `PKG`, `VERSION`.
