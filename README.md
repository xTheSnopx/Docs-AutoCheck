
# Docs-AutoCheck

Documentacion de AutoCheck

Este repositorio contiene la documentación visual e informes ejecutivos de AutoCheckAML. Para ver el informe de avance completo y los requisitos del sistema de forma interactiva, abra el archivo [index.html](file:///c:/Users/XJuan/source/repos/AutoCheckAML/Docs-AutoCheck/index.html) en su navegador.

## 💻 ¿Qué necesita un computador para correr el proyecto?

Para poder ejecutar y desarrollar en este proyecto, su computador requiere tener instalado:

### 🛠️ Para desarrollo local:

* **.NET 8.0 SDK** (o superior): Requerido para compilar y ejecutar el proyecto de Backend (`AutoCheckAML.Api`).
* **Node.js 18** (o superior) junto con **npm**: Requerido para instalar dependencias y ejecutar el Frontend desarrollado en React con Vite.
* **SQLite**: Usado como base de datos por defecto para el desarrollo local (el archivo `.db` se genera automáticamente al iniciar el Backend).
* **IDE / Editor**: Visual Studio 2022 o Visual Studio Code (con la extensión C# Dev Kit).

### 🐳 Para despliegue / ejecución unificada:

* **Docker & Docker Compose**: Permite levantar todos los servicios (Frontend, Backend y la base de datos de producción PostgreSQL) sin necesidad de configurar SDKs locales individuales.

---

## 🚀 Guía rápida de ejecución

### Opción 1: Desarrollo local (separado)

1. **Backend**:
   ```bash
   cd BackEnd-AutoCheck/AutoCheckAML.Api
   dotnet restore
   dotnet build
   dotnet run
   ```
2. **Frontend**:
   ```bash
   cd FrontEnd-AutoCheck
   npm install
   npm run dev
   ```

### Opción 2: Docker Compose (unificado)

1. En la raíz del repositorio principal ejecute:
   ```bash
   docker-compose up --build
   ```
