# Lab 6 - Respuestas

Alumno: `E004` · Contenedor: `oralab-26ai` (DBID `1509902213`) · Evidencias en `docs/bitacora/lab6/evidencia/`.

## Partes A-C · Rol operativo, parámetros y estados

**1. ¿Cuántos errores ORA- tenía tu alert log en 24 h en la Parte B y a qué se debían? ¿Por qué ese número no es una alarma por sí solo?**
Había 19 errores `ORA-` en las últimas 24 horas (`*_00-turno.script.log`). La mayoría son fallos ya conocidos del propio contenedor, como `ORA-00800: Set Priority Failed` (40 incidentes acumulados) o avisos de `DBMS_QOPATCH` / `ORA-20013` porque no puede leer el inventario de parches. Ese número por sí solo no sirve como alarma porque mezcla errores de entorno o de pruebas con problemas reales; hay que mirar si aparece algún error nuevo o inesperado.

**2. ¿A qué hora empieza y cuánto dura la ventana de mantenimiento de un miércoles en tu base? ¿Qué tareas automáticas corren en ella?**
La `WEDNESDAY_WINDOW` empieza a las 22:00 y dura 4 horas (`*_01-automatizacion.spool.log`). Dentro de esa ventana se ejecutan las tres tareas automáticas habilitadas: `auto optimizer stats collection` (estadísticas del optimizador), `auto space advisor` (asesor de espacio) y `sql tuning advisor` (ajuste SQL).

**3. Explica con tu salida de la Parte C la diferencia entre lo que mostraban V$SPPARAMETER y V$PARAMETER tras el SCOPE = SPFILE. ¿Qué habría pasado en el siguiente reinicio sin el RESET?**
Tras hacer `ALTER SYSTEM SET db_files = 250 SCOPE = SPFILE`, `V$SPPARAMETER` mostraba `VALUE = 250` e `ISSPECIFIED = TRUE`, mientras que `V$PARAMETER` seguía con `200` (`*_02-parametros.script.log`). Como `db_files` es un parámetro estático (`ISSYS_MODIFIABLE = FALSE`), Oracle no lo aplica en caliente y solo lo deja apuntado en el SPFILE para el próximo arranque. Sin el `RESET`, al reiniciar la base habría arrancado con `db_files = 250` en vez de 200.

**4. ¿Por qué RESET fue la vuelta atrás correcta para db_files y no lo habría sido para processes? Cita la línea de tu PFILE que lo demuestra.**
Porque `db_files` no estaba configurado explícitamente en el SPFILE original (`ISSPECIFIED = FALSE`), así que `RESET` lo devuelve a su valor por defecto. En cambio, `processes` sí estaba fijado a mano en el SPFILE y un `RESET` habría borrado nuestro ajuste dejando el valor por defecto del motor. Se ve en `*_02-init_desde_spfile.ora`, donde aparece `*.processes=200` mientras que `db_files` ni siquiera está listado.

**5. Tras el reinicio sin estado guardado, ¿qué error obtuvo C##OPS_RO y por qué la instancia decía «Database opened»?**
Obtuvo `ORA-01109: database not open` (`*_02-estados-pdb.script.log`). La instancia decía «Database opened» porque el contenedor raíz (`CDB$ROOT`) sí se abrió, pero como no habíamos hecho `SAVE STATE` antes de reiniciar, `FREEPDB1` se quedó en estado `MOUNTED` y no dejaba entrar a ningún usuario.

## Partes D-E · Servicios, sesiones y diccionario

**6. ¿Qué SID, SPID y evento tenía la sesión de la aplicación en tu ejecución? ¿Qué te dicen MODULE, ACTION y CLIENT_IDENTIFIER que no te diría USERNAME?**
Tenía `SID = 44` (`SERIAL# = 38513`), `SPID = 2979` y estaba esperando en `PL/SQL lock timer` (`*_03-sesiones.script.log`). Como la aplicación siempre entra con el mismo usuario de base de datos (`APP_ACADEMIA`), mirar solo `USERNAME` no sirve para distinguir qué está haciendo cada petición; con `MODULE` (`matriculas`), `ACTION` (`cierre-periodo`) y `CLIENT_IDENTIFIER` (`ana.garcia@ucam`) sabemos qué pantalla, qué proceso y qué usuario real hay detrás.

**7. Tras CANCEL SQL, ¿cuántas transacciones abiertas tenía la aplicación? ¿Y tras KILL SESSION? ¿En qué estado quedó la fila 101 y por qué?**
Tras `CANCEL SQL` seguía habiendo `1` transacción abierta (`TRANSACCIONES_ABIERTAS_DE_LA_APP = 1`), porque ese comando solo corta la sentencia que está corriendo en ese momento (`DBMS_SESSION.SLEEP`), pero mantiene la sesión y su transacción (`*_03-sesiones.script.log`). Tras `KILL SESSION ... IMMEDIATE` quedaron `0` sesiones vivas y la fila `101` volvió a `ACTIVO`, porque al matar la sesión Oracle hace rollback automático de todo lo que no tuviera `COMMIT`.

