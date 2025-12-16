# 🎓 Frontend – Sistema Académico UCB

Este repositorio contiene el **frontend** del sistema académico, desarrollado con **Vue 3** y **Vite**.  
La aplicación se encarga de la interfaz de usuario e interactúa con un **backend mediante una API REST** para la autenticación y consulta de información académica.

---

## Tecnologías utilizadas

- Vue.js 3
- Vite
- Vue Router
- JavaScript
- HTML5
- CSS3

---

## Estructura del proyecto

```txt
src/
│
├── assets/            # Imágenes y recursos estáticos
│   └── img/
│
├── router/            # Configuración de rutas
│   └── index.js
│
├── views/             # Vistas principales del sistema
│   ├── Login.vue
│   ├── Horario.vue
│   ├── SIAAN.vue
│   └── VistaResultados.vue
│
├── main.js            # Punto de entrada de la aplicación
│
vite.config.js         # Configuración de Vite

## Requisitos previos

Antes de ejecutar el proyecto, asegúrate de tener instalado:

Node.js (versión 18 o superior recomendada)

npm o yarn

## Instalación y ejecución

1. Clona el repositorio o descarga el proyecto
2. Accede a la carpeta del frontend:
-cd Vue

3. Instala las dependencias:
-npm install

4. Ejecuta el servidor de desarrollo:
-npm run dev

5. Abre el navegador y accede a:
-http://localhost:5173

## Conexión con el Backend

Este frontend consume servicios de un backend mediante HTTP.

Si el backend utiliza una URL configurable, crea un archivo .env en la raíz del proyecto con el siguiente contenido:
-VITE_API_URL=http://localhost:3000

##  Características

###  **Autenticación Dual**
- Login tradicional con validación en tiempo real
- **Google OAuth2** para cuentas @ucb.edu.bo
- Persistencia de sesión con localStorage

###  **Malla Curricular Interactiva**
- Visualización por semestres
- Estados diferenciados: ✅ Aprobado, ⭐ Seleccionado, 🔓 Disponible, 🔒 Bloqueado
- Sistema automático de **prerrequisitos**
- Materias de **inglés** como requisito transversal

###  **Generador Inteligente de Horarios**
- Selección visual de materias
- Algoritmo de generación de múltiples opciones
- Prevención de conflictos de horario
- Exportación **Excel y PDF**

### **Interfaz Moderna**
- Diseño **responsive** con temática UCB
- Sidebar colapsable con menú jerárquico
- Carrusel interactivo con autoplay
- Feedback visual en tiempo real

## Dependencia del Backend
Este proyecto **requiere** que el backend esté en ejecución para funcionar correctamente.
Sin el backend, solo se puede visualizar la interfaz.
