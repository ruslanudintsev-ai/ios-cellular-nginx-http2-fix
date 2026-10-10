# ¿Tu iPhone no abre un sitio HTTPS con datos móviles? Un caso real de Nginx y HTTP/2

**Idiomas:** [English](README.md) · [Русский](README.ru.md) · [Español](README.es.md) · [Português (Brasil)](README.pt-BR.md) · [简体中文](README.zh-CN.md)

**Estado del caso:** tras habilitar HTTP/2 en los servidores virtuales HTTPS de producción, el mismo iPhone pudo abrir la web de inmediato mediante datos móviles y sin VPN. La observación se registró el **10 de octubre de 2026**. En el momento de redactar este informe todavía no habían transcurrido 24 horas, por lo que la estabilidad a largo plazo seguía sin comprobarse.

Este caso corresponde a [polishelp.ru](https://polishelp.ru/). Es un informe práctico basado en las observaciones del responsable del sitio, **no un estudio de laboratorio**.

## Resumen

Durante la investigación, Safari en varios iPhone se quedaba cargando o no mostraba el sitio a través de datos móviles sin VPN. Por Wi-Fi o VPN funcionaba, y se informó de que Android con datos móviles también funcionaba. Los resultados fueron variables: algunas conexiones no llegaban a generar una solicitud HTTP; otras devolvían el HTML con **HTTP 200** sin que la página llegara a mostrarse correctamente.

Se probaron otro servidor y otros nombres de host, versiones estáticas simplificadas, JavaScript, gzip, la precarga de rutas de Next.js, la configuración de TLS y capturas de paquetes. Ninguna de esas pruebas proporcionó una solución fiable. En producción, Nginx estaba negociando **HTTP/1.1**.

Primero se habilitó HTTP/2 en un servidor virtual de diagnóstico. El responsable informó de que la página de prueba se abrió inmediatamente en el iPhone afectado, con datos móviles y sin VPN. Después se habilitó HTTP/2 en los servidores virtuales HTTPS del dominio principal y de `www`. La misma URL de producción se abrió de inmediato en el mismo iPhone y conexión móvil. Se conservó la compatibilidad con HTTP/1.1.

## Qué demuestra —y qué no— el resultado

La comparación antes/después en **el mismo dominio y certificado de producción** es una señal importante de que el cambio a HTTP/2, o su interacción con el camino anterior por HTTP/1.1, influyó en el problema. Es una prueba más útil que comparar dos dominios distintos.

Sin embargo, **no identifica un fallo concreto en Safari, Nginx, TLS, HTTP/1.1 o el operador móvil**. Las comprobaciones en el teléfono fueron comunicadas por el responsable y no estuvieron acompañadas de una captura de paquetes sincronizada. Las condiciones de la red y el estado del navegador también pueden cambiar.

Además, el cambio requirió una recarga gradual de Nginx, que pudo afectar al estado de las conexiones. Una apertura exitosa por sí sola no separa por completo el efecto de HTTP/2 del efecto de esa recarga. El mecanismo de fondo sigue siendo desconocido.

Al activar HTTP/2 en producción no se modificaron la política de cifrado/TLS, el código de la aplicación, el DNS ni el contenido del sitio.

## Documentación detallada

La documentación técnica está disponible en inglés y ruso:

- [Cronología de la investigación (EN)](docs/timeline.md) · [RU](docs/timeline.ru.md)
- [Pruebas fallidas o inconcluyentes (EN)](docs/failed-attempts.md) · [RU](docs/failed-attempts.ru.md)
- [Método de diagnóstico (EN)](docs/diagnostics.md) · [RU](docs/diagnostics.ru.md)
- [Activación y reversión de HTTP/2 en Nginx (EN)](docs/nginx-http2.md) · [RU](docs/nginx-http2.ru.md)

## Advertencia práctica

Este informe **no recomienda habilitar HTTP/2 a ciegas como solución universal**. Comprueba la versión de Nginx y la compatibilidad del módulo, guarda la configuración original, ejecuta `nginx -t`, aplica el cambio solo en los servidores virtuales previstos y conserva un plan de reversión. Verifica que realmente se ha negociado HTTP/2 y que HTTP/1.1 sigue funcionando. Realiza varias pruebas en el dispositivo y la red afectados.

La directiva `http2 on;` está documentada desde Nginx **1.25.1**: [documentación oficial](https://nginx.org/en/docs/http/ngx_http_v2_module.html).

## Privacidad y licencia

No se publican IP internas, registros de acceso sin anonimizar, rutas privadas de administración, credenciales ni enlaces al registro interno del incidente. No adjuntes capturas de red con información personal a los debates públicos.

Documentación bajo [CC BY 4.0](LICENSE).
