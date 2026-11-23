# 🚀 Documentación del Flujo CI/CD para Netlify (GitHub Actions)

Este documento detalla la integración continua (CI) y despliegue continuo (CD) configurado en el archivo [`.github/workflows/deploy-netlify.yml`](../.github/workflows/deploy-netlify.yml) para el proyecto **Delimenu × Maison** en **Netlify**.

---

## 📋 Requisitos Previos: Secretos en GitHub

Para que GitHub Actions pueda autenticarse y desplegar en tu cuenta de Netlify, debes configurar los siguientes secretos en tu repositorio de GitHub (**Settings** > **Secrets and variables** > **Actions** > **New repository secret**):

| Secreto | Descripción | Dónde obtenerlo |
| :--- | :--- | :--- |
| `NETLIFY_AUTH_TOKEN` | Token de acceso personal de Netlify | En Netlify: *User Settings* > *Applications* > *OAuth Apps / Personal access tokens* > *New access token*. |
| `NETLIFY_SITE_ID` | Identificador único del sitio (*API ID*) | En Netlify: Selecciona tu sitio > *Site configuration* > *General* > *Site details* > *API ID*. |
| `DELIMENU_MENU` *(Opcional)* | Identificador (slug) de tu restaurante | Tu slug en Delimenu (si no se define, se usa `'demo'`). |
| `DELIMENU_API_URL` *(Opcional)* | URL base de la API | Por defecto `'https://delimenu.co/api/v1'`. |

---

## 🔍 Explicación Línea por Línea del Workflow

A continuación se desglosa cada bloque y directiva del archivo YAML:

```yaml
1: name: CI/CD - Deploy to Netlify
```
* **Línea 1**: `name` asigna un título identificativo visible en la pestaña **Actions** de GitHub.

```yaml
3: on:
4:   push:
5:     branches:
6:       - main
7:   pull_request:
8:     branches:
9:       - main
10:  workflow_dispatch:
```
* **Líneas 3–10**: Define los disparadores (*triggers*) que inician la ejecución:
  * `push.branches: [main]`: Se ejecuta automáticamente cada vez que se suben cambios directamente a la rama `main`.
  * `pull_request.branches: [main]`: Se ejecuta cuando se crea o actualiza un Pull Request hacia `main`, permitiendo validar que el código compila y generar un entorno de prueba (*Preview*).
  * `workflow_dispatch`: Habilita un botón en la interfaz de GitHub para ejecutar el flujo manualmente en cualquier momento.

```yaml
12: jobs:
13:   ci-cd:
14:     name: Integración Continua y Despliegue en Netlify
15:     runs-on: ubuntu-latest
```
* **Líneas 12–15**:
  * `jobs`: Agrupa los trabajos a realizar.
  * `ci-cd`: Identificador único del trabajo.
  * `name`: Nombre descriptivo mostrado en la interfaz gráfica.
  * `runs-on: ubuntu-latest`: Ejecuta el trabajo en una máquina virtual Linux ligera y actualizada provista por GitHub.

```yaml
17:     steps:
18:       - name: 1. Descargar el código fuente del repositorio
19:         uses: actions/checkout@v4
```
* **Líneas 17–19**: Utiliza la acción oficial `actions/checkout@v4` para clonar el código fuente del repositorio dentro de la máquina virtual del runner.

```yaml
21:       - name: 2. Configurar entorno de Node.js
22:         uses: actions/setup-node@v4
23:         with:
24:           node-version: 20
25:           cache: 'npm'
```
* **Líneas 21–25**:
  * `uses: actions/setup-node@v4`: Instala Node.js en el runner.
  * `node-version: 20`: Fija la versión de Node en 20 (cumpliendo el requisito `>=20.9.0` de `package.json`).
  * `cache: 'npm'`: Activa la caché automática del directorio global de npm basada en `package-lock.json`, reduciendo significativamente los tiempos de instalación en futuras ejecuciones.

```yaml
27:       - name: 3. Instalar dependencias limpias (npm ci)
28:         run: npm ci
```
* **Líneas 27–28**: Ejecuta `npm ci` (*Clean Install*). A diferencia de `npm install`, garantiza una instalación exacta y reproducible basada estrictamente en `package-lock.json`, sin modificarlo ni resolver versiones ambiguas.

