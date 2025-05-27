
# Información del proyecto y datos que he usado.

## Pruebas para ver cómo funcionan las Media Query.

---

### Prueba MJML

```mjml
<mjml>
  <mj-head>
	<mj-title>¡Hola, {{nombre}}!</mj-title>
	<mj-attributes>
  	<mj-all font-family="Arial, sans-serif" />
  	<mj-text color="#333" font-size="16px" line-height="24px" />
  	<mj-button background-color="#007bff" color="#ffffff" border-radius="5px" />
	</mj-attributes>
	<mj-style>
  	.custom-media-query {
      	font-size: 14px !important;
  	}
  	/* Media Query 1: Para pantallas muy pequeñas */
  	@media only screen and (max-width: 480px) {
      	.mobile-hidden { display: none !important; }
      	.full-width-on-mobile { width: 100% !important; display: block !important; }
  	}
  	/* Media Query 2: Para tablets (orientación retrato) */
  	@media only screen and (min-width: 481px) and (max-width: 768px) {
      	.tablet-adjust-text { font-size: 18px !important; }
  	}
  	/* Media Query 3: Ejemplo de ajuste para pantallas grandes */
  	@media only screen and (min-width: 769px) {
      	.desktop-specific-padding { padding: 40px !important; }
  	}
	</mj-style>
  </mj-head>
  <mj-body background-color="#f4f4f4">
	<mj-section background-color="#ffffff" padding="20px">
  	<mj-column>
      	<mj-image src="[https://www.feriavalencia.com/wp-content/uploads/Feria-Valencia-Logo.png](https://www.feriavalencia.com/wp-content/uploads/Feria-Valencia-Logo.png)" alt="Logo Feria Valencia" width="150px" />
      	<mj-text align="center" font-size="24px" font-weight="bold">¡Bienvenido a Feria Valencia, {{nombre}}!</mj-text>
      	<mj-text>Estimado/a **{{nombre}}** de **{{empresa}}**,</mj-text>
      	<mj-text>Estamos encantados de invitarte a nuestro próximo evento. Esperamos verte allí.</mj-text>
      	<mj-button href="[https://www.feriavalencia.com/eventos/](https://www.feriavalencia.com/eventos/)" align="center">Ver Eventos</mj-button>
      	<mj-text css-class="mobile-hidden">Este texto se oculta en móviles.</mj-text>
      	<mj-text css-class="tablet-adjust-text">Este texto se ajusta en tablets.</mj-text>
      	<mj-text css-class="desktop-specific-padding">Conoce todas las novedades de Feria Valencia.</mj-text>
      	<mj-spacer height="20px" />
      	<mj-text font-size="12px" color="#999" align="center">Este correo ha sido enviado a {{email}}.</mj-text>
  	</mj-column>
	</mj-section>
  </mj-body>
</mjml>
```

## Tablas para la BBDD.

### Tabla suscriptores.

```
CREATE TABLE suscriptores (
	id_suscriptor INT AUTO_INCREMENT PRIMARY KEY,
	nombre VARCHAR(255) NOT NULL,
	email VARCHAR(255) NOT NULL UNIQUE,
	edad INT,
	idioma VARCHAR(50) NOT NULL DEFAULT 'es'
);
```
### Tabla newsletters.

```
-- Crea la tabla newsletters
CREATE TABLE newsletters (
	id_newsletter INT AUTO_INCREMENT PRIMARY KEY,
	nombre VARCHAR(255) NOT NULL,
	descripcion TEXT
);
```
### Tabla envios.

```
CREATE TABLE IF NOT EXISTS envios (
	id_envio INT AUTO_INCREMENT PRIMARY KEY,
	id_suscriptor INT NOT NULL, -- Clave foránea que apunta a la tabla 'suscriptores'
	id_newsletter INT NOT NULL, -- Clave foránea que apunta a la tabla 'newsletters'
	fecha_envio DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP, -- Fecha y hora en que se realizó el envío
	asunto VARCHAR(255) NOT NULL, -- Asunto del correo electrónico enviado
	estado_envio VARCHAR(50) NOT NULL, -- Estado del envío (ej. 'Enviado', 'Abierto', 'Rebotado')

	-- Definición de las claves foráneas para asegurar la integridad de los datos
	FOREIGN KEY (id_suscriptor) REFERENCES suscriptores(id_suscriptor) ON DELETE RESTRICT ON UPDATE CASCADE,
	FOREIGN KEY (id_newsletter) REFERENCES newsletters(id_newsletter) ON DELETE RESTRICT ON UPDATE CASCADE
);
```

## Tablas ficticias de relleno para la BBDD.

```
USE DosRuedasFeria;

-- 1. Insertar Datos en 'suscriptores' 
INSERT INTO suscriptores (nombre, email, edad, idioma) VALUES
('Elena Navarro', 'elena.navarro@webmail.com', 31, 'es'),
('Mark Johnson', 'mark.johnson@corp.com', 48, 'en'),
('Isabelle Moreau', 'isabelle.m@provider.fr', 24, 'fr'),
('Juanito Alimaña', 'juanito.lima@barrio.es', 65, 'es'),
('Emily White', 'emily.white@newco.org', 27, 'en');

-- 2. Insertar Datos en 'newsletters' 
INSERT INTO newsletters (nombre, descripcion) VALUES
('Actualidad Tecnológica', 'Las últimas noticias y análisis del mundo de la tecnología.'),
('Recetas Saludables', 'Ideas de comidas y consejos para una vida más sana.'),
('Finanzas para Dummies', 'Guía básica para entender el dinero y tus inversiones.'),
('Viajes y Aventuras', 'Inspiración y guías para tus próximas escapadas.');

-- 3. Insertar Datos en 'envios'
INSERT INTO envios (id_suscriptor, id_newsletter, fecha_envio, asunto, estado_envio) VALUES
(1, 1, '2025-05-27 10:00:00', 'Tech News Digest: Mayo 2025', 'Enviado'),
(2, 4, '2025-05-27 14:15:00', 'Your Next Adventure Awaits!', 'Abierto'),
(3, 2, '2025-05-26 11:45:00', 'Recette saine de la semaine', 'Enviado'),
(4, 3, '2025-05-25 09:30:00', 'Guía Rápida de Inversión', 'Clic'),
(5, 1, '2025-05-24 16:00:00', 'The Latest in AI: Weekly Update', 'Rebotado');
```