**8. ¿Qué valor tiene common_user_prefix en tu imagen? ¿Qué implica para el nombre de una cuenta común?**
Vale `C##` (`*_03-cuenta-app.script.log`). Esto obliga a que cualquier usuario común creado en `CDB$ROOT` empiece por `C##` (como `C##OPS_RO`), mientras que los usuarios locales creados dentro de una PDB (como `APP_ACADEMIA`) no pueden llevar ese prefijo.

**9. Completa con tus cifras la tabla de las tres miradas (ALL_TABLES, DBA_USERS, CDB_USERS, V$DATAFILE) y explica una diferencia.**

| Vista | Mirada 1 (`C##OPS_RO` en `CDB$ROOT`) | Mirada 2 (`C##OPS_RO` en `FREEPDB1`) | Mirada 3 (`APP_ACADEMIA` en `FREEPDB1`) |
| :--- | :---: | :---: | :---: |
| `ALL_TABLES` (de `ADMIN_ACADEMIA`) | 0 | 0 | 1 |
| `DBA_USERS` | 37 | 47 | `ORA-00942` (sin permiso) |
| `CDB_USERS` | `CON_ID 1`: 37 / `CON_ID 3`: 47 | `CON_ID 3`: 47 | `ORA-00942` (sin permiso) |
| `V$DATAFILE` | 17 | 10 | `ORA-00942` (sin permiso) |

En `ALL_TABLES`, `C##OPS_RO` ve `0` tablas de `ADMIN_ACADEMIA` porque `ALL_*` solo enseña objetos sobre los que tienes permisos directos, mientras que `APP_ACADEMIA` ve `1` (`ACA_ESTUDIANTES`) porque tiene `SELECT` y `UPDATE` concedidos sobre ella (`*_04-tres-miradas.script.log`).

**10. Con tu salida de 04-deltas: ¿cuántas lecturas lógicas por recorrido y cuánto DB time costó la carga? ¿Qué indica la relación entre DB time y DB CPU?**
La carga costó `1.92` segundos de `DB time`, `1.83` segundos de `DB CPU` y `5408` lecturas lógicas por recorrido (`54076` en total en los 10 recorridos, `*_04-deltas.script.log`). Que el `DB CPU` (`1.83 s`) sea casi igual que el `DB time` (`1.92 s`) indica que la consulta trabajó casi todo el tiempo tirando de CPU con los datos ya en caché, sin apenas esperas de disco ni bloqueos.

## Partes F-G · Espacio y acceso

**11. Da las tres medidas (asignado, libre, máximo) de TBS_ACADEMIA_HIST antes de la carga y explica cuál usarías para alertar.**
Antes de la carga tenía `10` MB asignados, `3.9` MB libres y `40` MB de máximo (`*_05-almacenamiento.script.log`). Para lanzar alertas usaría el porcentaje usado respecto al máximo (`PCT_SOBRE_MAX`, que era `15.16 %`), porque los datafiles tienen autoextensión hasta 40 MB y avisar sobre los 10 MB iniciales daría falsas alarmas cuando aún tiene margen de sobra para crecer.

**12. ¿Quién insertó y quién «pagó» en el ORA-01950? ¿Por qué ampliar el tablespace no habría resuelto el ORA-01536?**
Insertó `SYS` (que era el usuario conectado al lanzar el script), pero la cuota la pagó `CARGA_HIST`, que es el dueño de la tabla `REGISTRO` (`*_05-almacenamiento.script.log`). Ampliar el tablespace no habría arreglado el `ORA-01536` porque al tablespace aún le sobraba espacio hasta los 40 MB; lo que se agotó fue la cuota individual de `2 MB` que tenía asignada `CARGA_HIST`.

**13. ¿Cuánto tiempo estuvo suspendida la carga reanudable y qué evidencia de ello dejó Oracle en el alert log?**
La carga empezó a las `11:35:53` y terminó a las `11:36:10` (17 segundos en total, estando suspendida unos 12-13 segundos hasta que ampliamos la cuota, `*_05-resumable.script.log`). En el alert log Oracle dejó dos líneas seguidas: `statement in resumable session 'carga-historico' was suspended due to ORA-01536` y `statement in resumable session 'carga-historico' was resumed`.

**14. Antes y después del SHRINK: filas, bloques y cuota consumida. ¿Por qué un DELETE no devolvió el espacio?**
Antes de compactar había `20000` filas en `2944` bloques (`23` MB); tras borrar el 90 % quedaron `2000` filas pero seguían ocupando `2944` bloques (`23` MB); y tras el `SHRINK SPACE` quedaron `2000` filas en `304` bloques con `2.4` MB usados (`*_05-hwm-shrink-post.script.log`). El `DELETE` solo borra las filas por dentro sin bajar la marca de agua alta (HWM), así que los bloques siguen reservados para la tabla hasta que se compacta.

