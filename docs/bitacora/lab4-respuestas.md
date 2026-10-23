# Lab 4 - Respuestas

Evidencias en `docs/bitacora/lab4/evidencia/` (script, spool y config). Entorno: Oracle AI Database 26ai Free, contenedor `oralab-26ai`.

## Preguntas de comprobacion

### Respaldo

**1. Por que la copia de la Parte B es consistente? Y con la base abierta?**
Se hizo tras `SHUTDOWN IMMEDIATE`: hay checkpoint final, todos los datafiles comparten el mismo SCN y al restaurar no hace falta aplicar redo. Con la base abierta seria inconsistente y solo serviria con ARCHIVELOG y RMAN. Evidencia: `*_00-backup-lab3.script.log`, paso 4 (parada limpia).

**2. Para que sirven `manifiesto.txt` y `SHA256SUMS`? Que comprueba `restaurar-estado.sh`?**
El manifiesto dice que es la copia (etiqueta, fecha, imagen, commit); `SHA256SUMS` guarda la huella de cada archivo. Antes de tocar nada, `restaurar-estado.sh` ejecuta `sha256sum --check` y se detiene si algo no coincide. Evidencia: `backups/20261023T114451Z_lab3-final` integro.

**3. Define RPO y RTO con los datos de tu copia.**
RPO: se pierde todo lo posterior a la copia, es decir, lo hecho despues del estado del Lab 3 (23-oct-2026, 11:44 UTC). RTO: lo que tarda `restaurar-estado.sh` en devolver el volumen y arrancar la base (copia de unos cientos de MB, segun el manifiesto). No medi una restauracion completa.

### Diagnostico y correcciones

**4. Donde creo Oracle los datafiles de V000 y por que? Que parametro lo habria evitado?**
V000 uso rutas relativas (`academia01.dbf`) y no hay `db_create_file_dest` (sin OMF), asi que Oracle los pone relativos a `$ORACLE_HOME/dbs`. En esta imagen `dbs` es un enlace al volumen (`dbconfig/FREE/dbs`), pero es una carpeta no estandar y depende de un enlace. Lo evita una ruta absoluta o `db_create_file_dest`. Evidencia: `*_01-diagnostico.script.log`.

**5. Por que `RENAME DATAFILE` no basta para mover un archivo?**
Solo cambia la ruta que guarda el control file; no copia el archivo. Hay que poner el tablespace OFFLINE, copiar con el sistema operativo, hacer el RENAME y volver a ponerlo ONLINE. Evidencia: `*_03-reubicar-datafiles.script.log`.

**6. Por que se crea V003 en vez de editar V000? Por que invalidar una contrasena es mejor que borrarla?**
V000 ya se aplico: una migracion aplicada es historia y editarla desincroniza repositorio y base. Borrar la contrasena del archivo no la quita del historial de Git; `NO AUTHENTICATION` la deja inservible. Evidencia: spool de V003 y verificacion final `cuentas ADMIN_* con contrasena (0)`.

**7. Que demuestra la prueba de persistencia y por que va despues de reubicar?**
Que se puede borrar el contenedor y recrearlo sin perder la base: tras recrearlo siguen los 5 tablespaces y los 20000 estudiantes (fila 101 ACTIVO). Va despues de reubicar porque, si los datafiles siguieran fuera del volumen, `docker rm` los borraria. Evidencia: `*_03-prueba-persistencia.script.log`.

### Instancia y arranque

**8. Tres datos de la instancia y tres de la base de datos.**
Instancia: 53 procesos de background `db_*_FREE`; SGA de unos 1,5 GB (`sga_target` 1536M) y PGA; `startup_time` y estado OPEN en `v$instance`. Base: 16 datafiles (3964 MB); 2 control files; 3 grupos de redo online de 200 MB. Evidencia: `*_05-instancia-y-base.spool.log`, `*_11-archivos.spool.log`.

**9. Que lee Oracle en NOMOUNT, MOUNT y OPEN? Relacionalo con ORA-01507 y ORA-01219.**
NOMOUNT: lee el SPFILE, reserva la SGA y arranca los procesos; `v$parameter` responde, pero `v$database` da ORA-01507 porque el control file no esta abierto. MOUNT: abre los control files y conoce los 16 datafiles, pero `dba_tables` da ORA-01219 porque el diccionario esta en SYSTEM, aun cerrado. OPEN: abre datafiles y redo. Evidencia: `*_06-arranque-por-etapas.script.log`.

**10. Por que FREEPDB1 se abre sola? Que harias si no?**
Porque su estado esta guardado (`SAVE STATE`, visible en `dba_pdb_saved_states`). Si no, quedaria en MOUNTED (ORA-01109 para las aplicaciones): `ALTER PLUGGABLE DATABASE freepdb1 OPEN;` y despues `... SAVE STATE;`.

**11. Clasifica ORA-00800 y ORA-20013 del alert log.**
Ruido conocido, no alerta real. ORA-00800 es un aviso (Set Priority Failed en VKTM/LGWR): el contenedor no tiene la capacidad CAP_SYS_NICE. ORA-20013 viene de DBMS_QOPATCH, que no puede leer el inventario de parches en una imagen de contenedor. Se documentan y se excluyen de las alertas; cualquier otro ORA- se investiga.

### Parametros

**12. Diferencia entre `v$parameter` y `v$spparameter`, con `db_files`.**
`v$parameter` es el valor en memoria; `v$spparameter` es lo escrito en el SPFILE. Con `db_files`: tras `SCOPE=SPFILE` 300, memoria 200 y SPFILE 300 (pendiente de reinicio); tras `RESET`, memoria 200 y SPFILE sin especificar. Evidencia: `*_07-parametros-cambios.script.log`.

