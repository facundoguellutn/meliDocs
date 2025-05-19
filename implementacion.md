# 📘 Tutorial Completo: Integración con la API de Mercado Libre para recibir Leads

Este tutorial te guía paso a paso para integrar tu sistema (backend con Express + MongoDB y frontend con Next.js) con la API de Mercado Libre y recibir **leads** de publicaciones de **autos e inmuebles**.

---

## 🔁 Flujo general

1. Crear una aplicación en Mercado Libre.
2. Implementar login OAuth2 para obtener access token.
3. Guardar access token en tu base de datos.
4. Configurar Webhook para recibir leads.
5. Consultar los datos del lead con el token.
6. Guardar los leads en MongoDB.
7. Visualizar los leads desde el frontend.

---

## 🧱 Backend con Express + MongoDB

### Estructura del proyecto

```
/backend
├── index.js
├── models/
│   ├── Lead.js
│   └── Token.js
├── controllers/
│   ├── leadsController.js
│   └── tokensController.js
└── routes/
    ├── leads.js
    └── tokens.js
```

### 📦 index.js

```js
const express = require("express");
const mongoose = require("mongoose");
const dotenv = require("dotenv");
const leadRoutes = require("./routes/leads");
const tokenRoutes = require("./routes/tokens");
const app = express();

dotenv.config();
app.use(express.json());

app.use("/api/ml", leadRoutes);
app.use("/api/tokens", tokenRoutes);

mongoose.connect(process.env.MONGO_URI)
  .then(() => {
    console.log("MongoDB conectado");
    app.listen(3000, () => console.log("Servidor en puerto 3000"));
  })
  .catch(console.error);
```

### 📄 models/Lead.js

```js
const mongoose = require("mongoose");
const leadSchema = new mongoose.Schema({
  leadId: { type: String, required: true, unique: true },
  itemId: String,
  buyerName: String,
  buyerEmail: String,
  buyerPhone: String,
  dateCreated: Date,
  sellerId: Number
}, { timestamps: true });
module.exports = mongoose.model("Lead", leadSchema);
```

### 📄 models/Token.js

```js
const mongoose = require("mongoose");
const tokenSchema = new mongoose.Schema({
  userId: { type: Number, required: true, unique: true },
  accessToken: { type: String, required: true },
  refreshToken: { type: String, required: true },
  expiresIn: Number,
  obtainedAt: { type: Date, default: Date.now }
}, { timestamps: true });
module.exports = mongoose.model("Token", tokenSchema);
```

### 🔧 controllers/leadsController.js

```js
const Lead = require("../models/Lead");
const Token = require("../models/Token");
const axios = require("axios");

exports.handleWebhook = async (req, res) => {
  try {
    const { topic, resource, user_id } = req.body;
    if (topic !== "lead") return res.status(200).send("No es un lead");

    const leadId = resource.split("/").pop();
    const tokenDoc = await Token.findOne({ userId: user_id });
    if (!tokenDoc) return res.status(404).send("Token no encontrado");

    const response = await axios.get(`https://api.mercadolibre.com/leads/${leadId}?access_token=${tokenDoc.accessToken}`);
    const leadData = response.data;

    const newLead = new Lead({
      leadId: leadData.id,
      itemId: leadData.item_id,
      buyerName: leadData.buyer?.first_name || "",
      buyerEmail: leadData.buyer?.email || "",
      buyerPhone: leadData.buyer?.phone?.number || "",
      dateCreated: new Date(leadData.created_at),
      sellerId: user_id
    });

    await newLead.save();
    res.status(200).send("Lead guardado");
  } catch (err) {
    console.error("Error procesando lead", err);
    res.status(500).send("Error interno");
  }
};

exports.getAllLeads = async (req, res) => {
  try {
    const leads = await Lead.find().sort({ createdAt: -1 });
    res.json(leads);
  } catch (err) {
    console.error("Error obteniendo leads", err);
    res.status(500).json({ message: "Error al obtener los leads" });
  }
};
```

### 🔧 controllers/tokensController.js

```js
const Token = require("../models/Token");
const axios = require("axios");

exports.exchangeCodeForToken = async (req, res) => {
  try {
    const { code } = req.body;
    const response = await axios.post("https://api.mercadolibre.com/oauth/token", null, {
      params: {
        grant_type: "authorization_code",
        client_id: process.env.ML_CLIENT_ID,
        client_secret: process.env.ML_CLIENT_SECRET,
        code,
        redirect_uri: process.env.ML_REDIRECT_URI
      },
      headers: { "Content-Type": "application/x-www-form-urlencoded" }
    });

    const { access_token, refresh_token, expires_in, user_id } = response.data;
    const saved = await Token.findOneAndUpdate(
      { userId: user_id },
      {
        accessToken: access_token,
        refreshToken: refresh_token,
        expiresIn: expires_in,
        obtainedAt: new Date()
      },
      { upsert: true, new: true }
    );
    res.json({ message: "Token guardado", token: saved });
  } catch (err) {
    console.error("Error intercambiando código", err);
    res.status(500).json({ message: "Error al obtener el token" });
  }
};
```

### 📍 routes/leads.js

```js
const express = require("express");
const router = express.Router();
const { handleWebhook, getAllLeads } = require("../controllers/leadsController");
router.post("/webhook", handleWebhook);
router.get("/leads", getAllLeads);
module.exports = router;
```

### 📍 routes/tokens.js

```js
const express = require("express");
const router = express.Router();
const { exchangeCodeForToken } = require("../controllers/tokensController");
router.post("/exchange", exchangeCodeForToken);
module.exports = router;
```

---

## 🧑‍💻 Frontend con Next.js

### 🔑 Login con Mercado Libre

```jsx
// pages/login-ml.js
import { useRouter } from "next/router";
import { useEffect } from "react";

export default function LoginML() {
  const router = useRouter();

  useEffect(() => {
    const code = router.query.code;
    if (code) {
      fetch("http://localhost:3000/api/tokens/exchange", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ code }),
      })
        .then((res) => res.json())
        .then(() => alert("Login exitoso"))
        .catch(() => alert("Error en login"));
    }
  }, [router.query]);

  const loginWithML = () => {
    const authUrl = `https://auth.mercadolibre.com.ar/authorization?response_type=code&client_id=${process.env.NEXT_PUBLIC_ML_CLIENT_ID}&redirect_uri=${process.env.NEXT_PUBLIC_ML_REDIRECT_URI}`;
    window.location.href = authUrl;
  };

  return (
    <button onClick={loginWithML}>Conectar con Mercado Libre</button>
  );
}
```

### 📊 Visualización de Leads

```jsx
// pages/leads.js
import { useEffect, useState } from "react";

export default function LeadsDashboard() {
  const [leads, setLeads] = useState([]);

  useEffect(() => {
    fetch("http://localhost:3000/api/ml/leads")
      .then((res) => res.json())
      .then(setLeads);
  }, []);

  return (
    <div>
      <h1>Leads</h1>
      {leads.map((lead) => (
        <div key={lead._id}>
          <p><b>{lead.buyerName}</b> - {lead.buyerEmail}</p>
        </div>
      ))}
    </div>
  );
}
```

---

## 🧪 Testing y recomendaciones

* Usar [ngrok](https://ngrok.com/) para exponer `localhost` y testear el webhook.
* Guardar `ML_CLIENT_ID`, `ML_CLIENT_SECRET` y URIs en `.env`.
* Consultar documentación oficial: [https://developers.mercadolibre.com.ar](https://developers.mercadolibre.com.ar)

---

¿Querés que te prepare un repositorio base para clonar esto todo armado? 😄