```yaml
30:       - name: 4. Ejecutar validación de código estático (Linter)
31:         run: npm run lint
```
* **Líneas 30–31**: Fase de **CI**. Ejecuta ESLint sobre el código. Si existen violaciones o errores graves de sintaxis/reglas de Next.js, el workflow se detiene inmediatamente antes de intentar construir o desplegar.

```yaml
33:       - name: 5. Compilar proyecto Next.js (Build)
34:         env:
35:           DELIMENU_API_URL: ${{ secrets.DELIMENU_API_URL || 'https://delimenu.co/api/v1' }}
36:           DELIMENU_MENU: ${{ secrets.DELIMENU_MENU || 'demo' }}
37:         run: npm run build
```
* **Líneas 33–37**: Fase de **CI / Build**.
  * `env`: Inyecta las variables de entorno necesarias para la compilación estática e ISR de Next.js (leyendo de los secretos de GitHub o recurriendo a los valores por defecto).
  * `run: npm run build`: Compila la aplicación Next.js y genera la salida optimizada en `.next/`.

```yaml
39:       - name: 6. Despliegue de Preview en Netlify (Pull Requests)
40:         if: github.event_name == 'pull_request'
41:         env:
42:           NETLIFY_AUTH_TOKEN: ${{ secrets.NETLIFY_AUTH_TOKEN }}
43:           NETLIFY_SITE_ID: ${{ secrets.NETLIFY_SITE_ID }}
44:         run: |
45:           npx netlify-cli deploy \
46:             --site "$NETLIFY_SITE_ID" \
47:             --auth "$NETLIFY_AUTH_TOKEN" \
48:             --message "Deploy Preview PR #${{ github.event.pull_request.number }} [${{ github.sha }}]"
```
* **Líneas 39–48**: Despliegue de **Previsualización (CD)**:
  * `if: github.event_name == 'pull_request'`: Esta instrucción condicional asegura que el paso SOLO se ejecute cuando proviene de un Pull Request.
  * `env`: Provee el token de autenticación y el identificador del sitio.
  * `npx netlify-cli deploy`: Sube los artefactos compilados a Netlify sin marcar el despliegue como producción, generando una URL única de previsualización para revisar los cambios antes del *merge*.

```yaml
50:       - name: 7. Despliegue a Producción en Netlify (Push a main o manual)
51:         if: (github.event_name == 'push' && github.ref == 'refs/heads/main') || github.event_name == 'workflow_dispatch'
52:         env:
53:           NETLIFY_AUTH_TOKEN: ${{ secrets.NETLIFY_AUTH_TOKEN }}
54:           NETLIFY_SITE_ID: ${{ secrets.NETLIFY_SITE_ID }}
55:         run: |
56:           npx netlify-cli deploy \
57:             --prod \
58:             --site "$NETLIFY_SITE_ID" \
59:             --auth "$NETLIFY_AUTH_TOKEN" \
60:             --message "Deploy Producción commit ${{ github.sha }}"
```
* **Líneas 50–60**: Despliegue a **Producción (CD)**:
  * `if: ...`: Se ejecuta únicamente cuando el evento es un `push` en la rama `main` o se lanzó manualmente (`workflow_dispatch`).
  * `--prod`: Bandera crucial que indica a Netlify que este despliegue debe publicarse en el dominio principal de producción.
  * `--message`: Registra un mensaje descriptivo en el historial de despliegues de Netlify con el hash del commit (`github.sha`).

---

## ⚙️ Archivo de Configuración [`netlify.toml`](../netlify.toml)

Para asegurar que Netlify procese correctamente las funciones del App Router de Next.js y el almacenamiento en caché de ISR, se creó el archivo `netlify.toml`:

```toml
[build]
  command = "npm run build"
  publish = ".next"

[build.environment]
  NODE_VERSION = "20.18.0"

[[plugins]]
  package = "@netlify/plugin-nextjs"
```

* `[build]`: Define el comando de compilación estándar y la carpeta de publicación `.next`.
* `[build.environment]`: Asegura que el entorno de Netlify use Node.js 20.
* `[[plugins]]`: Registra `@netlify/plugin-nextjs`, el adaptador oficial que convierte las páginas dinámicas, rutas e ISR en Netlify Functions y Edge Handlers automáticamente.