**13. Que hace `SCOPE=BOTH` y cual es el SCOPE por defecto con SPFILE?**
Cambia memoria y SPFILE a la vez (inmediato y persistente). Con SPFILE el valor por defecto es BOTH; si se arranco con PFILE es MEMORY.

**14. Por que `PROCESSES` rechaza el cambio en memoria aunque `v$parameter` diga IMMEDIATE?**
Porque dimensiona estructuras de la SGA que se reservan al arrancar: da ORA-02095. En la practica es estatico; se cambia con `SCOPE=SPFILE` y reinicio. La metadata orienta, la prueba decide.

**15. Para que sirve versionar el PFILE? Explica configuration drift.**
Es una foto legible de la configuracion: con `git diff` se ven los cambios. Configuration drift es la deriva por cambios que nadie documento. Evidencia: `evidencia/config/initFREE_lab4.ora`.

### Memoria y procesos

**16. Que modelo de memoria usa tu instancia? Justificalo con tres parametros.**
ASMM (SGA automatica, PGA aparte). `memory_target` = 0 (y `memory_size` = 0, sin AMM), `sga_target` = 1536M, `pga_aggregate_target` = 512M. Evidencia: `*_08-memoria.spool.log`.

**17. Explica las cifras del experimento de cache.**
Ronda 1 (cache fria, tras `FLUSH BUFFER_CACHE`): las lecturas fisicas son casi tantas como bloques tiene la tabla (244, mas la cabecera del segmento). Ronda 2 (caliente): 0 fisicas y las mismas lecturas logicas, porque toda lectura fisica termina en una logica. Tasa de acierto = 1 - fisicas/logicas: baja en la ronda 1 y 100 % en la ronda 2. Evidencia: `*_08-experimento-cache.script.log`.

**18. Por que no se suma el RSS de los procesos?**
Porque casi todo el RSS de cada proceso de background es la SGA compartida, repetida en cada uno. Sumar 53 procesos daria mucho mas que los 2 GiB reales del contenedor. La memoria real se mira con `v$sga`, `v$pgastat` o `docker stats`.

**19. Relaciona el SPID de LGWR con el PID de /proc. Por que `docker top` muestra otro numero?**
El SPID de LGWR en `v$process` (1197) coincide con el PID de `db_lgwr_FREE` leido en /proc dentro del contenedor. `docker top` muestra el PID del anfitrion: son el mismo proceso en dos espacios de nombres de PID distintos. Evidencia: `*_09-procesos.spool.log` y `09-procesos-so.script.log`.

**20. Que hace PMON y que hace SMON? Un ejemplo de cada uno.**
PMON limpia los procesos de usuario caidos, deshace su transaccion y libera sus bloqueos (portatil apagado en mitad de un UPDATE). SMON hace la recuperacion de instancia al arrancar tras una caida y limpia segmentos temporales.

### Seguridad y trabajo profesional

**21. Por que `C##OPS_RO` y no SYSTEM para observar? Que no puede hacer?**
Minimo privilegio: mirar no requiere poder romper. Tiene `CREATE SESSION`, `SET CONTAINER`, `SELECT_CATALOG_ROLE` y `SELECT` sobre `V$DIAG_ALERT_EXT`. No puede escribir datos ni hacer DDL, y no tiene privilegios `ANY` de escritura (verificacion final: 0).

**22. Que dos vias de exposicion de contrasenas corrige este laboratorio?**
(1) La contrasena de SYS en la linea de comandos del script 08 (visible con `ps`): la version 2 usa autenticacion del sistema operativo. (2) Las contrasenas literales de los ADMIN_* en V000, que quedan en el historial de Git: V003 las invalida con `NO AUTHENTICATION`.

## Preguntas tipo examen

| P | Respuesta | Justificacion |
|---|---|---|
| P1 | A, D | Tras `SHUTDOWN IMMEDIATE` la copia es consistente y se abre con `STARTUP` sin recuperacion; no exige ARCHIVELOG ni RMAN. |
| P2 | B | Sin Online Move Datafile: OFFLINE, copia, `RENAME DATAFILE` y ONLINE. `RENAME` solo cambia el control file. |
| P3 | A, C | `STARTUP_TIME` cambia en cada arranque y `CREATED` es la fecha de creacion de la base; el DBID no cambia al recrear el contenedor. |
| P4 | A, C | En NOMOUNT responde `v$parameter` y se puede ejecutar `CREATE CONTROLFILE`; lo demas exige MOUNT u OPEN. |
| P5 | B | `ALTER DATABASE ARCHIVELOG` se ejecuta en MOUNT, con la base cerrada. |
| P6 | B | Con SPFILE y sin SCOPE, el valor por defecto es BOTH: memoria y SPFILE. |
| P7 | C | `PROCESSES` es de la instancia; desde una PDB da ORA-65040. |
| P8 | A, C | `STARTUP PFILE=` con un PFILE bueno y `CREATE SPFILE FROM PFILE` para reparar el SPFILE. |
| P9 | B | Con `SGA_TARGET` fijado, ASMM gestiona la SGA y `DB_CACHE_SIZE` actua como minimo. |
| P10 | A, D | El area de ordenacion y la memoria de sesion estan en la PGA; shared pool, log buffer y buffer cache son SGA. |
| P11 | C | LGWR escribe el redo antes de que el COMMIT devuelva el control; DBWn escribe los bloques despues. |
| P12 | B, D | PMON y SMON son obligatorios; ARC0, RECO y CJQ0 dependen de la configuracion. |