**15. ¿Qué valores tenía tu perfil DEFAULT? Justifica dos de los límites de PRF_PERSONAS para una persona de secretaría.**
El perfil `DEFAULT` tenía `FAILED_LOGIN_ATTEMPTS = 10`, `PASSWORD_LIFE_TIME = 180`, `PASSWORD_GRACE_TIME = 7`, `PASSWORD_LOCK_TIME = 1`, `INACTIVE_ACCOUNT_TIME = 365`, `PASSWORD_ROLLOVER_TIME = 0`, `PASSWORD_REUSE_MAX = UNLIMITED`, `PASSWORD_REUSE_TIME = UNLIMITED` y `PASSWORD_VERIFY_FUNCTION = NULL` (`*_06-gobierno-acceso.script.log`). En `PRF_PERSONAS` bajamos `FAILED_LOGIN_ATTEMPTS` a `3` para cortar rápido intentos de adivinar contraseñas en cuentas personales, y activamos una función de verificación de complejidad (`ORA-28003`) porque las personas tienden a poner claves sencillas si no se les exige longitud y caracteres especiales.

**16. ¿Por qué Ana pudo leer ACA_MATRICULAS si nadie se la concedió por nombre? ¿Qué error obtuvo al cambiar el nombre de un alumno?**
Porque `ANA_GARCIA` tiene el rol `ROL_SECRETARIA_ACADEMIA`, que hereda `ROL_LECTURA_ACADEMIA`, y ese rol tiene `SELECT ANY TABLE ON SCHEMA ADMIN_ACADEMIA` (`*_06-gobierno-acceso.script.log`). Al intentar cambiar la columna `nombres` le saltó `ORA-41900: missing UPDATE privilege on "ADMIN_ACADEMIA"."ACA_ESTUDIANTES"`, porque solo tiene permiso de `UPDATE` sobre la columna `ESTADO` (`*_06-probar-acceso.script.log`).

## Partes H-I · Objetos, diagnóstico y guardia

**17. ¿Qué objetos se invalidaron en cada uno de los tres cambios de columna y por qué no se invalidó nada con la columna nueva?**
Al añadir la columna nueva no se invalidó nada porque Oracle controla las dependencias a nivel de columna y ningún objeto la usaba. Al ampliar `email_institucional` solo se invalidó `V_CONTACTO_LAB6` (la única vista que lee ese campo), y al ampliar `nombres` se invalidaron `V_ACTIVOS_LAB6` y, por dependencia suya, `F_TOTAL_ACTIVOS_LAB6` (`*_07-objetos.script.log`).

**18. Según tus advisors, ¿mejoraría el DB time duplicar la SGA? ¿Y por debajo de qué tamaño la PGA tendría sobreasignaciones?**
No mejoraría: con `FACTOR = 1` (`1536` MB) el `ESTD_DB_TIME_FACTOR` vale `1` y duplicándola a `3072` MB (`FACTOR = 2`) sigue valiendo `1` (`*_07-memoria.spool.log`). En la PGA empiezan las sobreasignaciones por debajo de `384` MB (`FACTOR = 0.75`), donde `OVERALLOC` pasa a `1` (y sube a `18` con 256 MB y `19` con 128 MB).

**19. ¿Cuántas lecturas lógicas necesitó la consulta de la aplicación según la traza (o tkprof)? ¿Qué línea de la traza lo dice?**
Necesitó `247` lecturas lógicas en modo consistente (`query = 247`, `current = 0`, `disk = 0`, `*_07-adr-trazas.script.log`). En la traza en bruto lo pone el campo `cr=247` de la línea `FETCH #128802858087216:c=8809,e=12244,p=0,cr=247,cu=0,mis=0,r=1...`, y en `tkprof` sale en la columna `query` de la fila `Fetch` y `total`.

**20. Para cada incidente de la guardia: evidencia clave, intervención y validación (una línea por cada uno).**
- **INC-601 (bloqueo de fila):** `SID 17` (`ANA_GARCIA`) esperaba en `enq: TX - row lock contention` por `SID 47` (`APP_ACADEMIA`, `USED_UREC=1`); al terminar `SID 47` (expiró justo antes del `KILL SESSION`) se validó en `08-diagnostico.sql` con `0` sesiones bloqueadas y rastro en ASH (`*_08-diagnostico-incidente1*.script.log`).
- **INC-602 (`ORA-12514`):** `08-diagnostico.sql` mostró el servicio `academia_app` con `ARRANCADO = NO` mientras `FREEPDB1` seguía `SI`; se arrancó con `DBMS_SERVICE.START_SERVICE('academia_app')` y se validó `ARRANCADO = SI` (`*_08-diagnostico-incidente2*.script.log`).
- **INC-603 (`ORA-28000`):** `08-diagnostico.sql` detectó `APP_ACADEMIA` con `ACCOUNT_STATUS = LOCKED` (bloqueo manual, no `LOCKED(TIMED)`); se desbloqueó con `ALTER USER app_academia ACCOUNT UNLOCK` y se validó sin cuentas bloqueadas (`*_08-diagnostico-incidente3*.script.log`).