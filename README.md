# Backups automágicos de bases de datos PostgreSQL

Pequeño *script* de *bash* para llevar a cabo copias de seguridad de bases de datos PostgreSQL, al que acompaña un fichero de configuración donde se definen sus modos de ejecución.


## Opciones del «backup» a partir del fichero de configuración

* **HOST** (opcional) → Nombre o IP de la máquina donde se enuentran las bases de datos a respaldar. Si se deja en blanco se supondrá la misma máquina en la que se ejecuta el _script_, esto es, *localhost*.
* **USERNAME** (opcional) → El rol que se usará para conectar a las bases de datos. Si no se informa, se usará *postgres*.
* **BACKUP_DIR** → Directorio donde se almacenan las copias.
* **ENABLE_GLOBALS_BACKUPS** ( *yes*, por defecto | *no* ) → Habilita o deshabilita la copia de objetos globales: `pg_dumpall --globals-only`. Más información sobre `pg_dumpall` y sus opciones, [aquí](https://www.postgresql.org/docs/current/app-pg-dumpall.html).
* **SCHEMA_ONLY_LIST** (opcional) → listado de bases de datos de las que solo queremos respaldar el esquema. Pueden ir separadas por puntos o por comas, y tener el nombre de manera parcial; por ejemplo, si ponemos «system_log» respaldaremos el esquema de «system_log_2021» y «system_log_2022».
* **ENABLE_PLAIN_BACKUPS** ( *yes*, por defecto | *no* ) → Habilita o deshabilita la copia en formato SQL plano, que es el modo por defecto de `pg_dump`: `pg_dump --format=p`. Más información sobre `pg_dump` y sus opciones, [aquí](https://www.postgresql.org/docs/current/app-pgdump.html).
* **ENABLE_CUSTOM_BACKUPS** ( *yes*, por defecto | *no* ) → Habilita o deshabilita la copia en formato *custom*: `pg_dump --format=c`. Este formato es uno de los más flexibles, ya que permite la selección manual y el reordenamiento de los elementos archivados durante la restauración de la copia con `pg_restore`. Además, se comprime por defecto al vuelo.
* **DAY_OF_WEEK_TO_KEEP** ( 1-7, lunes-domingo) → Día de la semana en el que se ejecutará las copias semanales de las bases de datos.
* **DAYS_TO_KEEP** → Número de días que se conservarán las copias diarias de las bases de datos.
* **WEEKS_TO_KEEP** → Número de semanas que se conservarán las copias semanales de las bases de datos.

### Ejemplo de fichero de configuración:
```
##############################
## POSTGRESQL BACKUP CONFIG ##
##############################

HOST=192.168.243.21

USERNAME=tevadm

BACKUP_DIR=/u01/backups/database/postgresql/

ENABLE_GLOBALS_BACKUPS=no

SCHEMA_ONLY_LIST=""

ENABLE_PLAIN_BACKUPS=no

ENABLE_CUSTOM_BACKUPS=yes

#### SETTINGS FOR ROTATED BACKUPS ####
# Which day to take the weekly backup from (1-7 → Monday-Sunday)
DAY_OF_WEEK_TO_KEEP=7

# Number of days to keep daily backups
DAYS_TO_KEEP=7

# How many weeks to keep weekly backups
WEEKS_TO_KEEP=4
######################################
```

## Programación de la ejecución

Para poder delegar la ejecución del _script_ a `crond`, esto es, hacer las copias automágicamente de manera desatendida, debemos crear un fichero `pgpass` en la `home` del usuario y dar de alta las credenciales necesarias para las cadenas de conexión a las bases de datos:

```
vim ~/.pgpass
*:5432:*:username:password
```

Importante no olvidar los permisos adecuados que debe tener el fichero `pgpass`:
```
chmod 0600 ~/.pgpass
```

Más info sobre los ficheros `pgpass`, [aquí](https://www.postgresql.org/docs/current/libpq-pgpass.html) 
