##### ReadMe creado por Lucía Ferrandis Martínez.
---
# Información sobre el envío de correos.

En este ReadMe documentaré como he realizado el manejo de envíos automatizados pasándole como argumentos los suscriptores y el correo del Supuesto 4.

## Construcción de los archivos.

#### Comenzamos creando el archivo JavaScript.

- `enviarCorreo.js`

```js
const nodemailer = require('nodemailer');
const fs = require('fs');
const path = require('path');
const { execSync } = require('child_process');
const csv = require('csv-parser');

// --- 1. Configuración de Nodemailer ---
const transporter = nodemailer.createTransport({
	host: 'smtp.gmail.com', 
	port: 587,
	secure: false,
	auth: {
    	user: 'luciferma14@gmail.com,
    	pass: '**************' // Censurado por seguridad
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
	console.error('Uso: node enviarCorreo.js arch.csv arch.mjml');
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

// --- Enviar un correo ---
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

// --- Enviar a todos los suscriptores ---
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

sendCampaign();
```
--- 
## ¿Por qué pasarlo todo por argumentos?
El hecho de pasar los archivos como argumentos, nos ayuda a poder reutilizar el código todo lo que deseemos, ya que nos permite mantener la estructura sin tener que cambiar ningún datos dentro del código.

## ¿Cómo usamos este archivo?

#### Para ejecutar `enviarCorreo.js`:

- Primero nos aseguramos que tiene permiso de ejecución:

```bash
chmod +x enviarCorreo.js
```
- Y lo ejecutamos pasandole el archivo CSV y el correo:

```bash
node enviarCorreo.js arch.csv arch.mjml
```
