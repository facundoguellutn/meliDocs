# 🔄 Flujo de Integración con Mercado Libre para recibir Leads

Este documento explica paso a paso cómo funciona el flujo de integración con la API de Mercado Libre para recibir leads de publicaciones de autos e inmuebles. Además incluye un ejemplo realista con valores ficticios para entender mejor cómo se conecta todo.

---

## 📌 Paso a paso del flujo

### 1. El vendedor inicia sesión en tu sistema y hace clic en "Conectar con Mercado Libre"

* Se lo redirige a:

```
https://auth.mercadolibre.com.ar/authorization?response_type=code&client_id=123456789&redirect_uri=http://localhost:3001/login-ml
```

### 2. El vendedor acepta los permisos → Mercado Libre redirige a:

```
http://localhost:3001/login-ml?code=TG-123abcXYZ
```

### 3. Tu frontend detecta el `code` y lo envía al backend:

```http
POST http://localhost:3000/api/tokens/exchange
{
  "code": "TG-123abcXYZ"
}
```

### 4. Tu backend hace una petición a Mercado Libre:

```http
POST https://api.mercadolibre.com/oauth/token
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code
&client_id=123456789
&client_secret=abcdef123456
&code=TG-123abcXYZ
&redirect_uri=http://localhost:3001/login-ml
```

### 5. Mercado Libre responde:

```json
{
  "access_token": "APP_USR-abc.def.ghi",
  "refresh_token": "rt-xyz123",
  "expires_in": 21600,
  "user_id": 987654321
}
```

### 6. El backend guarda el token en MongoDB:

```js
{
  userId: 987654321,
  accessToken: "APP_USR-abc.def.ghi",
  refreshToken: "rt-xyz123",
  expiresIn: 21600,
  obtainedAt: new Date()
}
```

---

## 📨 Webhook de lead

### 7. Un usuario en Mercado Libre consulta una publicación del vendedor.

Mercado Libre envía un `POST` a tu webhook:

```json
{
  "resource": "/leads/1122334455",
  "topic": "lead",
  "user_id": 987654321,
  "application_id": 123456789
}
```

### 8. Tu backend recibe eso en `/api/ml/webhook` y hace:

```http
GET https://api.mercadolibre.com/leads/1122334455?access_token=APP_USR-abc.def.ghi
```

### 9. Mercado Libre responde con los datos del lead:

```json
{
  "id": "1122334455",
  "item_id": "MLA123456",
  "created_at": "2025-05-19T15:30:00.000Z",
  "buyer": {
    "first_name": "Juan",
    "email": "juan@mail.com",
    "phone": { "number": "+5493511234567" }
  }
}
```

### 10. Tu backend guarda ese lead:

```js
{
  leadId: "1122334455",
  itemId: "MLA123456",
  buyerName: "Juan",
  buyerEmail: "juan@mail.com",
  buyerPhone: "+5493511234567",
  dateCreated: new Date("2025-05-19T15:30:00.000Z"),
  sellerId: 987654321
}
```

---

## 👁 Visualización desde el frontend

### 11. Tu frontend consulta todos los leads:

```http
GET http://localhost:3000/api/ml/leads
```

### 12. El backend devuelve todos los leads guardados:

```json
[
  {
    "buyerName": "Juan",
    "buyerEmail": "juan@mail.com",
    "buyerPhone": "+5493511234567",
    "itemId": "MLA123456",
    "dateCreated": "2025-05-19T15:30:00.000Z"
  }
]
```

### 13. El frontend los muestra en una tabla o tarjetas.

---

## ✅ Conclusión

Este flujo permite:

* Autenticar a vendedores.
* Recibir y guardar leads de sus publicaciones.
* Consultarlos y visualizarlos desde tu sistema.

Es completamente escalable para varios vendedores y categorías, siempre que las publicaciones estén en autos o inmuebles y tengan contacto habilitado.
