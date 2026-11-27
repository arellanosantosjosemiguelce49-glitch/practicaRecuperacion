<p align="center">
  <img src="imagenes/logo1.png" width="140">
  <img src="imagenes/logo2.png" width="140">
</p>

<h1 align="center">Instituto Tecnológico Nacional de México</h1>

<h2 align="center">Práctica recuperación - Cuestionario y anexo</h2>

<p align="center">
<b>Alumno:</b><br>
Arellano Santos Jose Miguel<br><br>
<b>Materia:</b><br>
Administración de base de datos<br><br>
<b>Profesor:</b><br>
Diego Alberto Bollas Paredes
</p>

---

## Cuestionario

### 1. ¿Qué datos se habrían perdido si solo hubieras usado el respaldo de la Parte A?

Se habrían perdido los 2 clientes que agregué después del respaldo, **Pedro Nunez y Laura Vega**. Con solo el respaldo de la Parte A, la base se habría restaurado con los 5 clientes que tenía cuando hice la copia y nada más.

### 2. ¿Para qué sirve la opción `--master-data=2`?

Sirve para que el respaldo guarde, escrito dentro del archivo, **el nombre del archivo del binlog y la posición** en la que iba MariaDB cuando se hizo la copia. Así después sé desde dónde tengo que aplicar el binlog para recuperar lo que pasó después del respaldo. El `2` hace que esa línea quede como comentario y no se ejecute al restaurar; solo sirve para leerla (yo la busqué con `grep`).

### 3. ¿Por qué es necesario activar el binlog antes de que ocurra el error?

Porque el binlog solo guarda los cambios **desde que se activa**. Si el error pasa y apenas después lo activo, no tendría guardados los cambios anteriores y no habría nada que aplicar para recuperarlos.

### 4. ¿Por qué se esperaron 5 segundos antes y después de anotar la hora?

Para que la hora anotada quedara claramente **después de agregar los datos y antes de borrarlos**. El binlog guarda la hora en segundos, y si todo pasara en el mismo segundo, al poner la hora como límite podría quedarse fuera algún dato que sí quería recuperar o incluirse el `DELETE`. Con los 5 segundos de separación el límite queda en medio y es más seguro.

---

## Anexo: términos y herramientas que investigué

### Herramientas y comandos

| Herramienta | Qué es |
|---|---|
| `mysqldump` / `mariadb-dump` | Programa que saca un respaldo de una base en un archivo `.sql`. En versiones desde la 10.5 se llama `mariadb-dump`; en la mía (10.3.39) es `mysqldump`. |
| `mysqlbinlog` / `mariadb-binlog` | Programa que lee el archivo del binlog y lo convierte en instrucciones SQL que se pueden volver a ejecutar. |
| `mariadb` / `mysql` | El programa para conectarse a MariaDB desde la terminal. |
| `mariadb-backup` | Herramienta que hace respaldos copiando los archivos de la base directamente. Es más rápida para bases grandes. |
| `systemctl restart mariadb` | Comando para reiniciar el servicio de MariaDB y que tome los cambios de la configuración. |
| `sudo` | Sirve para ejecutar un comando con permisos de administrador. |
| `nano` | Editor de texto de la terminal; lo usé para editar la configuración. |
| `grep` | Comando que busca una palabra dentro de un archivo. Lo usé para encontrar la línea `CHANGE MASTER`. |
| `50-server.cnf` | Archivo de configuración de MariaDB en Ubuntu. |
| `ls -lh` | Muestra los archivos con su tamaño. |

### Opciones usadas

| Opción | Qué hace |
|---|---|
| `--databases` | Agrega al respaldo la creación de la base y su nombre. |
| `--flush-logs` | Cierra el binlog actual y empieza uno nuevo cuando empieza el respaldo. |
| `--master-data=2` | Guarda en el respaldo (como comentario) el archivo y la posición del binlog. |
| `--start-position` | Le dice a `mysqlbinlog` desde qué posición empezar a leer. |
| `--stop-datetime` | Le dice hasta qué fecha y hora leer; lo que pasó después no se aplica. |
| `--stop-position` | Igual que el anterior, pero con una posición en lugar de una hora. |
| `--no-defaults` | Hace que el programa no lea los archivos de configuración. |
| `-e` | Ejecuta una consulta directo desde la terminal. |
| `>` y `<` | `>` guarda la salida en un archivo; `<` le pasa un archivo a un comando. |
| `\|` (tubería) | Manda la salida de un comando como entrada del siguiente. |

### Términos de bases de datos

| Término | Qué significa |
|---|---|
| **Recuperación** | Regresar la base de datos a un estado correcto después de un fallo o un error. |
| **Binlog (binary log)** | Archivo donde MariaDB va anotando todos los cambios que se hacen (insertar, modificar, borrar). Sirve para recuperar datos y para la replicación. |
| **`log_bin`** | Opción de configuración que activa el binlog y dice dónde se guarda. |
| **`server_id`** | Número que identifica al servidor. Es necesario para usar el binlog. |
| **Recuperación a un punto en el tiempo (PITR)** | Método que usa un respaldo completo más el binlog para dejar la base como estaba en un momento exacto, por ejemplo justo antes de un error. |
| **Posición del binlog** | Número que indica en qué parte del archivo del binlog va un cambio. |
| **Respaldo lógico** | Archivo con instrucciones SQL para volver a crear la base (como el que hace `mysqldump`). |
| **Respaldo físico** | Copia de los archivos reales de la base (como hace `mariadb-backup`). |
| **`DROP DATABASE`** | Instrucción que borra una base de datos completa. |
| **`DELETE` sin `WHERE`** | Borra todos los registros de una tabla. Es un error común. |
| **`SELECT NOW()`** | Muestra la fecha y hora actual del servidor. |
| **ACID** | Propiedades que hacen confiable a una base de datos: atomicidad, consistencia, aislamiento y durabilidad. |
| **Redo log / Undo log** | Registros internos de InnoDB: el redo sirve para volver a aplicar lo confirmado y el undo para deshacer lo que no se confirmó. |
| **Crash recovery** | Recuperación automática que hace InnoDB al reiniciar después de una caída (como un apagón). |
| **`innodb_force_recovery`** | Opción de último recurso para arrancar MariaDB cuando está dañada y poder sacar los datos. |
| **Replicación** | Tener otro servidor copiando los cambios del principal, para que si uno falla el otro lo reemplace. |
| **GTID** | Identificador de cada transacción que se usa en la replicación. |
| **RPO y RTO** | RPO es cuántos datos puedo permitirme perder; RTO es cuánto tiempo puedo estar sin servicio. |
| **Regla 3-2-1** | Tener 3 copias de los datos, en 2 tipos de medios distintos y 1 fuera del lugar principal. |
| **`[mysqld]` / `[mariadbd]`** | Sección del archivo de configuración donde van las opciones del servidor. En versiones nuevas es `[mariadbd]`; en la 10.3 es `[mysqld]`. |
