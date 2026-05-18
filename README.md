<div align="center">

# 🍽️ jPOS — Sistema de Gestión de Punto de Venta

**Aplicación web fullstack para la gestión integral de establecimientos de hostelería**

> Proyecto de Fin de Grado — Ciclo Formativo de Grado Superior en Desarrollo de Aplicaciones Multiplataforma (2º Curso)
>
> **Autor:** Jesús Delgado Sánchez

</div>

***

## 📖 Descripción

**jPOS** es un sistema de punto de venta (TPV) completo orientado a establecimientos de hostelería (restaurantes, bares, cafeterías...). Nace de la necesidad de digitalizar la operativa de restaurantes que aún no cuentan con una solución integrada, cubriendo el flujo completo desde que el cliente llega y escanea el código QR de su mesa, hasta que el personal de cocina prepara el pedido, el camarero lo sirve y el encargado realiza el cobro y cuadra la caja al final del turno.

A diferencia de los TPV tradicionales, jPOS funciona íntegramente en el navegador y en **tiempo real** gracias a WebSockets: cualquier cambio en un pedido se refleja instantáneamente en todas las pantallas conectadas (mesas, cocina, terminal camarero, menú cliente) sin necesidad de recargar la página.

El sistema se compone de **dos proyectos independientes** que trabajan en conjunto:

- **Backend** (`jpos-backend`): API REST + servidor WebSocket desarrollado con **NestJS 11** sobre Node.js, con **Prisma ORM** como capa de acceso a la base de datos **PostgreSQL**.
- **Frontend** (`jpos-frontend`): Aplicación web SPA desarrollada con **Angular 21**, diseñada con **TailwindCSS** y comunicada con el servidor mediante HTTP y Socket.IO.

***

## ✨ Funcionalidades principales

### 👨‍💼 Panel de Control (Dashboard)
- Inicio / gestión de caja y estadísticas del turno en tiempo real
- Gestión de mesas en tiempo real (estados: INACTIVA / ACTIVA / OCUPADA / PAGANDO)
- Carta y catálogo de productos (categorías y productos bilingüe ES/EN)
- Gestión de empleados y usuarios (roles: ADMINISTRADOR, JEFE, EMPLEADO)
- Reportes y estadísticas globales con gráficos interactivos (Chart.js / ng2-charts)
- Configuración del local (nombre, NIF/CIF, IVA por defecto, moneda, idioma, horarios)

### 🍳 Visor de Cocina (KDS — Kitchen Display System)
- Pantalla independiente para el personal de cocina
- Recibe pedidos en tiempo real vía WebSocket
- El cocinero puede marcar cada línea de pedido como lista (`marcar-linea-kds`)
- Permite marcar productos como no disponibles directamente desde cocina

### 🧑‍🍳 Terminal del Camarero (TPV)
- Vista de mesas con estado actual y número de líneas pendientes
- Vista TPV con la carta completa dividida por categorías para añadir productos
- Solicitud de cuenta desde el terminal (igual que desde el menú QR del cliente)
- Recibe notificaciones sonoras cuando un plato está listo en cocina

### 📱 Menú del Cliente (acceso por código QR)
- Interfaz pública sin autenticación, accesible escaneando el QR de la mesa
- Muestra todos los productos disponibles agrupados por categorías (con nombre, descripción, precio e imagen)
- El cliente puede añadir productos al carrito, ajustar cantidades y añadir notas por línea
- Al confirmar, el pedido llega instantáneamente a cocina y terminal
- El idioma del menú se ajusta automáticamente al configurado en el panel (ES/EN)

### 💳 Gestión de Cobro y Caja
- Modal de cobro con desglose por método de pago (efectivo, tarjeta, otro); permite pagos mixtos
- Cierre de turno con cuadre de caja: muestra fondo inicial, total por tarjeta y efectivo esperado en cajón
- Registro completo de operaciones del turno con hora, número de mesa, métodos de pago e importe

***

## 🏗️ Arquitectura y tecnologías

| Capa | Tecnología | Versión | Uso en el proyecto |
|------|-----------|---------|-------------------|
| Framework backend | NestJS | 11.x | API REST, módulos, controladores, servicios e inyección de dependencias |
| ORM | Prisma | 5.x | Acceso type-safe a la base de datos, gestión de migraciones y schema |
| Base de datos | PostgreSQL | 15+ | Almacena usuarios, mesas, pedidos, carta, configuración y turnos de caja |
| Autenticación | JWT + bcrypt | — | Tokens firmados con id, email y rol; contraseñas nunca en texto plano |
| Tiempo real | Socket.IO / WebSockets | 4.x | Comunicación bidireccional: pedidos, estados de cocina y avisos de cuenta |
| Framework frontend | Angular | 21 | SPA con componentes standalone, nuevo sistema de control de flujo (`@if`, `@for`) |
| Estilos | TailwindCSS | 3.x | Interfaz construida directamente en HTML; diseño responsive incluido |
| Internacionalización | ngx-translate | 16.x | Carga archivos `es.json` / `en.json` y sustituye claves en tiempo de ejecución |
| Gráficos | Chart.js / ng2-charts | 4.x / 10.x | Gráficos interactivos (dona y líneas) en reportes e inicio del dashboard |
| Generación de QR | angularx-qrcode | 21.x | Genera el QR de cada mesa codificando la URL pública del menú del cliente |
| Programación reactiva | RxJS | 7.x | Los servicios devuelven `Observable`s; los WebSockets se consumen como streams reactivos |

