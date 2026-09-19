# 📖 Documentación del Proyecto: Menú Cafés Coyo

## 📌 Descripción General

**Menú Cafés Coyo** es una aplicación web sencilla diseñada para que los clientes de una cafetería puedan visualizar el menú y las promociones vigentes. El sistema incluye un panel de administración exclusivo para el personal autorizado, desde donde se gestionan los artículos del menú y las promociones mediante operaciones CRUD (Crear, Leer, Actualizar, Eliminar).

El proyecto está desarrollado con un enfoque ligero: el backend sirve tanto la vista pública como el panel de administración, y la persistencia de datos se realiza mediante archivos JSON, lo que facilita su despliegue y mantenimiento sin necesidad de una base de datos compleja.

---

## 🛠️ Tecnologías Utilizadas

Las tecnologías empleadas en el proyecto son las siguientes:

| Capa | Tecnología |
|------|------------|
| **Frontend** | HTML5, CSS3, JavaScript (Vanilla) |
| **Backend** | Node.js, Express |
| **Persistencia** | Archivos JSON |
| **Autenticación** | OTP (One-Time Password) vía correo electrónico con **NodeMailer** |

---

## 📂 Estructura del Proyecto

```text
Menu/
├── backend/
│   ├── data/
│   │   ├── menu.json          # Datos del menú (categorías y artículos)
│   │   └── promos.json        # Datos de las promociones
│   └── server.js              # Servidor Express y lógica de la API
├── frontend/
│   ├── admin/                 # Panel de administración
│   │   ├── admin.css
│   │   ├── admin.js
│   │   └── index.html
│   └── public/                # Vista pública para clientes
│       ├── app.js
│       ├── index.html
│       └── styles.css
├── node_modules/
├── package-lock.json
├── package.json
└── README.md
```

---

## 🚀 Instalación y Configuración

### Requisitos previos

- **Node.js** (versión 18 o superior recomendada)
- **npm** (incluido con Node.js)

### Pasos de instalación

1. **Clonar el repositorio:**

   ```bash
   git clone https://github.com/EnriqueJeronimo/Menu.git
   cd Menu
   ```

2. **Instalar dependencias:**

   ```bash
   npm install
   ```

3. **Configurar credenciales de correo (opcional):**

   El sistema de autenticación utiliza un servicio SMTP de Ethereal para enviar códigos OTP. Las credenciales están definidas directamente en `backend/server.js`. Para producción, se recomienda reemplazarlas por variables de entorno.

4. **Iniciar el servidor:**

   ```bash
   node backend/server.js
   ```

   El servidor se ejecutará en `http://localhost:3000` por defecto. Puedes cambiar el puerto mediante la variable de entorno `PORT`.

---

## 🧭 Guía de Uso

### 👤 Vista Cliente (Pública)

Accede a `http://localhost:3000/` para visualizar el menú y las promociones activas. La página carga dinámicamente los datos desde la API y los renderiza en una cuadrícula responsive.

**Características:**

- Muestra las promociones activas en la parte superior (si existen).
- Lista las categorías del menú con sus respectivos artículos y precios.
- Diseño adaptable (1 columna en móvil, 2 en escritorio).

### 🔐 Panel de Administración

Accede a `http://localhost:3000/admin` para gestionar el contenido.

**Flujo de autenticación:**

1. **Solicitar código:** El administrador ingresa su correo electrónico autorizado y solicita un código OTP.
2. **Recibir código:** El sistema envía un código de 6 dígitos al correo configurado (válido por 10 minutos).
3. **Verificar código:** El administrador ingresa el código recibido. Si es correcto, obtiene un token de sesión que se almacena en memoria.
4. **Acceder al panel:** Una vez autenticado, se muestra el dashboard con las secciones de gestión.

**Funcionalidades del dashboard:**

- **Gestión de Menú:** Editar nombres y precios de artículos existentes, agregar nuevos artículos a cualquier categoría, y eliminar artículos. Los cambios se guardan en el servidor mediante una petición `PUT /api/menu` con el token de autorización.
- **Gestión de Promociones:** Crear nuevas promociones (título y descripción), listar las promociones existentes, y eliminarlas mediante una petición `DELETE /api/promos/:id` protegida por token.

---

## 🔌 API Endpoints

El backend expone los siguientes endpoints RESTful:

| Método | Ruta | Autenticación | Descripción |
|--------|------|---------------|-------------|
| `GET` | `/api/menu` | No | Obtiene el menú completo (categorías y artículos). |
| `PUT` | `/api/menu` | **Sí** (Token) | Actualiza el menú completo. |
| `GET` | `/api/promos` | No | Obtiene todas las promociones (activas e inactivas). |
| `POST` | `/api/promos` | **Sí** (Token) | Crea una nueva promoción. |
| `DELETE` | `/api/promos/:id` | **Sí** (Token) | Elimina una promoción por su ID. |
| `POST` | `/api/auth/login` | No | Solicita el envío del código OTP al correo autorizado. |
| `POST` | `/api/auth/verify` | No | Verifica el código OTP y devuelve un token de sesión. |

---

## 🗃️ Estructura de Datos

### `menu.json`

El archivo contiene un objeto con una clave `categorias`, que es un arreglo de categorías. Cada categoría tiene un `id`, un `nombre` y un arreglo de `articulos`. Cada artículo tiene `id`, `nombre` y `precio`.

```json
{
  "categorias": [
    {
      "id": "bebidas-frias",
      "nombre": "Bebidas Frías",
      "articulos": [
        { "id": "1", "nombre": "Frapuccino", "precio": 60 },
        { "id": "2", "nombre": "Frappe Mocca", "precio": 65 }
      ]
    }
  ]
}
```

### `promos.json`

Es un arreglo de objetos. Cada promoción tiene `id`, `titulo`, `descripcion` y un campo booleano `activa`.

```json
[
  {
    "id": "p2",
    "titulo": "¡Inicia tu semana santa con todo!",
    "descripcion": "En la compra de un frappe, llévate un wafle gratis!.",
    "activa": true
  }
]
```

---

## 🔐 Sistema de Autenticación

El sistema utiliza autenticación basada en **OTP (One-Time Password)** enviado por correo electrónico:

1. El administrador envía su correo a `POST /api/auth/login`.
2. El backend genera un código aleatorio de 6 dígitos, lo almacena en memoria con una expiración de 10 minutos, y lo envía mediante NodeMailer.
3. El administrador envía el código a `POST /api/auth/verify`.
4. Si el código es válido y no ha expirado, el backend genera un token de sesión simple y lo almacena en un `Set` en memoria. Este token debe incluirse en el encabezado `Authorization: Bearer <token>` para las rutas protegidas.

> **Nota:** El correo autorizado está hardcodeado como `letitia61@ethereal.email`. En un entorno de producción, esta configuración debe parametrizarse y utilizar credenciales SMTP reales.

---

## 📄 Licencia

El proyecto está bajo la licencia **ISC**, según se indica en el archivo `package.json`. El autor es **EnriqueJeronimo_Dev** (Enrique Jerónimo).

---

## 📌 Notas Adicionales

- El proyecto se encuentra en versión **1.5 beta**.
- La rama principal de desarrollo es `develop`.
- El servidor sirve la vista pública desde `frontend/public` y el panel de administración desde `frontend/admin` mediante `express.static`.

---
