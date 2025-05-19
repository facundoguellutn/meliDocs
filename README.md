# 🚗📦 Leads de Mercado Libre - Tutoriales de Integración

Este repositorio contiene una serie de tutoriales creados por **Facu** con ayuda de **ChatGPT** y la **documentación oficial de Mercado Libre**, con el objetivo de aprender e implementar cómo recibir **leads** (consultas de usuarios) de publicaciones en categorías como **autos** e **inmuebles** usando la API de Mercado Libre.

---

## 📚 Contenido del repositorio

En este repo vas a encontrar **tres archivos principales**, cada uno con un enfoque distinto:

### 1. `introMeli.md`
Un archivo introductorio que te brinda un **vistazo rápido** y simplificado sobre cómo funciona el proceso de autenticación y recepción de leads desde Mercado Libre. Ideal para entender el concepto general antes de meterte en el código.

---

### 2. `mercado-libre-leads-tutorial-completo.md`
Una guía **técnica y completa** con implementación paso a paso de todo el proceso:

- Backend con **Express** y **MongoDB**
- Frontend con **Next.js/React**
- Uso de **OAuth2** para obtener tokens
- Recepción de leads vía **webhook**
- Visualización de leads desde un panel de control

Este archivo es ideal si querés **armar una app real** que reciba leads de Mercado Libre.

---

### 3. `flujo-ejemplo-leads-ml.md`
Explica el **flujo completo** implementado con un **ejemplo realista** paso a paso:

- Redirección de login
- Obtención y almacenamiento del token
- Recepción de leads
- Consulta y guardado de la información del comprador
- Visualización de los datos desde el frontend

Perfecto para entender cómo se conectan todas las piezas entre sí.

---

## 🛠 Tecnologías utilizadas

- Node.js + Express
- MongoDB
- React / Next.js
- API de Mercado Libre (OAuth, Webhooks, Leads)

---

## 📎 Recursos oficiales

- [Documentación de Mercado Libre](https://developers.mercadolibre.com.ar)
- [API de autenticación y autorización](https://developers.mercadolibre.com.ar/es_ar/autenticacion-y-autorizacion)
- [Notificaciones de productos y leads](https://developers.mercadolibre.com.ar/es_ar/productos-recibe-notificaciones)

---

## ✍️ Autor

Tutoriales creados por **Facu**, desarrollador fullstack, como parte de una investigación técnica y práctica para integrar sistemas externos con Mercado Libre.

---

> Si este repo te sirvió, ¡no olvides dejar una estrella ⭐ y compartirlo con otros desarrolladores!
