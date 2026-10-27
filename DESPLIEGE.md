
# Guía Completa de Despliegue de Angular Pokedex en Azure App Service

  

Documentación técnica detallada del procedimiento de despliegue de la

aplicación Angular Pokedex en Azure App Service (Linux), cubriendo desde

el método manual hasta la automatización CI/CD con GitHub Actions y la

inyección de cabeceras de seguridad.

  

------------------------------------------------------------------------

  

# PARTE 1: Despliegue Manual en Azure App Service

  

Esta primera fase explica cómo preparar los archivos de producción

localmente y subirlos manualmente al entorno de hosting de Azure.

  

## 1. Paso 1: Creación del App Service en Azure Portal

  

1. Ingresa a [portal.azure.net](https://portal.azure.net/).

2. Haz clic en **Crear un recurso**  \>  **Web App** (o busca **App

Services** en la barra superior y presiona **Crear**).

3. Configura los parámetros básicos:

-  **Suscripción:** Selecciona tu suscripción (*Azure for Students*

o equivalente).

-  **Grupo de recursos:** Crea uno nuevo o selecciona uno existente

(ej. `rg-pokedex`).

-  **Nombre de la App:** Asigna un nombre único global (ej.

`app-pokedex-web-prod`).

-  **Publicar:** Selecciona **Código** (*Code*).

-  **Pila de runtime:** Selecciona **Node 18 LTS** (o versión

acorde a tu Angular).

-  **Sistema operativo:**  **Linux**.

-  **Región:** Selecciona la que tengas disponoble para tu cuenta de estudiante.

4. Haz clic en **Revisar y crear** y luego en **Crear**.

  

------------------------------------------------------------------------

  

## 2. Paso 2: Generación del Build (`dist`) Localmente

  

En la terminal de tu proyecto Angular local, ejecuta el comando de

compilación para producción:

  

``` bash

ng  build

```

  

Este comando procesa el proyecto y crea la carpeta con los archivos

estáticos en la ruta `dist/pokedex-angular`.

  

> **Nota:** La carpeta compilada contendrá los archivos optimizados

> (`index.html`, `main.js`, `styles.css`, etc.).

  

## 3. Paso 3: Carga de Archivos mediante Herramientas Avanzadas (Kudu / File Manager)

  

1. En Azure Portal, ve a tu **App Service** creado

(`app-pokedex-web-prod`).

  

2. En el menú lateral izquierdo, busca la sección **Herramientas de

desarrollo** (*Development Tools*) y haz clic en **Herramientas

avanzadas** (*Advanced Tools* / *Kudu*).

  

3. Haz clic en **Ir** (*Go*). Se abrirá una nueva pestaña en el

navegador con la consola de Kudu.

  

4. En el menú superior de Kudu, selecciona **Tools**  \>  **Zip Deploy**

o abre el gestor de archivos / SSH.

  

5. Navega a la ruta de destino dentro del servidor de Azure:

  

``` text

/home/site/wwwroot/

```

  

6. Arrastra y copia los archivos de tu compilación local dentro de esta

carpeta. Si subiste la carpeta completa `pokedex-angular`, los

archivos quedarán ubicados en `/home/site/wwwroot/pokedex-angular`.

## 4. Paso 4: Configurar los Headers de Seguridad (`serve.json`)

  

Para proteger la aplicación contra vulnerabilidades web (como XSS o

Clickjacking) e indicar las reglas de enrutamiento SPA, debemos incluir

un archivo de configuración llamado **`serve.json`**.

  

Crea el archivo `serve.json` dentro de la carpeta compilada de tu

proyecto antes de servirla o súbelo a la raíz del repositorio:

  

``` json

{

"public": ".",

"rewrites": [

{ "source": "**", "destination": "/index.html" }

],

"headers": [

{

"source": "**",

"headers": [

{

"key": "Strict-Transport-Security",

"value": "max-age=31536000; includeSubDomains"

},

{

"key": "X-Frame-Options",

"value": "SAMEORIGIN"

},

{

"key": "X-Content-Type-Options",

"value": "nosniff"

},

{

"key": "Referrer-Policy",

"value": "strict-origin-when-cross-origin"

},

{

"key": "Permissions-Policy",

"value": "geolocation=(), microphone=(), camera=()"

},

{

"key": "Content-Security-Policy",

"value": "default-src 'self'; script-src 'self' 'unsafe-inline' 'unsafe-eval'; style-src 'self' 'unsafe-inline'; img-src 'self' data: https:; font-src 'self' data:; connect-src 'self' https://pokeapi.co https://*.pokeapi.co;"

}

]

}

]

}

```

  

> **Importante:** La directiva `connect-src` autoriza las peticiones a

> `https://pokeapi.co` y sus subdominios (`https://*.pokeapi.co`) para

> evitar bloqueos en el consumo de la PokeAPI desde Angular.


  

## 5. Paso 5: Configuración del Comando de Inicio (*Startup Command*)

  

Para indicarle a Azure cómo servir la aplicación SPA de Angular sin un

servidor backend Node custom:

  

1. Vuelve al portal de tu **App Service** en Azure.

  

2. En el menú lateral, ve a **Configuración**  \>  **Configuración**

(icono de barritas) \> pestaña **Configuracion de la pila**.

  

3. En el campo **Comando de inicio** (*Startup Command*), ingresa

exactamente la siguiente orden:

  

``` bash

npx serve -s /home/site/wwwroot/pokedex-angular -l 8080

```

  

4. Presiona **Guardar** (*Save*) en la barra superior.

  

5. Dirígete a la pestaña **Información general** (*Overview*) y haz

clic en **Reiniciar** (*Restart*).

  

# PARTE 2: Automatización con CI/CD (GitHub Actions) y Headers de Seguridad

  

En esta segunda fase se automatiza el proceso para que cada cambio

subido al repositorio se despliegue automáticamente.
  

## 1. Paso 1: Conectar GitHub y Generar el Pipeline Automático

  

Para habilitar la Integración y Despliegue Continuo (CI/CD):

  

1. Ve a tu **App Service** en Azure Portal.

2. En el menú lateral, busca la sección **Implementacion** (*Deployment*) y

selecciona **Centro de implementación** (*Deployment Center*).

3. En la pestaña **Configuración**:

-  **Origen** (*Source*): Selecciona **GitHub**.

-  **Autenticación:** Inicia sesión con tu cuenta de GitHub y

autoriza a Azure.

-  **Organización:** Selecciona tu usuario u organización.

-  **Repositorio:** Selecciona el repositorio de tu proyecto

(`pokedex-angular`).

-  **Rama** (*Branch*): Selecciona la rama principal (`main` o

`master`).

4. En **Proveedor de compilación** (*Build provider*), selecciona

**GitHub Actions**.

5. Haz clic en **Guardar** (*Save*) en la parte superior.

  

Al guardar, Azure creará automáticamente un archivo de flujo de trabajo

`.yml` dentro de la carpeta `.github/workflows/` en tu repositorio de

GitHub.

  

A partir de este momento, cada vez que hagas un `git push` a tu rama

principal, GitHub Actions compilará la aplicación, ejecutará las pruebas

y desplegará la versión más reciente a tu App Service en Azure.