# Coderhouse - Diplomatura en Desarrollo Full Stack

**Materia:** Mentalidad de Crecimiento y Comunicación en Entornos Digitales  
**Entregable:** Blog Técnico y Control de Versiones  
**Estudiante:** Fer Woolley  
**Contacto:** ferdevinwoolley@gmail.com  
**Fecha:** 30 de junio, 2026  

---

## 1. Ficha Técnica de la Entrega (Checklist)

A continuación, se detalla el checklist de verificación y los enlaces del proyecto:

*   **URL del Blog Técnico (GitHub Pages):** `https://ferwoolley.github.io/coderhouse-blog/` *(Nota: Esta URL se generará de manera pública una vez que subas esta carpeta a tu repositorio de GitHub y habilites GitHub Pages en `Settings > Pages`).*
*   **Enlace al Repositorio Público:** `https://github.com/ferwoolley/coderhouse-blog`
*   **Evidencia de Control de Versiones (Commits del Historial Local):**
    *   `e16ce79` - *docs: agregar entregable en formato markdown para Coderhouse*
    *   `006d724` - *chore: actualizar hashes de commits reales en el blog HTML*
    *   `737abc3` - *chore: documentar solución final de CORS y feedback de mentalidad de crecimiento*
    *   `fc2f0f3` - *fix: integrar estilos CSS del blog con soporte responsive y tipografía premium*
    *   `3d6d93e` - *feat: inicializar estructura HTML del blog técnico para entregable Coderhouse*

---

## 2. Entrada de Blog: "Desafíos en Producción: Depurando CORS y Sesiones en el Despliegue de un Stack MERN"

### Contexto
Durante el desarrollo del proyecto final de nuestra Diplomatura en Desarrollo Full Stack, implementamos una aplicación de gestión de tareas colaborativas basada en el stack **MERN** (MongoDB, Express, React y Node.js). En nuestro entorno local (localhost), la autenticación de usuarios y el manejo de datos fluía sin inconvenientes: el cliente React enviaba las credenciales del usuario al servidor Express, este autenticaba y retornaba una cookie HTTP-Only cifrada que guardaba de manera segura la sesión.

El desafío real se presentó al momento de preparar el entorno de producción. Para el despliegue del frontend seleccionamos **Vercel** (dominio: `https://gestor-frontend.vercel.app`), mientras que la API backend y la base de datos se alojaron en **Render** (dominio: `https://gestor-api.onrender.com`).

---

### Problema
Una vez completado el despliegue de ambos servicios, intentamos realizar la primera prueba de inicio de sesión en producción. La aplicación falló de inmediato. Al abrir la consola del navegador, nos encontramos con un error crítico de **CORS (Cross-Origin Resource Sharing)**:

```text
Access to XMLHttpRequest at 'https://gestor-api.onrender.com/api/auth/login' from origin 'https://gestor-frontend.vercel.app' has been blocked by CORS policy: Response to preflight request doesn't pass access control check: No 'Access-Control-Allow-Origin' header is present on the requested resource.
```

Para solucionar este primer bloqueo, habilitamos CORS en el backend de forma global y básica. Esto permitió que la petición de login fuera exitosa (HTTP status 200 OK), pero desencadenó un segundo problema silencioso: **las cookies de sesión no se guardaban en el navegador del usuario**. Como consecuencia, cualquier petición posterior a rutas protegidas (como obtener el listado de tareas pendientes) devolvía un código `401 Unauthorized`. El sistema estaba completamente inaccesible en producción, lo cual generó gran preocupación en el equipo técnico a pocas horas de la entrega del MVP.

---

### Acciones
Adoptando una **mentalidad de crecimiento**, abordamos el problema de forma constructiva. En lugar de desesperar o buscar culpables por el fallo del servidor en producción, analizamos el comportamiento del protocolo HTTP y de las políticas de cookies modernas para dominios cruzados. Implementamos las siguientes soluciones:

1.  **Reunión y Post-Mortem de Diagnóstico:**
    Determinamos que en `localhost`, al compartir el mismo puerto base o considerarse entornos de desarrollo de confianza, el navegador es flexible con las cookies. Sin embargo, en producción, al estar en dominios completamente distintos (Vercel y Render), los navegadores modernos (Chrome, Safari, Edge) aplican estrictamente la especificación *SameSite* para prevenir ataques de falsificación de peticiones en sitios cruzados (CSRF).

