# 📘 Tutorial: Cómo recibir leads de publicaciones de autos e inmuebles en Mercado Libre

Este tutorial te guía paso a paso para integrar tu sistema con la API de Mercado Libre y recibir leads (consultas de potenciales compradores) en tiempo real de publicaciones de **autos** e **inmuebles**.

---

## 🧩 ¿Qué se puede hacer?

* Crear una aplicación oficial en Mercado Libre.
* Autenticar usuarios (vendedores) con OAuth2.
* Obtener sus publicaciones.
* Suscribirse a **notificaciones (webhooks)**.
* Recibir leads de publicaciones de categorías como vehículos o inmuebles.
* Integrar esa información en tu propio sistema.

---

## 🔁 Flujo general

1. Crear aplicación en Mercado Libre.
2. Implementar autenticación OAuth2.
3. Obtener token de acceso del vendedor.
4. Recibir notificaciones cuando alguien consulta una publicación.
5. Consultar los datos del lead con el token.
6. Procesar y guardar los leads en tu sistema.

---

## 🧱 Paso 1: Crear una aplicación en Mercado Libre

1. Ir a: [https://developers.mercadolibre.com.ar/apps](https://developers.mercadolibre.com.ar/apps)
2. Crear una nueva app:

   * Elegir nombre y país.
   * Definir URL de redirección (ej: `https://tusitio.com/callback`).
   * Guardar el **App ID** y **App Secret**.
   * Definir una **URL de notificaciones** (puede ser modificada luego).

Más info: [https://developers.mercadolibre.com.ar/es\_ar/crea-una-aplicacion-en-mercado-libre-es](https://developers.mercadolibre.com.ar/es_ar/crea-una-aplicacion-en-mercado-libre-es)

---

## 🔐 Paso 2: Autenticación OAuth2

### 1. Redirigir al usuario:

```url
https://auth.mercadolibre.com.ar/authorization?response_type=code&client_id=APP_ID&redirect_uri=REDIRECT_URI
```

### 2. Recibir el `code` en tu `REDIRECT_URI` y hacer un POST:

```bash
POST https://api.mercadolibre.com/oauth/token
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code
&client_id=APP_ID
&client_secret=APP_SECRET
&code=CODE
&redirect_uri=REDIRECT_URI
```

### 3. Recibirás:

```json
{
  "access_token": "...",
  "refresh_token": "...",
  "user_id": 12345678,
  "expires_in": 21600
}
```

Más info: [https://developers.mercadolibre.com.ar/es\_ar/autenticacion-y-autorizacion](https://developers.mercadolibre.com.ar/es_ar/autenticacion-y-autorizacion)

---

## 📦 Paso 3: Suscribirse a notificaciones (webhooks)

### 1. Desde tu app configurás la URL de notificación:

```json
https://tusitio.com/api/ml/webhook
```

### 2. Mercado Libre te enviará un `POST` cuando haya un lead:

```json
{
  "resource": "/leads/1234567890",
  "topic": "lead",
  "user_id": 123456,
  "application_id": 987654321,
  "sent": "2024-01-01T12:00:00.000Z"
}
```

### 3. Consultar los datos del lead:

```bash
GET https://api.mercadolibre.com/leads/1234567890?access_token=ACCESS_TOKEN
```

Más info: [https://developers.mercadolibre.com.ar/es\_ar/productos-recibe-notificaciones](https://developers.mercadolibre.com.ar/es_ar/productos-recibe-notificaciones)

---

## 🛠 Cómo integrarlo con tu sistema propio

1. **Crear un endpoint** en tu backend para recibir los POST del webhook.
2. Validar que el `topic` sea `lead`.
3. Usar el `resource` para obtener la info detallada del lead.
4. Guardar en tu base de datos (ej: MongoDB, PostgreSQL).
5. Notificar a tus vendedores o sistema interno.

---

## 🧪 Tips para desarrollo y testing

* Podés usar [ngrok](https://ngrok.com) para exponer tu servidor local.
* Usá Postman para testear los endpoints.
* Guardá y refrescá el `access_token` automáticamente (usando el `refresh_token`).

---

## ✅ Requisitos y consideraciones

* Las publicaciones deben estar en categorías de **autos** o **inmuebles**.
* El vendedor debe tener activada la opción de recibir contactos.
* El lead solo puede consultarse con el token del vendedor asociado.
* La app debe estar en producción o modo sandbox para pruebas controladas.

---

## 🧭 Recursos útiles

* Crear app: [https://developers.mercadolibre.com.ar/es\_ar/crea-una-aplicacion-en-mercado-libre-es](https://developers.mercadolibre.com.ar/es_ar/crea-una-aplicacion-en-mercado-libre-es)
* Autenticación: [https://developers.mercadolibre.com.ar/es\_ar/autenticacion-y-autorizacion](https://developers.mercadolibre.com.ar/es_ar/autenticacion-y-autorizacion)
* Notificaciones: [https://developers.mercadolibre.com.ar/es\_ar/productos-recibe-notificaciones](https://developers.mercadolibre.com.ar/es_ar/productos-recibe-notificaciones)
* Docs generales: [https://developers.mercadolibre.com.ar/es\_ar/api-docs-es](https://developers.mercadolibre.com.ar/es_ar/api-docs-es)

---

## 📌 Resumen paso a paso

1. Crear una aplicación en Mercado Libre y guardar los datos.
2. Implementar login OAuth2 en tu app para obtener el `access_token`.
3. Configurar URL de webhook.
4. Al recibir un POST de tipo `lead`, obtener la data con el token.
5. Integrar y guardar los datos en tu sistema.
6. Automatizar notificaciones internas o dashboards de gestión de leads.

---

Si necesitás que te genere un proyecto base con Next.js, Node.js o Express para esto, avisame y lo armamos. 🚀
