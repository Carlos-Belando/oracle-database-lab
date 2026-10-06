# Lab 3: respuestas de comprobacion

## Docker

1. La imagen es la plantilla de solo lectura (en G2, `hello-world`; en G4, `alpine:3.20`). El contenedor es una instancia en ejecucion creada a partir de ella (en G4, `prueba`).
2. En G5 el archivo estaba en el sistema de archivos del contenedor, que es temporal: desaparece al borrar el contenedor. En G6 estaba en un volumen, que vive fuera del contenedor y sobrevive a el.
3. `docker ps` lista solo los contenedores en marcha; `docker ps -a` tambien los detenidos. `Exited (0)` significa que el contenedor termino y lo hizo sin errores.
4. En `-p 8181:8181` el primer numero es el de mi equipo y el segundo el del contenedor. `-p 80:8080` con nginx estaria invertido: nginx escucha en el 80 dentro del contenedor, asi que la pagina no cargaria. Lo correcto es `-p 8080:80`.
5. Oracle ejecuta un servidor de base de datos que se queda escuchando, asi que el contenedor sigue vivo. `hello-world` solo imprime un mensaje y termina.
6. El digest es la huella digital exacta de la imagen. `latest` cambia con el tiempo; el digest no, asi que deja registrada la version exacta que instale.
7. `docker volume rm oralab-26ai-data` borra los datos. `docker rm oralab-26ai` solo borra el contenedor; los datos estan en el volumen y siguen ahi.

## Git, organizacion y evidencia

8. Porque asi el cambio queda trazable, reproducible y revisado por otra persona, como en una empresa. Una carpeta aparte pierde el historial, la conexion con GitHub y la proteccion de `main`.
9. `source` ejecuta el script en mi terminal, por lo que las variables quedan disponibles. `bash` lo ejecuta en una terminal hija y las variables se pierden al terminar. Por eso `00-config.sh` se carga con `source`.
10. `20260915T091230Z` es la fecha y hora en UTC; `02` es el numero del paso que genero la evidencia; `docker` es la descripcion en kebab-case; `.script.log` indica que es salida de terminal.
11. Fija los finales de linea (LF) de los `.sh`, `.sql` y `.md`. Evita que scripts con finales de linea de Windows (CRLF) fallen en Linux con `command not found`.
12. Porque cada commit corresponde a una Parte y tiene valor propio. El merge commit conserva ese historial paso a paso; con squash se perderia.

## Seguridad

13. Capa 1: `.gitignore` para que Git ignore el archivo real. Capa 2: una plantilla `config/.env.example` versionada y sin secretos. Capa 3: mi archivo real local, `config/.env`. Capa 4: cargar las contrasenas con `set -a; source` sin teclearlas. Si me salto la primera, Git podria rastrear y subir el archivo con las contrasenas reales.
14. Porque todo lo que escribo queda en el historial de la terminal (`.bash_history`) en texto plano, aunque el script no se suba a Git.
15. No basta: la contrasena sigue en el historial de Git. Hay que darla por comprometida, cambiarla (rotarla), recrear el contenedor con otra y avisar al docente para limpiar la rama.

## Oracle y herramientas

16. Dentro del contenedor, `SPOOL` escribe en el sistema de archivos del contenedor y `@archivo.sql` no encuentra los archivos de mi equipo (error `SP2-0310`). En su lugar enviamos el SQL por la entrada estandar con `docker exec -i`, capturamos con `tee`, y usamos `SPOOL` desde SQLcl en mi equipo.
17. Hace que el script se detenga en el primer error SQL. Sin esa linea seguiria ejecutando y podria dejar la base a medio configurar sin que me entere.
18. Una migracion es un cambio versionado y ordenado de la base de datos. Una vez aplicadas, `V000` y `V001` no se editan porque otras personas y entornos ya dependen de ellas; los cambios nuevos van en una migracion nueva (`V002`).
19. `FREEPDB1` es la base de datos de trabajo (PDB). `FREE` o un SID llevan al contenedor raiz (CDB), donde no se trabaja.
20. SQLcl aporta autocompletado, historial, formato automatico, conexiones guardadas e integracion con Liquibase. SQL*Plus existe en cualquier servidor Oracle y a veces es lo unico disponible, por eso un DBA debe dominar ambas.

## Entorno de trabajo

21. Git Bash es una emulacion: convierte rutas como `/opt` a rutas de Windows y rompe argumentos de Docker, y necesita `winpty` para terminales interactivas (`docker run -it`). Ademas no trae `ss`, `htop` y otras utilidades, y las herramientas Java tienen problemas con la peticion de contrasenas. En Ubuntu en WSL 2 no hay emulacion.
22. Trabajar en `/mnt/c/...` cruza dos sistemas de archivos, es mas lento, no conserva permisos como el de ejecucion y trae problemas de finales de linea. Recomendamos bash porque es la shell que existe en todos los servidores, asi un script se comporta igual en cualquier sitio.
