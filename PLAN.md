# Plan de auditoría · Seguridad del canal digital de la Clínica Santa Lucía

**SI-084 · Auditoría de Sistemas** · Examen práctico de Unidad II · CONFIDENCIAL — USO ACADÉMICO

| | |
|---|---|
| **Ancco Suaña, Bruno Enrique** | Alumno |
| **2023077472** | Código |
| **https://github.com/Brunoenr02/si084-caso-clinica/tree/examen-u2** | URL GitHub |
| **28/10/2026** | Fecha |

## 1. Resultados del reconocimiento · OWASP WSTG 4.2

| Prueba | Identificador WSTG | Lo que observé, con cifras | Declaración del proveedor que contrasta | ¿Coincide? | Evidencia |
|---|---|---|---|---|---|
| E1 | WSTG-INFO-04 | Responden 3 aplicaciones en puertos 8080 (portal web HTTP 200), 8081 (entorno de pruebas HTTP 200 con base de datos de producción restaurada el 12/09/2026 y usuario admin_pruebas) y 8088 (API REST de citas v1.4.0 HTTP 200 con entorno produccion y endpoints /v1/citas, /v1/pacientes, /v1/resultados expuestos). | D1 | No | `evidencias/E1_aplicaciones.txt` |
| E2 | WSTG-INFO-02 | Servidor web expone Server: nginx/1.20.2 (rama 1.20 en fin de ciclo de vida / EOL desde 2022) y lenguaje X-Powered-By: PHP/7.4.3 (soporte oficial y parches de seguridad finalizados el 28/11/2022). Ambas versiones sin soporte vigente. | D2 | No | `evidencias/E2_servidor.txt` |
| E3 | WSTG-INFO-03 | El archivo robots.txt prohíbe la indexación de 4 rutas críticas (/admin/, /respaldos/, /resultados/, /wp-content/plugins/wp-file-manager/) y referencia a sitemap.xml. El archivo security.txt no existe (retorna HTTP 404). | — | — | `evidencias/E3_metarchivos.txt` |
| E4 | WSTG-INFO-05 | Se identifican múltiples fugas internas en comentarios y etiquetas: versión WordPress 5.8.1, usuario sensible sin depurar (admin_pruebas), IP y puerto interno de laboratorio (http://10.20.0.15:8090/lis/resultados), contraseña inicial débil (formato ddmmaaaa) y patrón directo de consulta de resultados (ver.php?orden=NNNNN). | D8 | No | `evidencias/E4_contenido.txt` |
| E5 | WSTG-INFO-08 | Versión de WordPress 5.8.1 (antigua, afectada por CVE-2022-21661 SQLi y CVE-2022-21662 XSS), tema medicalpro v2.1.0, plugin contact-form-7 v5.3.1 (afectado por CVE-2020-35489 de subida arbitraria), jquery v3.4.1 (CVE-2020-11022 XSS) y wp-file-manager v6.8 con elFinder (vulnerable a ejecución remota RCE). Componentes sin actualización mensual. | D3 | No | `evidencias/E5_componentes.txt` |
| E6 | WSTG-INFO-06 | Se detectan 4 formularios (/citas/, /resultados/, /postula/, /admin/). El formulario /citas/ solicita datos personales y de salud (motivo_consulta y síntomas) y los transmite por método GET en texto claro HTTP sin cifrado. El formulario /postula/ recibe archivos (input file cv) vía multipart. Todos los actions usan esquema http no seguro. | D7 | No | `evidencias/E6_entradas.txt` |
| E7 | WSTG-CONF-04 · WSTG-CONF-05 | Directorio /respaldos/ con indexación habilitada lista públicamente 4 archivos: citas_export_2026-08.csv, portal_2025-09-30.sql.gz, portal_2026-03-31.sql.gz y wp-config.php.bak. La consola de administración /admin/ responde HTTP 200 accesible desde cualquier dirección externa sin restricción perimetral. | D5 · D6 | No | `evidencias/E7_expuestos.txt` |
| E8 | WSTG-CONF-07 · WSTG-CONF-12 · WSTG-SESS-02 | Ausencia total de las 4 cabeceras de seguridad requeridas (Strict-Transport-Security, Content-Security-Policy, X-Frame-Options, X-Content-Type-Options: todas AUSENTE). La cookie PHPSESSID carece de atributos Secure, HttpOnly y SameSite. El servicio HTTPS en puerto 443 no responde (HTTP 000). | D4 | No | `evidencias/E8_cabeceras.txt` |

## 2. Materia de control · Sección III

Auditoría de seguridad y protección de datos personales en el canal digital de la Clínica Santa Lucía S.A.C., correspondiente al periodo del 01 de enero al 30 de setiembre de 2026. Esta materia se justifica por el incidente crítico reportado el 08/09/2026 donde una paciente visualizó el hemograma de otra persona, y se sustenta en las observaciones de reconocimiento E4 y E7, que evidenciaron la exposición de parámetros directos de consulta (`ver.php?orden=NNNNN`) y el acceso público a copias completas de bases de datos en `/respaldos/`, comprometiendo gravemente la confidencialidad de la información médica.

## 3. Alcance · Sección IV

| Bloque | Contenido |
|---|---|
| Incluye | Activos cubiertos con sus direcciones y poblaciones: <br>1. **Portal web institucional** (`http://127.0.0.1:8080/`): comprende módulo `/citas/` (población de 3 850 citas mensuales), módulo `/resultados/` (12 400 pacientes registrados con cuenta), módulo `/postula/` (214 postulaciones de empleo con CV en 2026), directorio `/respaldos/` (4 archivos de copias de seguridad expuestos) y consola `/admin/`. <br>2. **Entorno de pruebas no declarado** (`http://127.0.0.1:8081/`): base de datos de producción restaurada el 12/09/2026 con registros históricos de citas y pacientes. <br>3. **API REST de citas** (`http://127.0.0.1:8088/`): versión 1.4.0 en entorno de producción con endpoints `/v1/citas`, `/v1/pacientes`, `/v1/resultados` y documentación `/swagger/`. |
| Periodo auditado | Del **01/01/2026 al 30/09/2026**. Se delimita este lapso porque abarca el funcionamiento operativo del canal digital durante el presente ejercicio y contiene el incidente de vulneración de datos clínicos ocurrido el 08/09/2026, permitiendo evaluar la eficacia y persistencia de las configuraciones y controles en dicho intervalo. |
| Excluye expresamente | 1. Red de área local interna de la clínica y servidores locales del Sistema de Información de Laboratorio (LIS en `10.20.0.15:8090`), por no encontrarse expuestos públicamente en el canal digital y requerir acceso físico no autorizado.<br>2. Pruebas de denegación de servicio (DoS/DDoS) o escaneos destructivos de fuerza bruta masiva, a fin de garantizar la continuidad asistencial de los servicios de salud conforme a `ALCANCE-AUTORIZADO.md`.<br>3. Dispositivos de usuario final utilizados por pacientes y postulantes, dado que están fuera del control técnico y de alojamiento de la entidad y del proveedor. |
| Limitaciones inherentes | 1. La auditoría proporciona una seguridad razonable, mas no absoluta, sujeta a la representatividad de los muestreos efectuados y a la exactitud de los registros y bitácoras disponibles.<br>2. Imposibilidad de revisar la totalidad del código fuente subyacente si el proveedor WebSur S.A.C. no formaliza la entrega de los repositorios de desarrollo, circunscribiendo el análisis a los artefactos e interfaces desplegados en la réplica autorizada. |

## 4. Objetivos · Sección V

**Objetivo general.**
Determinar si los controles de seguridad de la información y protección de datos implementados en el canal digital de la Clínica Santa Lucía S.A.C. garantizan de manera razonable la confidencialidad, integridad y disponibilidad de los datos de salud de los pacientes, de conformidad con la Guía de Pruebas de Seguridad Web OWASP WSTG 4.2, el Estándar de Verificación de Seguridad en Aplicaciones OWASP ASVS 4.0.3, la norma NTP-ISO/IEC 27001:2022 y la Ley N.º 29733 de Protección de Datos Personales.

| N.º | Objetivo específico | Pregunta de auditoría |
|---|---|---|
| OE1 | Determinar si los mecanismos de autenticación, gestión de sesiones y protección perimetral del portal institucional y consolas administrativas previenen el acceso no autorizado y la fuga de información sensible, conforme a OWASP WSTG 4.2 (WSTG-ATHN, WSTG-SESS), OWASP ASVS 4.0.3 (V2, V3) y NTP-ISO/IEC 27001:2022 A.8.5, A.8.20 y A.8.24. | ¿Los mecanismos de autenticación, sesión y protección de directorios sensibles impiden que usuarios no autorizados accedan al panel de gestión y descarguen las copias de seguridad y configuraciones de la clínica? |
| OE2 | Evaluar si el módulo de consulta de resultados de laboratorio implementa controles de autorización a nivel de objeto que impidan la visualización de datos médicos pertenecientes a terceros (respondiendo al incidente del 08/09/2026), conforme a OWASP WSTG 4.2 (WSTG-ATHZ-04), OWASP ASVS 4.0.3 (V4.1, V4.2), NTP-ISO/IEC 27001:2022 A.8.3 y la Ley 29733 Arts. 13 y 39. | ¿El mecanismo de entrega y consulta de resultados valida rigurosamente la sesión del paciente para evitar que mediante la modificación de identificadores u órdenes se expongan análisis de laboratorio de otros pacientes? |
| OE3 | Verificar si los canales de transmisión de datos y los formularios de captura y recepción de archivos implementan cifrado robusto y validaciones seguras conforme a OWASP WSTG 4.2 (WSTG-CRYP-03, WSTG-INPV), OWASP ASVS 4.0.3 (V5, V9) y NTP-ISO/IEC 27001:2022 A.8.24 y el principio de seguridad de la Ley 29733. | ¿El canal digital implementa cifrado en tránsito HTTPS y controles eficaces en la carga de archivos para salvaguardar los datos de los formularios y evitar la inyección de código ejecutable malicioso? |

## 5. Enfoque, materialidad y riesgo · Sección VI

| Elemento | Decisión | Sustento en lo que observó o en la solicitud |
|---|---|---|
| Enfoque | Sustantivo | El reconocimiento técnico evidenció la ausencia absoluta de controles preventivos esenciales declarados por el proveedor (0 de 8 coincidencias, puerto 443 sin HTTPS, directorio `/respaldos/` con indexación libre, panel `/admin/` desprotegido y envío de datos de salud por GET). Dado que un control inexistente no admite pruebas de eficacia operativa, la comisión ejecutará pruebas sustantivas directas de validación e inspección. |
| Materialidad cuantitativa | S/ 186 000 | Establecida en el 1% de los ingresos ordinarios del año 2025 de la clínica (S/ 18 600 000 reportados en la solicitud de auditoría), umbral financiero alineado al impacto potencial de multas graves aplicables por SUSALUD y la ANPDP (de hasta 100 UIT). |
| Materialidad cualitativa | Compromiso de datos sensibles de salud | Constituye una desviación material cualquier evento o vulnerabilidad que exponga datos sensibles de salud (Ley N.º 29733, Arts. 2 y 13; D.S. 003-2013-JUS) de los 12 400 pacientes o datos de postulantes, independientemente del monto económico, por el daño irreparable a los derechos fundamentales y la reputación asistencial. |
| Riesgo inherente | Alto | Justificado en el volumen y sensibilidad extrema de la información clínica tratada (diagnósticos, citas médicas y resultados de 12 400 pacientes registrados), la exposición pública directa en internet sin filtrado perimetral y el antecedente del incidente ocurrido el 08/09/2026. |
| Riesgo de control | Alto | Sustentado en la falta comprobada de controles internos por parte de WebSur S.A.C.: omisión de cifrado TLS, almacenamiento de respaldos en el servidor web, uso de versiones obsoletas sin soporte (PHP 7.4.3 en EOL, WordPress 5.8.1), contraseñas por defecto predecibles y ausencia de cabeceras de seguridad. |
| Riesgo de detección | Bajo | La comisión auditora estructurará un programa de auditoría exhaustivo, aplicando procedimientos sustantivos detallados sobre la réplica completa del canal digital para reducir a un nivel bajo la probabilidad de no detectar deficiencias existentes. |

## 6. Procedimientos · Sección VII

El programa de auditoría comprende seis procedimientos sustantivos detallados (P-01 a P-06) que cubren los tres objetivos específicos formulados, diseñados para ser ejecutados en la Unidad III sobre la réplica autorizada del canal digital.

### P-01

| Columna | Contenido |
|---|---|
| Objetivo específico | OE2 |
| Origen en el reconocimiento | E4 · Comentario en código fuente de `/resultados/` que expone el patrón de consulta directa mediante parámetro `ver.php?orden=NNNNN` sin validación de contexto de sesión. |
| Riesgo | Exposición y divulgación no autorizada de resultados de exámenes clínicos ajenos mediante referencia directa a objetos insegura (IDOR), respondiendo directamente al incidente del 08/09/2026. |
| Criterio | WSTG-ATHZ-04 · OWASP ASVS 4.0.3 V4.1, V4.2 · NTP-ISO/IEC 27001:2022 A.8.3 · Ley 29733 Arts. 13 y 39 |
| Tipo de prueba | Sustantiva / Verificación de control de autorización a nivel de objeto |
| Población y muestra | Población: 12 400 pacientes registrados y totalidad de exámenes médicos emitidos en 2026. Muestra: 30 registros de órdenes de laboratorio seleccionados de forma correlativa y aleatoria. |
| Procedimiento | 1. Iniciar sesión en el portal con una cuenta de paciente de prueba asignada.<br>2. Localizar y registrar el identificador numérico de orden retornado en la URL de consulta `/resultados/ver.php?orden=NNNNN`.<br>3. Modificar manualmente el valor del parámetro `orden` mediante cliente HTTP o interceptor proxy, sustituyéndolo por identificadores de órdenes contiguas correspondientes a otros pacientes.<br>4. Reenviar la petición modificada y registrar el código de respuesta HTTP y el contenido del cuerpo recibido.<br>5. Enviar una petición directa `GET /resultados/ver.php?orden=NNNNN` desde una sesión anónima (sin cookie de sesión).<br>6. Verificar si la respuesta expone nombres, datos de filiación o valores del informe de laboratorio solicitados. |
| Evidencia a obtener | Capturas de tráfico HTTP de peticiones y respuestas con visualización de resultados ajenos, registros de logs del servidor web y reporte técnico de hallazgo de IDOR. |
| Definición de excepción | Toda respuesta con código HTTP 200 que muestre datos personales, clínicos o análisis médicos asociados a un identificador de orden que no pertenezca al usuario actualmente autenticado, o cualquier entrega de resultados a peticiones no autenticadas. |
| Horas estimadas | 6 horas |

### P-02

| Columna | Contenido |
|---|---|
| Objetivo específico | OE1 |
| Origen en el reconocimiento | E7 · Directorio `/respaldos/` con indexación libre habilitada listando volcados `.sql.gz`, exportaciones `.csv` y archivos `.bak`. |
| Riesgo | Descarga no autorizada de copias completas de la base de datos y credenciales maestras por actores externos, derivando en la filtración masiva de datos y pérdida total de confidencialidad. |
| Criterio | WSTG-CONF-04 · OWASP ASVS 4.0.3 V14.3 · NTP-ISO/IEC 27001:2022 A.8.20, A.8.24 · Ley 29733 Art. 17 |
| Tipo de prueba | Sustantiva / Inspección de directivas de configuración y permisos del servidor web |
| Población y muestra | Población: 4 archivos de respaldo detectados en `/respaldos/` y archivos sensibles en la raíz del portal. Muestra: 100% (los 4 archivos identificados: `wp-config.php.bak`, `citas_export_2026-08.csv`, `portal_2025-09-30.sql.gz`, `portal_2026-03-31.sql.gz`). |
| Procedimiento | 1. Emitir peticiones HTTP `HEAD` y `GET` hacia cada una de las URLs de los 4 archivos alojados en `/respaldos/`.<br>2. Comprobar si el servidor retorna código de estado `200 OK` y permite la transferencia completa del contenido sin requerir autenticación.<br>3. Examinar el archivo de configuración `portal.conf` del servidor Nginx para constatar la presencia de la directiva `autoindex on`.<br>4. Verificar si existen directivas de bloqueo explícito (`deny all`) para extensiones críticas (`.bak`, `.sql`, `.gz`, `.csv`, `.env`).<br>5. Documentar los permisos del sistema de archivos en el volumen o directorio `/respaldos`. |
| Evidencia a obtener | Encabezados de respuesta HTTP 200 con tipos de contenido transferidos, copia del archivo de configuración `portal.conf` y registro de accesibilidad de archivos sensibles. |
| Definición de excepción | Respuesta HTTP con código 200 o habilitación de descarga directa para cualquier archivo con extensión `.bak`, `.sql`, `.gz` o `.csv` ubicado dentro del árbol web público, o presencia de `autoindex on` en el servidor web. |
| Horas estimadas | 4 horas |

### P-03

| Columna | Contenido |
|---|---|
| Objetivo específico | OE1 |
| Origen en el reconocimiento | E4 · Mensaje en formulario `/resultados/` indicando que la contraseña inicial corresponde a la fecha de nacimiento (`ddmmaaaa`) y comentario con usuario residual `admin_pruebas`. |
| Riesgo | Acceso indebido y suplantación de cuentas de pacientes o perfiles administrativos mediante adivinación o ataques de fuerza bruta basados en credenciales predecibles. |
| Criterio | WSTG-ATHN-02, WSTG-ATHN-03 · OWASP ASVS 4.0.3 V2.1, V2.2 · NTP-ISO/IEC 27001:2022 A.8.5 |
| Tipo de prueba | Sustantiva / Evaluación de robustez de mecanismos de autenticación y políticas de bloqueo |
| Población y muestra | Población: Formularios de autenticación de `/admin/` y `/resultados/`. Muestra: Formularios de ambos módulos sometidos a secuencias de 20 intentos de acceso. |
| Procedimiento | 1. Enviar una ráfaga controlada de 10 intentos de autenticación consecutivos con contraseñas inválidas en `/admin/` para el usuario `admin_pruebas`.<br>2. Constatar si el servicio aplica mecanismos de bloqueo de cuenta, retraso de respuesta (tarpitting) o solicitud de desafío CAPTCHA.<br>3. Probar en el módulo `/resultados/` la autenticación de una cuenta de prueba empleando una clave con patrón de 8 dígitos numéricos `ddmmaaaa`.<br>4. Verificar si tras el primer inicio de sesión con contraseña por defecto el sistema fuerza la renovación de la clave por una contraseña robusta.<br>5. Analizar si la aplicación emite mensajes de error diferenciados que permitan enumerar usuarios válidos. |
| Evidencia a obtener | Bitácora de peticiones POST de autenticación, tiempos de respuesta HTTP, mensajes devueltos por el servidor y comprobación de ausencia de bloqueo de cuentas. |
| Definición de excepción | Omisión de mecanismos de bloqueo de cuenta o retardo tras 5 intentos fallidos sucesivos de acceso, o admisión de contraseñas débiles por defecto de 8 dígitos sin requerir su cambio forzoso en el primer inicio de sesión. |
| Horas estimadas | 5 horas |

### P-04

| Columna | Contenido |
|---|---|
| Objetivo específico | OE1 |
| Origen en el reconocimiento | E8 · Cookie de sesión `PHPSESSID` emitida sin directivas `Secure`, `HttpOnly` ni `SameSite`. |
| Riesgo | Robo y secuestro de sesión (*Session Hijacking*) y facilitación de ataques Cross-Site Scripting (XSS) y Cross-Site Request Forgery (CSRF) sobre sesiones activas de pacientes y administradores. |
| Criterio | WSTG-SESS-02 · OWASP ASVS 4.0.3 V3.4 · NTP-ISO/IEC 27001:2022 A.8.20, A.8.24 |
| Tipo de prueba | Sustantiva / Análisis de configuración de identificadores de sesión en cabeceras HTTP |
| Población y muestra | Población: Todas las cookies emitidas por el aplicativo (`PHPSESSID` y cookies de WordPress). Muestra: 100% de las cookies generadas en transacciones públicas y autenticadas. |
| Procedimiento | 1. Realizar peticiones HTTP `GET` a las rutas `/`, `/admin/` y `/resultados/` inspeccionando las cabeceras de respuesta `Set-Cookie`.<br>2. Verificar la ausencia de las banderas de seguridad obligatorias: `Secure`, `HttpOnly` y `SameSite` (`Strict` o `Lax`).<br>3. Evaluar mediante consola de desarrollador si la cookie `PHPSESSID` es accesible mediante código JavaScript (`document.cookie`).<br>4. Iniciar sesión exitosamente y contrastar si el identificador de sesión es regenerado o si mantiene el mismo valor previo a la autenticación (vulnerabilidad de fijación de sesión). |
| Evidencia a obtener | Captura textual de cabeceras HTTP `Set-Cookie`, captura de ejecución de script en consola confirmando lectura de cookies y comparación de tokens pre y post autenticación. |
| Definición de excepción | Generación de cualquier cookie de sesión o autenticación que carezca de alguna de las directivas `HttpOnly`, `Secure` o `SameSite`, o conservación del mismo token de sesión antes y después de iniciar sesión. |
| Horas estimadas | 4 horas |

### P-05

| Columna | Contenido |
|---|---|
| Objetivo específico | OE3 |
| Origen en el reconocimiento | E8 · Puerto HTTPS 443 inactivo (HTTP 000) y ausencia de cabeceras de endurecimiento; y E6 · Envío de datos personales y síntomas de salud por HTTP GET. |
| Riesgo | Interceptación y lectura no autorizada de datos personales y clínicos en tránsito por la red (ataques Man-in-the-Middle) e incrustación maliciosa en páginas fraudulentas (Clickjacking). |
| Criterio | WSTG-CRYP-03 · OWASP ASVS 4.0.3 V9.1, V14.4 · NTP-ISO/IEC 27001:2022 A.8.20, A.8.24 · Ley 29733 Art. 17 |
| Tipo de prueba | Sustantiva / Verificación de protocolos criptográficos en tránsito y cabeceras de seguridad |
| Población y muestra | Población: 3 aplicaciones web detectadas (puertos 8080, 8081, 8088). Muestra: 100% de los servicios expuestos y formularios web. |
| Procedimiento | 1. Ejecutar sonda TCP y solicitud TLS hacia el puerto 443 de la infraestructura para verificar la disponibilidad de canal seguro.<br>2. Enviar peticiones HTTP sobre el puerto 80 hacia formularios de captura y constatar si existe redirección forzosa `301 Moved Permanently` a `https://`.<br>3. Examinar en las respuestas HTTP la presencia y configuración de las cabeceras `Strict-Transport-Security`, `Content-Security-Policy`, `X-Frame-Options` y `X-Content-Type-Options`.<br>4. Verificar que los atributos `action` de los formularios `/citas/`, `/resultados/` y `/postula/` utilicen estrictamente el protocolo `https://`. |
| Evidencia a obtener | Registro de sondeo TCP/TLS en puerto 443, matriz de evaluación de cabeceras de seguridad HTTP y capturas de solicitudes transmitiendo en texto claro. |
| Definición de excepción | Disponibilidad de canales de comunicación o formularios que acepten peticiones en texto plano HTTP sin redirección automática a HTTPS, o ausencia de las cabeceras HSTS y X-Frame-Options en páginas de ingreso de datos. |
| Horas estimadas | 5 horas |

### P-06

| Columna | Contenido |
|---|---|
| Objetivo específico | OE3 |
| Origen en el reconocimiento | E6 · Formulario `/postula/` con recepción de archivos (`input file cv`) vía multipart; y E5 · Componentes desactualizados con riesgo de ejecución de código. |
| Riesgo | Carga y almacenamiento de archivos ejecutables maliciosos (web shells) disfrazados de documentos de postulación, propiciando la toma de control del servidor web. |
| Criterio | WSTG-INPV-12 · OWASP ASVS 4.0.3 V12.1, V12.5 · NTP-ISO/IEC 27001:2022 A.8.28, A.8.31 |
| Tipo de prueba | Sustantiva / Evaluación de controles de subida y almacenamiento de archivos |
| Población y muestra | Población: Formulario de postulación laboral `/postula/` y carpeta de almacenamiento de currículums. Muestra: 10 ensayos de subida con extensiones ejecutables directas, dobles y tipos MIME manipulados. |
| Procedimiento | 1. Enviar a través del formulario `/postula/` archivos con extensiones ejecutables directas (`.php`, `.phtml`, `.phar`) y dobles extensiones (`.pdf.php`).<br>2. Probar la subida de archivos modificando la cabecera `Content-Type` simulando un PDF legítimo pero conteniendo scripts de prueba inocuos.<br>3. Verificar si el aplicativo valida el número mágico (*magic bytes*) del encabezado binario del archivo antes de almacenarlo.<br>4. Inspeccionar la ruta de almacenamiento en el servidor para comprobar si se ubica fuera de la raíz web (`docroot`) y si se encuentran deshabilitados los permisos de ejecución.<br>5. Verificar si el servidor renombra el archivo con una cadena pseudoaleatoria única eliminando el nombre original aportado por el postulante. |
| Evidencia a obtener | Registros de peticiones HTTP POST multipart de carga de archivos, respuestas emitidas por el servidor web y listado de atributos de la carpeta de subidas en el servidor. |
| Definición de excepción | Almacenamiento exitoso de archivos que contengan extensiones o firmas binarias ejecutables, conservación del nombre original del archivo subido o ubicación del repositorio de archivos en un directorio con permisos de ejecución dentro del árbol web. |
| Horas estimadas | 6 horas |

## 7. Anexo A · Matriz de riesgos

| N.º | Riesgo | Causa, con su evidencia | Impacto | Nivel | Control existente | Control esperado |
|---|---|---|---|---|---|---|
| R1 | Exposición y divulgación de resultados de laboratorio de pacientes sin autorización (IDOR). | El módulo de consulta de resultados emplea parámetros secuenciales directos (`ver.php?orden=NNNNN`) sin validar la sesión del titular legítimo, evidenciado en E4. | Violación de la confidencialidad médica de 12 400 pacientes, sanciones de SUSALUD y ANPDP (hasta 100 UIT) y daño reputacional irreparable. | Alto | Inexistente (declaración D7 y alegato de WebSur de «error puntual de caché»). | Control de autorización a nivel de objeto verificando la sesión activa y uso de identificadores indirectos o UUIDs no secuenciales (OWASP ASVS V4.2). |
| R2 | Descarga pública y compromiso total de bases de datos y configuraciones del portal. | El directorio `/respaldos/` cuenta con listado de directorios habilitado y archivos descargables (`.sql.gz`, `.bak`, `.csv`) sin autenticación ni restricción, evidenciado en E7. | Acceso irrestricto a historiales de citas, credenciales maestras y estructura de datos de la clínica por cualquier usuario de internet. | Alto | Regla disuasiva en `robots.txt` (sin efectividad de control técnico perimetral). | Almacenamiento de respaldos fuera de la raíz web pública, directiva `autoindex off` y restricción perimetral de acceso (NTP-ISO/IEC 27001:2022 A.8.20). |
| R3 | Interceptación de datos personales y síntomas de salud durante la transmisión en red (MitM). | Inexistencia de cifrado HTTPS (puerto 443 inactivo HTTP 000) y formulario de citas transmitiendo datos personales y síntomas por HTTP GET, evidenciado en E6 y E8. | Exposición de datos de filiación y motivos de consulta médica en registros proxy, caché y redes interceptables, violando la Ley 29733. | Alto | Inexistente (declaración D4 desvirtuada). | Implementación obligatoria de TLS 1.3 con redirección 301, cabecera HSTS y uso exclusivo del método POST bajo HTTPS (OWASP ASVS V9.1). |
| R4 | Suplantación de identidad de pacientes y acceso ilegítimo mediante fuerza bruta de claves. | Asignación de contraseñas iniciales predecibles (`ddmmaaaa`), ausencia de bloqueo de intentos y cookie de sesión sin banderas de seguridad, evidenciado en E4 y E8. | Compromiso masivo de cuentas de usuarios, suplantación de identidad de pacientes y acceso indebido a información privada. | Alto | Mensaje sugiriendo contacto mediante WhatsApp (sin control de seguridad en la aplicación). | Políticas de contraseñas robustas, bloqueo temporal tras 5 intentos fallidos y cookies configuradas con flags `Secure`, `HttpOnly` y `SameSite` (ASVS V2.1, V3.4). |
| R5 | Ejecución remota de código (RCE) y control del servidor mediante componentes desactualizados o subida de archivos. | Servidor ejecutando WordPress 5.8.1, PHP 7.4.3 en EOL y plugins vulnerables (`wp-file-manager 6.8`, `contact-form-7 5.3.1`) con formulario de subida de archivos, evidenciado en E2, E5 y E6. | Compromiso del servidor web, alteración maliciosa del sitio (*defacement*), robo de datos y potencial movimiento lateral hacia sistemas internos (LIS). | Alto | Inexistente (declaraciones D2 y D3 no cumplidas por el proveedor). | Proceso formal de parchado mensual verificado, actualización de versiones de PHP y CMS, y validación estricta de extensiones y tipos de archivo con ejecución deshabilitada (ASVS V12.1). |
