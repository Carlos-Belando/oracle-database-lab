# Lab 6 - Postmortems de guardia nocturna

## INC-601 · Bloqueo de fila sobre `ACA_ESTUDIANTES` (prioridad ALTA)
- **Síntoma:** `ANA_GARCIA` (secretaría) reporta que la ficha del alumno 101 se queda colgada al guardar.
- **Diagnóstico en solo lectura (`*_08-diagnostico-incidente1.script.log`):** En `V$SESSION` y `V$LOCK`, la sesión `SID=17` (`SERIAL#=55003`, `ANA_GARCIA`) estaba detenida en `enq: TX - row lock contention` sobre la fila `5` del bloque `779` de `ACA_ESTUDIANTES`, bloqueada por `SID=47` (`SERIAL#=27587`, `APP_ACADEMIA`, módulo `matriculas`, acción `cierre-periodo`) en `PL/SQL lock timer` con `LMODE=6` y solo `1` registro de undo (`USED_UREC=1`).
- **Intervención y validación:** Se lanzó `ALTER SYSTEM KILL SESSION '47,27587' IMMEDIATE` (la sesión bloqueadora había acabado de liberar por timeout segundos antes, devolviendo `ORA-00030`), y se confirmó en `08-diagnostico.sql` que no quedaban bloqueos activos y que la traza quedó en ASH (`*_08-diagnostico-incidente1-resuelto.script.log`).

## INC-602 · Caída del servicio `academia_app` (`ORA-12514`, prioridad CRÍTICA)
- **Síntoma:** La aplicación de matrículas recibe `ORA-12514` al intentar conectar y sospecha caída de la base de datos.
- **Diagnóstico en solo lectura (`*_08-diagnostico-incidente2.script.log`):** La instancia y `FREEPDB1` seguían abiertas (`FREEPDB1` con `ARRANCADO = SI`), pero el servicio de aplicación `academia_app` aparecía con `ARRANCADO = NO` en `V$ACTIVE_SERVICES`, por lo que el listener no lo anunciaba.
- **Intervención y validación:** Dentro de `FREEPDB1` se ejecutó `EXEC DBMS_SERVICE.START_SERVICE('academia_app');` y se comprobó con `08-diagnostico.sql` que `academia_app` volvía a figurar con `ARRANCADO = SI` (`*_08-diagnostico-incidente2-resuelto.script.log`).

## INC-603 · Bloqueo manual de `APP_ACADEMIA` (`ORA-28000`, prioridad ALTA)
- **Síntoma:** Tras un despliegue y un script de bajas del turno de tarde, `APP_ACADEMIA` falla al entrar con `ORA-28000` sin haber cambiado su contraseña.
- **Diagnóstico en solo lectura (`*_08-diagnostico-incidente3.script.log`):** En `CDB_USERS`, `APP_ACADEMIA` figuraba con `ACCOUNT_STATUS = LOCKED` (bloqueo administrativo explícito, no `LOCKED(TIMED)` por intentos fallidos).
- **Intervención y validación:** Dentro de `FREEPDB1` se ejecutó `ALTER USER app_academia ACCOUNT UNLOCK;` y se verificó con `08-diagnostico.sql` que no quedaba ninguna cuenta no-Oracle fuera de `OPEN` (`*_08-diagnostico-incidente3-resuelto.script.log`).
