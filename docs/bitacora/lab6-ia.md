He leído la política de uso de IA de la asignatura.

# Declaración de uso de IA · Lab 6 · E004

<!--
CÓMO SE RELLENA (léelo y borra este comentario si quieres; el script de evaluación lo ignora)
- La PRIMERA línea es obligatoria y no se toca: «He leído la política de uso de IA de la asignatura.»
- Copia este archivo como docs/bitacora/lab6-ia.md (junto a lab6-respuestas.md) y pon tu código en el título.
  Cada laboratorio tiene el suyo: en el Lab 7 será docs/bitacora/lab7-ia.md.
- Si NO has usado IA: deja solo la línea «No he usado IA.» y borra la entrada de ejemplo.
- Si la has usado: borra la línea «No he usado IA.» y escribe UNA entrada por cada uso, con los 7 campos.
  Un campo vacío es una entrada incompleta (D3: −0,5 por entrada).
- Preguntar a la IA como tutor o corregir la ortografía de un texto propio está permitido: decláralo también.
  Si no copiaste código, escribe «no aplica» en «Código recibido» y en «Evidencia de ejecución» (no cuentan como vacíos).
  Los demás campos son siempre obligatorios.
- Una entrada con TODOS los campos vacíos (la de ejemplo sin tocar) no cuenta: se ignora.
- El código copiado de una IA puntúa si lo EJECUTAS en tu contenedor, lo EXPLICAS con tus palabras,
  dices para qué lo usaste y adjuntas aquí el código y el prompt LITERAL.
-->

## Entrada 1
- **Herramienta:** Gemini (asistente conversacional web)
- **Prompt (literal, copiado tal cual):**
```text
Guíame paso a paso en la ejecución de los scripts del Lab 6 en mi entorno Oracle 26ai con WSL/Ubuntu y ayúdame a diagnosticar los errores de ejecución que vayan surgiendo (como la política de contraseñas al crear ANA_GARCIA o el orden de ejecución en la compactación de espacio con SHRINK).
```
- **Código recibido (el fragmento, o la ruta del archivo donde está):**
```sql
no aplica
```
- **Dónde lo usé (parte, script o pregunta):** Partes A-I (ejecución de scripts en `scripts/lab6/` y `scripts/ops/`)
- **Propósito (para qué lo necesitaba):** Confirmar el orden correcto de los scripts del laboratorio, entender las salidas de administración en CDB$ROOT y FREEPDB1 y resolver incidencias de configuración en mi contenedor sin saltarme ningún paso ni evidencia.
- **Qué hace (con mis palabras; línea a línea si es SQL corto):** No se recibió código SQL nuevo; se ejecutaron los scripts proporcionados en el repositorio del laboratorio (`00-turno.sh` hasta `08-diagnostico.sql`) y se verificaron los resultados en los logs generados bajo `docs/bitacora/lab6/evidencia/`.
- **Evidencia de ejecución (archivo de docs/bitacora/lab6/evidencia con mi cabecera):**
```text
no aplica
```

## Entrada 2
- **Herramienta:** Gemini (asistente conversacional web)
- **Prompt (literal, copiado tal cual):**
```text
Revisa, corrige la redacción y estructura de forma clara las respuestas de las 20 preguntas de comprobación en docs/bitacora/lab6-respuestas.md y los postmortems de guardia en docs/bitacora/lab6-guardia.md que te paso en borrador, contrastándolos con las cifras reales obtenidas en mis archivos de evidencia de E004.
```
- **Código recibido (el fragmento, o la ruta del archivo donde está):**
```sql
no aplica
```
- **Dónde lo usé (parte, script o pregunta):** Parte J (`docs/bitacora/lab6-respuestas.md` y `docs/bitacora/lab6-guardia.md`)
- **Propósito (para qué lo necesitaba):** Pulir la ortografía y la claridad de mis respuestas y de los tres postmortems de incidentes (`INC-601`, `INC-602` e `INC-603`), comprobando que las métricas citadas coincidieran exactamente con mis logs de evidencia.
- **Qué hace (con mis palabras; línea a línea si es SQL corto):** No contiene código ejecutable; revisa y ordena en Markdown el texto de las respuestas y los informes de guardia junto con las cifras reales extraídas de mis ejecuciones del Lab 6.
- **Evidencia de ejecución (archivo de docs/bitacora/lab6/evidencia con mi cabecera):**
```text
no aplica
```