***

## 🗂️ Estructura del código

```
jpos/
├── jpos-backend/                  # API REST + WebSocket (NestJS)
│   └── src/
│       ├── auth/                  # Módulo de autenticación: login, guards JWT, interceptor de tokens, cifrado bcrypt
│       ├── mesas/                 # Gestión de mesas: CRUD, estados, generación de URL para QR, apertura/cierre de sesión de mesa
│       ├── pedidos/               # Pedidos: CRUD, lógica de comandas, descuentos, cobro, cancelación y WebSocket Gateway
│       ├── carta/                 # Catálogo: CRUD de categorías y productos con soporte bilingüe (ES/EN)
│       ├── caja/                  # Turnos de caja: apertura con fondo inicial, cierre con arqueo, estadísticas del turno
│       ├── reportes/              # Informes de ventas: filtros por fecha, productos más vendidos, facturación total
│       ├── usuarios/              # Gestión de empleados: CRUD, activación/desactivación, asignación de roles
│       ├── configuracion/         # Datos del negocio: nombre, NIF, IVA, moneda, idioma, horarios
│       └── prisma/                # Servicio singleton de acceso a BD, schema completo y migraciones SQL
│
└── jpos-frontend/                 # Aplicación web SPA (Angular 21)
    └── src/app/
        ├── features/
        │   ├── auth/              # Pantalla de login con validación y opción "Recuérdame" (token 7d vs 24h)
        │   ├── cliente/           # Menú público: redirección QR y vista de carta del cliente (menú-cliente)
        │   ├── cocina/            # Visor KDS para que la cocina gestione los pedidos entrantes
        │   ├── pos/terminal/      # Terminal TPV del camarero: vista de mesas y carta para añadir productos
        │   └── dashboard/         # Panel de control: inicio (caja), mesas, carta, empleados, reportes, configuración
        ├── app/core/
        │   ├── services/          # Servicios Angular: auth, socket, caja, carta, mesa, reportes, configuración, notificaciones
        │   └── guards/            # Guards de ruta: authGuard (requiere login) y rolGuard (requiere rol mínimo)
        └── public/assets/i18n/    # Archivos de traducción: es.json (español) y en.json (inglés) para toda la interfaz
```

***

## ⚙️ Requisitos previos

### Obligatorios
- **Node.js** v18 o superior (recomendado v22 LTS)
- **npm** v9 o superior (incluido con Node.js)
- **PostgreSQL** v15 o superior, en ejecución en el puerto `5432`
- **Sistema operativo**: Windows 10/11, macOS 12+ o Linux (Ubuntu 22.04+)
- **Navegador**: Google Chrome, Mozilla Firefox o Microsoft Edge (versión reciente)
- **RAM mínima**: 4 GB (se recomiendan 8 GB para desarrollo cómodo)
- **Espacio en disco**: 400 MB libres (sin incluir `node_modules`)

### Opcionales
- **Prisma Studio**: interfaz visual de base de datos integrada en Prisma (`npx prisma studio`)
- **Postman** o **Bruno**: clientes REST para probar los endpoints de la API manualmente
- **Docker**: para ejecutar PostgreSQL en un contenedor sin instalación local
- **Dispositivo móvil en la misma red**: para simular el flujo completo del cliente escaneando el QR

### Orden de arranque (crítico)
El orden de arranque es crítico para el correcto funcionamiento del sistema:

1. ▶️ **PostgreSQL** debe estar en ejecución antes de arrancar el backend
2. ▶️ **Backend** (NestJS, puerto `3000`) debe estar activo antes que el frontend
3. ▶️ **Frontend** (Angular, puerto `4200`) conecta con el backend al iniciarse
4. 🌐 **Clientes** (navegadores) se conectan al frontend una vez está en marcha

***

## 🚀 Puesta en marcha

### 1. Clonar el repositorio

```bash
git clone https://github.com/tu-usuario/jpos.git
cd jpos
```

### 2. Configurar y arrancar el Backend

```bash
cd jpos-backend

# Instalar dependencias
npm install

# Crear el archivo de variables de entorno
cp .env.example .env
```

Edita el archivo `.env` con tus datos:

```env
DATABASE_URL="postgresql://usuario:contraseña@localhost:5432/jpos_db"
JWT_SECRET="tu_clave_secreta_muy_segura"
PORT=3000
```

```bash
# Ejecutar las migraciones de base de datos
npx prisma migrate deploy

# (Opcional) Cargar datos iniciales de ejemplo
npx prisma db seed

# Iniciar el servidor
npm run start:dev
```

El backend estará disponible en `http://localhost:3000`.

### 3. Configurar y arrancar el Frontend