2.  **Reconfiguración de CORS en el Backend (Express):**
    Modificamos el archivo principal del servidor backend (`server.js`) para reemplazar la configuración global permisiva por una política de CORS explícita que permita el intercambio de credenciales:
    ```javascript
    const cors = require('cors');
    
    app.use(cors({
        origin: 'https://gestor-frontend.vercel.app',
        credentials: true, // Habilita la transmisión de cookies
        methods: ['GET', 'POST', 'PUT', 'DELETE', 'OPTIONS'],
        allowedHeaders: ['Content-Type', 'Authorization']
    }));
    ```

3.  **Configuración de Seguridad en las Cookies:**
    Actualizamos el controlador de autenticación para asegurar que las cookies de sesión incluyeran las banderas necesarias para el comportamiento entre dominios cruzados (cross-site):
    ```javascript
    res.cookie('token', token, {
        httpOnly: true, // Protege contra vulnerabilidades XSS
        secure: true,   // Requerido por SameSite=None (fuerza HTTPS)
        sameSite: 'none', // Permite que la cookie se envíe en peticiones cross-site
        maxAge: 24 * 60 * 60 * 1000 // Expira en 1 día
    });
    ```

4.  **Ajuste del Cliente en el Frontend (React & Axios):**
    Configuramos la instancia global de Axios para asegurar que todas las llamadas asíncronas adjuntaran las credenciales en las cabeceras HTTP de forma nativa:
    ```javascript
    import axios from 'axios';
    
    const api = axios.create({
        baseURL: 'https://gestor-api.onrender.com/api',
        withCredentials: true // Adjunta cookies de sesión automáticamente
    });
    ```

---

### Aprendizajes
Este desafío nos dejó valiosas lecciones para futuros desarrollos técnicos:
*   **Entornos Heterogéneos:** Nunca debemos dar por sentado que el funcionamiento en local (localhost) equivale a un despliegue exitoso. La paridad de entornos de desarrollo y producción debe gestionarse con rigor.
*   **Mecanismos de Seguridad Web:** Comprendimos a nivel técnico la interacción entre CORS, las peticiones *Preflight* y el comportamiento de las cookies HTTP-Only frente a las políticas *SameSite* impulsadas por los navegadores modernos.
*   **Proactividad en el Diseño de Red:** En futuros proyectos de la Diplomatura, diseñaremos y probaremos el flujo de CORS y sesiones seguras desde la primera semana de desarrollo para evitar cuellos de botella de última hora en el despliegue.

---

## 3. Reflexión sobre Feedback Radicalmente Sincero (Radical Candor)

Durante la crisis generada por la caída del inicio de sesión en producción, el equipo experimentó momentos de tensión. El desarrollo del backend y el despliegue estaban bajo la responsabilidad de un único programador. En lugar de adoptar conductas defensivas, decidimos aplicar el modelo de **Feedback Radicalmente Sincero (Radical Candor)** de Kim Scott, enfocándonos en el cuadrante de **Afecto Personal con Desafío Directo**:

*   **Desafío Directo (Decir la verdad con claridad):** Nos comunicamos de forma honesta indicando que el sistema estaba fallando en su módulo principal (autenticación) y que necesitábamos corregir el problema de raíz con urgencia para no comprometer la entrega. No suavizamos el problema de forma artificial ni fingimos que no era crítico.
*   **Afecto Personal (Mostrar interés por la persona):** Evitamos las culpas individuales o la agresividad. Le transmitimos al responsable del backend que sabíamos lo compleja que es la configuración de servidores y que confiábamos plenamente en su capacidad. Además, nos involucramos todos en la investigación (haciendo pair programming y pruebas locales).

Al combinar estas dos dimensiones, el compañero se sintió respaldado en lugar de atacado. Evitamos caer en la *agresión ofensiva* (regaños destructivos) y en la *empatía ruinosa* (no decirle nada para no incomodarlo, lo cual hubiera llevado a reprobar el proyecto). El resultado fue la resolución técnica del error en menos de una hora y un equipo mucho más cohesionado y comunicativo.

---

## 4. Registro Histórico de Commits (Evidencia de Git)

Como constancia del trabajo ordenado y el uso de Git como herramienta de control de versiones y colaboración, se anexa el historial de commits ejecutados para la construcción de esta solución:

```bash
$ git log --oneline

e16ce79 docs: agregar entregable en formato markdown para Coderhouse
006d724 chore: actualizar hashes de commits reales en el blog HTML
737abc3 chore: documentar solución final de CORS y feedback de mentalidad de crecimiento
fc2f0f3 fix: integrar estilos CSS del blog con soporte responsive y tipografía premium
3d6d93e feat: inicializar estructura HTML del blog técnico para entregable Coderhouse
```
