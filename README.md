# 🍽️ Delimenu × Maison — Menú Digital Interactivo

[![Next.js](https://img.shields.io/badge/Next.js-16.3-black?logo=next.js)](https://nextjs.org/)
[![React](https://img.shields.io/badge/React-19.2-61dafb?logo=react)](https://react.dev/)
[![Tailwind CSS](https://img.shields.io/badge/Tailwind_CSS-v4-38bdf8?logo=tailwindcss)](https://tailwindcss.com/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.0-3178c6?logo=typescript)](https://www.typescriptlang.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

Un menú web de restaurante moderno con diseño editorial, construido sobre **Next.js** y conectado en tiempo real a la [API pública de Delimenu](https://delimenu.co/developers).

El diseño replica la experiencia de la aclamada plantilla [Maison](https://maisonmenu.framer.website/menu): maquetación en dos columnas, cuadrícula interactiva de platillos con efecto hover para descripciones, filtrado instantáneo por categorías con query params compartibles y wordmark responsivo adaptado al ancho de pantalla.

---

## 📌 Tabla de Contenidos

- [Características Principales](#-características-principales)
- [¿Cómo Funciona?](#-cómo-funciona)
- [Mejoras y Cambios Realizados](#-mejoras-y-cambios-realizados)
- [Requisitos Previos](#-requisitos-previos)
- [Instalación y Configuración](#-instalación-y-configuración)
- [Cómo Iniciar el Servidor](#-cómo-iniciar-el-servidor)
- [Estructura del Proyecto](#-estructura-del-proyecto)
- [Despliegue Continuo (CI/CD en Netlify)](#-despliegue-continuo-cicd-en-netlify)
- [Repositorio Original y Créditos](#-repositorio-original-y-créditos)
- [Licencia](#-licencia)

---

## ✨ Características Principales

* **Consumo de API sin fricción:** Lee el menú publicado sin requerir API keys privadas ni configuraciones complejas.
* **ISR (Incremental Static Regeneration):** Se revalida en segundo plano cada 5 minutos, garantizando carga ultrarrápida y datos siempre frescos.
* **Diseño Editorial Responsivo:** Optimizado tanto para pantallas móviles como para monitores de escritorio con animaciones suaves.
* **Filtros por Categoría Compartibles:** La URL guarda el estado de la categoría seleccionada (`?categoria=postres`) para que los clientes puedan compartir secciones específicas.
* **Reserva Directa vía WhatsApp:** Genera enlaces automáticos con mensaje preconfigurado al número del restaurante.
* **Lanzador Ejecutable para Windows:** Incluye un ejecutable nativo `.exe` para iniciar el servidor y abrir el navegador con un solo clic.

---

## ⚙️ ¿Cómo Funciona?

Toda la lógica de conexión con Delimenu reside en [`lib/delimenu.ts`](lib/delimenu.ts). La aplicación realiza una consulta al endpoint público:

```ts
const res = await fetch(`https://delimenu.co/api/v1/menus/${identifier}`, {
  next: { revalidate: 300 }, // Caché de 5 minutos
})
const { restaurant, categories } = await res.json()
```

### Puntos clave del backend y datos:
1. **Público y Seguro:** No expone credenciales. La lectura de menús publicados con plan activo es pública; las API Keys solo se emplean en operaciones administrativas desde el servidor.
2. **Precios y Moneda Local:** Los precios son servidos en unidades mayores (`6.25` para USD o `12500` para COP). Se utiliza la API nativa de internacionalización (`Intl.NumberFormat`) según la moneda del restaurante.
3. **Manejo de Estados de Error:** Si el slug no existe responde `404 NOT_FOUND` y si el plan está inactivo devuelve `403 RESTAURANT_UNAVAILABLE`. En ambos casos, el componente `MenuUnavailable` despliega una pantalla amigable de aviso.
4. **Optimización de Imágenes:** Delimenu entrega imágenes ya escaladas (1080×1080 para productos y 512×512 para logos) desde CDN, por lo que la optimización interna de Next.js está desactivada para reducir consumo de recursos.

---

## 🛠️ Mejoras y Cambios Realizados

Respecto al repositorio base original, se incorporaron las siguientes mejoras para simplificar su uso y despliegue:

### 1. Lanzador Nativo Windows (`iniciar-servidor.exe`)
* Se desarrolló y compiló un ejecutable nativo en C# (`iniciar-servidor.exe`) utilizando las herramientas nativas de Windows (`csc.exe`).
* **Automatización completa:** Al hacer doble clic, detecta si el usuario cuenta con `pnpm` o `npm`, levanta el servidor local (`localhost:3000`), mantiene los logs visibles en consola y abre el navegador por defecto automáticamente.
* Se incluye también la versión script tradicional en [`iniciar-servidor.bat`](iniciar-servidor.bat) y el código fuente en [`iniciar-servidor.cs`](iniciar-servidor.cs).

### 2. Optimización Integral de Dependencias (`package.json`)
* **Migración de estilos a producción:** Se trasladaron `@tailwindcss/postcss` y `tailwindcss` a `dependencies` principales. Esto previene fallos visuales o errores de compilación al instalar en servidores o contenedores con `npm install --omit=dev`.
* **Rangos Semánticos Flexibles:** Se añadieron rangos `^` para evitar bloqueos por parches menores entre React 19 y Next.js 16.
* **Compatibilidad Universal:** Se eliminó la restricción estricta de `packageManager`, permitiendo instalar y correr el proyecto fluidamente tanto con `npm`, `pnpm`, `yarn` o `bun`.
* **Compatibilidad de Motor:** Declaración de `engines` (`node >= 20.9.0`) para prevenir incompatibilidades con versiones antiguas de Node.js.

### 3. Licenciamiento y Documentación
* Adición del archivo formal [LICENSE](LICENSE) bajo términos MIT respetando los derechos de autor originales.

---

## 📋 Requisitos Previos

* **[Node.js](https://nodejs.org/):** Versión `20.9.0` o superior instalada.
* **Gestor de paquetes:** Puedes usar `npm` (incluido con Node), `pnpm`, `yarn` o `bun`.

---

## 🚀 Instalación y Configuración

### 1. Clonar o descargar el repositorio
```bash
git clone https://github.com/JuandaGarcia/delimenu-maison-example.git
cd delimenu-maison-example
```

### 2. Instalar las dependencias
Ejecuta con tu gestor favorito:

```bash
# Con npm
npm install

# O con pnpm (recomendado)
pnpm install
```

### 3. Configurar otro menú (Opcional)
Por defecto, el proyecto carga el menú de demostración (`demo` — *Delicias Urbanas*). Para usar tu propio menú:

1. Crea un archivo `.env.local` copiando el ejemplo:
   ```bash
   cp .env.example .env.local
   ```
2. Modifica la variable con el identificador (slug) de tu restaurante en Delimenu:
   ```env
   DELIMENU_MENU=tu-restaurante
   ```

---

## 💻 Cómo Iniciar el Servidor

Tienes tres formas sencillas de correr la aplicación:

### Opción A — Con el Ejecutable (Recomendado en Windows)
Haz doble clic sobre el archivo **`iniciar-servidor.exe`** en la carpeta del proyecto. 
El servidor iniciará y tu navegador abrirá `http://localhost:3000` automáticamente.

### Opción B — Con el archivo Batch
Haz doble clic sobre **`iniciar-servidor.bat`**.

### Opción C — Desde la Terminal
```bash
# Modo desarrollo
npm run dev
# o
pnpm dev
```
Abre tu navegador en [http://localhost:3000](http://localhost:3000) (o directamente en `/menu`).

### Para Compilar y Correr en Producción:
```bash
npm run build
npm start
```

---

## 📂 Estructura del Proyecto

```text
├── app/
│   ├── layout.tsx              # Tipografía Mouse Memoirs y estructura HTML base
│   ├── page.tsx                # Redirección a /menu
│   ├── globals.css             # Configuración de Tailwind CSS v4, animaciones y tokens
│   └── menu/
│       └── page.tsx            # Server Component principal: obtiene y renderiza el menú
├── components/
│   ├── BackToTop.tsx           # Botón flotante para volver arriba
│   ├── Favourites.tsx          # Ticker animado de platos recomendados/promociones
│   ├── Footer.tsx              # Pie de página con créditos e información de contacto
│   ├── MenuBrowser.tsx         # Selector de categorías y grid filtrado de platos
│   ├── MenuUnavailable.tsx     # Vista de cortesía para menús caídos o inexistentes
│   ├── ProductCard.tsx         # Tarjeta de producto con foto, precio y descripción hover
│   ├── Reveal.tsx              # Contenedor de transición de entrada progresiva
│   ├── ScrollPane.tsx          # Panel derecho con scroll independiente y barra de progreso
│   ├── Sidebar.tsx             # Panel izquierdo con logo, navegación y llamado a reserva
│   └── Wordmark.tsx            # Tipografía responsiva de gran formato adaptada al contenedor
├── .github/
│   └── workflows/
│       └── deploy-netlify.yml  # Workflow de CI/CD para GitHub Actions y Netlify
├── docs/
│   └── CI_CD_NETLIFY.md        # Documentación línea por línea del CI/CD
├── lib/
│   └── delimenu.ts             # Cliente de la API de Delimenu, tipos TypeScript y utilidades
├── iniciar-servidor.exe        # Lanzador nativo Windows de un solo clic
├── iniciar-servidor.bat        # Lanzador batch para Windows
├── iniciar-servidor.cs         # Código fuente en C# del lanzador
├── netlify.toml                # Configuración de compilación y plugin Next.js en Netlify
├── package.json                # Dependencias y scripts del proyecto
└── README.md                   # Documentación del proyecto
```

---

## 🌐 Despliegue Continuo (CI/CD en Netlify)

El proyecto cuenta con un flujo automatizado de CI/CD mediante **GitHub Actions** configurado en [`.github/workflows/deploy-netlify.yml`](.github/workflows/deploy-netlify.yml):

* **Fase CI (Integración Continua)**: Verifica el código con ESLint (`npm run lint`) y compila la aplicación (`npm run build`) en cada commit o Pull Request.
* **Fase CD (Despliegue Continuo)**:
  * Genera un **Deploy Preview** en cada Pull Request para validar visualmente los cambios antes de fusionar.
  * Publica automáticamente a **Producción** cuando se hace push a la rama `main` o mediante ejecución manual (`workflow_dispatch`).

Para ver la explicación detallada línea por línea de cada instrucción del workflow y la configuración de secretos (`NETLIFY_AUTH_TOKEN`, `NETLIFY_SITE_ID`), consulta la [Guía completa de CI/CD para Netlify](docs/CI_CD_NETLIFY.md).

---

## 🤝 Repositorio Original y Créditos

Este proyecto es una versión mejorada basada en el trabajo original de:

* **Repositorio Base:** [JuandaGarcia/delimenu-maison-example](https://github.com/JuandaGarcia/delimenu-maison-example) por [Juan David García (@JuandaGarcia)](https://github.com/JuandaGarcia).
* **API de Menús:** [Delimenu](https://delimenu.co) — Consulta su [portal de desarrolladores](https://delimenu.co/developers).
* **Inspiración de Diseño:** Plantilla [Maison](https://maisonmenu.framer.website/menu) desarrollada en Framer. Las fotografías, marcas y textos mostrados pertenecen al menú demo de Delimenu.

---

## 📄 Licencia

Este proyecto se distribuye bajo la licencia **MIT**. Consulta el archivo [LICENSE](LICENSE) para más detalles.