```bash
cd ../jpos-frontend

# Instalar dependencias
npm install

# Iniciar la aplicación
ng serve
# o bien:
npm start
```

El frontend estará disponible en `http://localhost:4200`.

### 4. Acceso inicial

Abre `http://localhost:4200` en tu navegador. Usa las credenciales del usuario administrador creado por el seed:

| Campo | Valor |
|-------|-------|
| Email | `admin@jpos.com` |
| Contraseña | `admin123` |
| Rol | ADMINISTRADOR |

***

## 🔐 Roles y permisos

| Sección | ADMINISTRADOR | JEFE | EMPLEADO |
|---------|:---:|:---:|:---:|
| Dashboard completo | ✅ | — | — |
| Reportes y estadísticas | ✅ | — | — |
| Configuración del local | ✅ | — | — |
| Gestión de empleados | ✅ | — | — |
| Gestión de carta | ✅ | ✅ | — |
| Inicio de caja | ✅ | ✅ | ✅ |
| Gestión de mesas | ✅ | ✅ | ✅ |
| Descuentos y cancelaciones | ✅ | ✅ | — |
| Cobro de pedidos | ✅ | ✅ | ✅ |
| Terminal camarero (TPV) | ✅ | ✅ | ✅ |
| Visor de cocina (KDS) | ✅ | ✅ | ✅ |

***

## ⚡ Eventos WebSocket principales

| Evento | Dirección | Descripción |
|--------|-----------|-------------|
| `enviar-comanda` | Cliente → Servidor | El camarero/cliente envía el pedido confirmado |
| `nuevo-pedido-cocina` | Servidor → Cocina | Llega un nuevo pedido al KDS de cocina |
| `pedido-actualizado` | Servidor → Todos | Se ha modificado un pedido (producto añadido, descuento, etc.) |
| `mesas-actualizadas` | Servidor → Todos | Cambio de estado en alguna mesa |
| `marcar-linea-kds` | Cocina → Servidor | La cocina marca una línea como lista |
| `plato-listo-notificacion` | Servidor → Terminal | Notificación sonora: el plato está listo para servir |
| `solicitar-cuenta` | Cliente/Terminal → Servidor | El cliente o el camarero solicita la cuenta |
| `cuenta-solicitada` | Servidor → Dashboard | Aviso visual en la tarjeta de la mesa correspondiente |
| `cuenta-cerrada` | Servidor → Todos | Se ha cobrado y cerrado la cuenta; la mesa queda libre |

***

## 🗺️ Posibles ampliaciones

- **Pedidos a domicilio y recogida en local**: módulo de pedidos online donde el cliente se registra con sus datos, con nuevo rol de "Repartidor" e integración con pasarela de pago para cobro previo.
- **Impresión de tickets térmicos**: integración con impresoras térmicas vía protocolo ESC/POS para imprimir comandas automáticamente en cocina y tickets de cobro en caja.
- **App móvil nativa (iOS/Android)**: desarrollar las interfaces del camarero, repartidor y cliente como app autónoma con Ionic/Capacitor, aprovechando el backend existente sin cambios.
- **Gestión de reservas**: módulo de reservas con calendario, control de aforo y notificaciones automáticas por correo o SMS al cliente.
- **Múltiples métodos de pago digitales**: integración con pasarelas como Stripe o Redsys para permitir el pago con tarjeta directamente desde el móvil del cliente, sin datafono físico.
- **Estadísticas avanzadas**: análisis de productos más vendidos por categoría y franja horaria, horas pico, ticket medio por zona o mesa.
- **Modo offline y PWA**: convertir el frontend en una Progressive Web App con Service Worker para que siga funcionando durante cortes de red puntuales y sincronice cuando se recupere la conexión.
- **Gestión de inventario y proveedores**: control de stock de ingredientes, alertas de escasez y generación automática de pedidos a proveedores.
- **Multi-restaurante**: soporte para gestionar varias sucursales desde una misma instancia del sistema, con configuraciones y personal independientes por local.
- **Fidelización de clientes**: sistema de puntos o tarjeta de fidelización integrado en el menú del cliente, con historial de visitas y descuentos personalizados.

***

## 💬 Valoración personal

> *"La idea de desarrollar jPOS viene de una experiencia directa: durante años he trabajado en distintos restaurantes donde la digitalización del servicio era parcial o inexistente. Ver a diario cómo los camareros tomaban comandas en papel, cómo la cocina recibía tickets escritos a mano y cómo los errores o los retrasos en la comunicación entre sala y cocina eran una fuente constante de estrés y pérdida de tiempo. Esa experiencia me hizo preguntarme si, con los conocimientos adquiridos durante el ciclo, podría hacer algo que resolviera esos problemas de verdad. Así nació jPOS: como un reto personal."*

***

## 📄 Licencia

Este proyecto ha sido desarrollado como Proyecto de Fin de Grado del Ciclo Formativo de Grado Superior en Desarrollo de Aplicaciones Multiplataforma. Todos los derechos reservados © Jesús DS.

***

<div align="center">
  <sub>Desarrollado con ❤️ por <strong>Jesús DS</strong></sub>
</div>