## Introducir los suscriptores a la BBDD pasándole como argumento el CSV.

```
#!/bin/bash

DB_HOST="localhost"
DB_USER="usuario_app"
DB_PASS="tu_contraseña_fuerte_aqui"
DB_NAME="marketing_db"

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

## Enviar los correos pasándole como argumentos el archivo CSV y el propio correo.

```
const nodemailer = require('nodemailer');
const fs = require('fs');
const path = require('path');
const { execSync } = require('child_process');
const csv = require('csv-parser');

// --- 1. Configuración de Nodemailer ---
const transporter = nodemailer.createTransport({
	host: 'smtp.gmail.com', // O el SMTP de O365 (ej: 'smtp.office365.com')
	port: 587,
	secure: false,
	auth: {
    	user: 'luciferma14@gmail.com,
    	pass: 'ckob twyk bfdb xdvc'
	}
});

// --- 2. Leer suscriptores desde CSV (por argumento) ---
function leerSuscriptoresDesdeCSV(rutaCSV) {
	return new Promise((resolve, reject) => {
    	const suscriptores = [];
    	fs.createReadStream(rutaCSV)
        	.pipe(csv())
        	.on('data', (row) => suscriptores.push(row))
        	.on('end', () => resolve(suscriptores))
        	.on('error', reject);
	});
}

// --- 3. Ruta a tu plantilla MJML (por argumento) ---
const rutaCSV = process.argv[2];
const mjmlTemplateArg = process.argv[3];

if (!rutaCSV || !mjmlTemplateArg) {
	console.error('Uso: node sendEmail.js arch.csv arch.mjml');
	process.exit(1);
}

const rutaCompletaCSV = path.isAbsolute(rutaCSV) ? rutaCSV : path.join(__dirname, rutaCSV);
const mjmlTemplatePath = path.isAbsolute(mjmlTemplateArg) ? mjmlTemplateArg : path.join(__dirname, mjmlTemplateArg);
const mjmlTemplateContent = fs.readFileSync(mjmlTemplatePath, 'utf8');

// --- Función para personalizar y compilar MJML ---
function compileMjmlToHtml(template, data) {
	let personalizedMjml = template;
	for (const key in data) {
    	personalizedMjml = personalizedMjml.replace(new RegExp(`\\{\\{${key}\\}\\}`, 'g'), data[key]);
	}

	// Guarda el MJML personalizado temporalmente en un archivo
	const tempMjmlFilePath = path.join(__dirname, 'temp_personalized_arch.mjml');
	fs.writeFileSync(tempMjmlFilePath, personalizedMjml);

	// Ejecuta el comando MJML CLI para compilarlo a HTML
	try {
    	const htmlOutput = execSync(`mjml "${tempMjmlFilePath}"`).toString();
    	fs.unlinkSync(tempMjmlFilePath); // Elimina el archivo temporal
    	return htmlOutput;
	} catch (error) {
    	console.error('Error al compilar MJML:', error.stderr ? error.stderr.toString() : error.message);
    	fs.unlinkSync(tempMjmlFilePath);
    	throw new Error('Fallo la compilación de MJML.');
	}
}

// --- Función para enviar un correo ---
async function sendEmail(subscriberData, htmlContent) {
	try {
    	const info = await transporter.sendMail({
        	from: '"Feria Valencia" <tu_correo@gmail.com>',
        	to: subscriberData.email,
        	subject: `¡Tenemos novedades para ti, ${subscriberData.nombre}!`,
        	html: htmlContent,
    	});
    	console.log(`Correo enviado a ${subscriberData.email}: %s`, info.messageId);
    	return { success: true, subscriber: subscriberData.email, messageId: info.messageId };
	} catch (error) {
    	console.error(`Error al enviar correo a ${subscriberData.email}:`, error);
    	return { success: false, subscriber: subscriberData.email, error: error.message };
	}
}

// --- Bucle principal para enviar a todos los suscriptores ---
async function sendCampaign() {
	console.log('Iniciando envío de campaña con:', rutaCompletaCSV);
	const results = [];
	const subscribers = await leerSuscriptoresDesdeCSV(rutaCompletaCSV);

	for (const subscriber of subscribers) {
    	try {
        	const htmlEmail = compileMjmlToHtml(mjmlTemplateContent, subscriber);
        	const result = await sendEmail(subscriber, htmlEmail);
        	results.push(result);
    	} catch (error) {
        	console.error(`No se pudo procesar el correo para ${subscriber.email}:`, error.message);
        	results.push({ success: false, subscriber: subscriber.email, error: error.message });
    	}
	}
	console.log('\nResultados del envío:');
	console.table(results);
}

// --- Ejecutar la campaña ---
sendCampaign();
```