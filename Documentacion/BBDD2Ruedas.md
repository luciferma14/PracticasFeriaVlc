##### ReadMe creado por Lucía Ferrandis Martínez.
---
# Información sobre la BBDD de Feria Dos Ruedas.

En este ReadMe documentaré como he realizado toda la estructura de la BBDD del Supuesto 4 y como añadimos suscriptores de forma automática.

## Construcción de las diferentes partes.

#### Comenzando con la creación de la base de datos:

```sql
CREATE DATABASE DosRuedasFeria;
```

#### Después de esto, creamos las tablas que componen la base:

- Tabla suscriptores

```sql
CREATE TABLE suscriptores (
	id_suscriptor INT AUTO_INCREMENT PRIMARY KEY,
	nombre VARCHAR(255) NOT NULL,
	email VARCHAR(255) NOT NULL UNIQUE,
	edad INT,
	idioma VARCHAR(50) NOT NULL DEFAULT 'es'
);
```
- Tabla newsletters.

```sql
CREATE TABLE newsletters (
	id_newsletter INT AUTO_INCREMENT PRIMARY KEY,
	nombre VARCHAR(255) NOT NULL,
	descripcion TEXT
);
```
- Tabla envios.

```sql
CREATE TABLE IF NOT EXISTS envios (
	id_envio INT AUTO_INCREMENT PRIMARY KEY,
	id_suscriptor INT NOT NULL, -- Clave foránea que apunta a la tabla 'suscriptores'
	id_newsletter INT NOT NULL, -- Clave foránea que apunta a la tabla 'newsletters'
	fecha_envio DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP, -- Fecha y hora en que se hizo el envío
	asunto VARCHAR(255) NOT NULL, 
    
	estado_envio VARCHAR(50) NOT NULL, -- Estado del envío (ej. 'Enviado', 'Abierto', 'Rebotado')

	-- Definición de las claves
	FOREIGN KEY (id_suscriptor) REFERENCES suscriptores(id_suscriptor) ON DELETE RESTRICT ON UPDATE CASCADE,
	FOREIGN KEY (id_newsletter) REFERENCES newsletters(id_newsletter) ON DELETE RESTRICT ON UPDATE CASCADE
);
```

#### Ahora añadiremos datos ficticios a las tablas para poder trabajar con ellos:

```sql
USE DosRuedasFeria;
```
- Introducir datos en 'suscriptores'.

```sql 
INSERT INTO suscriptores (nombre, email, edad, idioma) VALUES
('Elena Navarro', 'elena.navarro@webmail.com', 31, 'es'),
('Mark Johnson', 'mark.johnson@corp.com', 48, 'en'),
('Isabelle Moreau', 'isabelle.m@provider.fr', 24, 'fr'),
('Juanito Alimaña', 'juanito.lima@barrio.es', 65, 'es'),
('Emily White', 'emily.white@newco.org', 27, 'en');
```
- Introducir datos en 'newsletters'.

```sql
INSERT INTO newsletters (nombre, descripcion) VALUES
('Actualidad Tecnológica', 'Las últimas noticias y análisis del mundo de la tecnología.'),
('Recetas Saludables', 'Ideas de comidas y consejos para una vida más sana.'),
('Finanzas para Dummies', 'Guía básica para entender el dinero y tus inversiones.'),
('Viajes y Aventuras', 'Inspiración y guías para tus próximas escapadas.');
```
- Introducir datos en 'envios'.

```sql
INSERT INTO envios (id_suscriptor, id_newsletter, fecha_envio, asunto, estado_envio) VALUES
(1, 1, '2025-05-27 10:00:00', 'Tech News Digest: Mayo 2025', 'Enviado'),
(2, 4, '2025-05-27 14:15:00', 'Your Next Adventure Awaits!', 'Abierto'),
(3, 2, '2025-05-26 11:45:00', 'Recette saine de la semaine', 'Enviado'),
(4, 3, '2025-05-25 09:30:00', 'Guía Rápida de Inversión', 'Clic'),
(5, 1, '2025-05-24 16:00:00', 'The Latest in AI: Weekly Update', 'Rebotado');
```

#### Lo siguiente que haremos será crear un script en Bash para añadir suscriptores desde un cvs pasado como argumento:

- `leerSus.sh`

```bash
#!/bin/bash

DB_HOST="localhost"
DB_USER="lucifer"
DB_PASS="**********" # Censurado por seguridad
DB_NAME="DosRuedasFeria"

if [ -z "$1" ]; then
	echo "Uso: $0 <ruta_al_archivo_csv>"
	echo "Ejemplo: $0 suscriptores.csv"
	exit 1
fi

CSV_FILE="$1"

echo "Iniciando importación de suscriptores desde '$CSV_FILE'..."

if [ ! -f "$CSV_FILE" ]; then
	echo "Error: El archivo CSV '$CSV_FILE' no se encontró."
	exit 1
fi

if ! mysql -h "$DB_HOST" -u "$DB_USER" -p"$DB_PASS" "$DB_NAME" -e "SELECT 1;" &> /dev/null; then
	echo "Error: No se pudo conectar a la base de datos MySQL. Revisa las credenciales y la conectividad."
	exit 1
fi
echo "Conexión a la base de datos MySQL exitosa."

LINE_NUM=1
tail -n +2 "<span class="math-inline">CSV\_FILE" \| while IFS\=',' read \-r nombre email edad\_str idioma; do
LINE\_NUM\=</span>((LINE_NUM + 1))

	nombre=$(echo "<span class="math-inline">nombre" \| xargs\)
email\=</span>(echo "<span class="math-inline">email" \| xargs\)
edad\=</span>(echo "<span class="math-inline">edad\_str" \| xargs\)
idioma\=</span>(echo "$idioma" | xargs)

	if [ -z "$email" ]; then
    	echo "Advertencia: Línea $LINE_NUM: Email vacío encontrado. Saltando línea."
    	continue
	fi

	if ! [[ "<span class="math-inline">edad" \=\~ ^\[0\-9\]\+</span> ]] && [ -n "$edad" ]; then
     	echo "Advertencia: Línea $LINE_NUM: Edad '$edad' no es un número válido. Se intentará insertar como NULL."
     	edad="NULL"
	elif [ -z "$edad" ]; then
     	edad="NULL"
	fi

	SQL_QUERY="INSERT INTO suscriptores (nombre, email, edad, idioma, fecha_registro) VALUES ('$nombre', '$email', ${edad}, '$idioma', NOW()) ON DUPLICATE KEY UPDATE nombre = VALUES(nombre), edad = VALUES(edad), idioma = VALUES(idioma);"

	if mysql -h "$DB_HOST" -u "$DB_USER" -p"$DB_PASS" "$DB_NAME" -s -N -e "$SQL_QUERY"; then
    	echo "Procesado: $email"
	else
    	echo "Error: Línea $LINE_NUM: Fallo al procesar: $email."
	fi
done

echo "Proceso de importación de suscriptores finalizado."
exit 0
```

## ¿Cómo usamos este archivo?

#### Para ejecutar `leerSus.sh`:

- Primero nos aseguramos que tiene permiso de ejecución:

```bash
chmod +x leerSus.sh
```
- Y lo ejecutamos pasándole el archivo CSV:

```bash
bash leerSus.sh suscriptores.csv
```
