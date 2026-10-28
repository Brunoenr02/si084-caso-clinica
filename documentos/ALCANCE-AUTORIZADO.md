# Alcance autorizado del reconocimiento

**Autoriza** Gerencia General de la Clínica Santa Lucía · **Fecha** 20/10/2026 · **Vigencia** la sesión del examen

| Campo | Valor |
|---|---|
| Direcciones autorizadas | `127.0.0.1`, puertos 8080 a 8090. **Ninguna otra** |
| Fase autorizada | OWASP WSTG 4.2, sección 4.1 *Information Gathering*, y la observación pasiva de cabeceras y archivos de la sección 4.2 *Configuration and Deployment Management Testing* |
| Peticiones permitidas | `GET` y `HEAD` a páginas públicas, a `robots.txt`, a `sitemap.xml`, a `/.well-known/` y a las rutas que esos archivos nombran |
| **Prohibido** | Enviar formularios · probar usuarios o contraseñas · descargar archivos de `/respaldos/` · consultar los endpoints `/v1/` de la API · escaneo activo con ZAP, fuerza bruta o *fuzzing* · cualquier dirección fuera de `127.0.0.1` |
| Si encuentra datos personales | Detenga la prueba, anote la ruta y no copie el dato en ninguna evidencia |

> Lo que el reconocimiento encuentre **se planifica, no se prueba**. Las pruebas de autenticación, autorización, sesión y entrada de datos se ejecutan en la Unidad III, con el plan aprobado.
