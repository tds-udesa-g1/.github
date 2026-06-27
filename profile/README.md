# Documento de Retrospectiva y Cierre de Proyecto

**Fecha:** 27/06/2026  
**Versión:** 3.0.0  
**Autores:** Giacometti, Mateo - Martín Bernal, Tiziano Levi - Pettinari, Fausto Julián - Torres, Santiago Tomás

---

## 1. Funcionalidades No Implementadas
*Detalle de aquellas características, módulos o integraciones que fueron planificadas o debatidas durante el ciclo de desarrollo, pero que fueron excluidas del alcance final por limitaciones de tiempo o priorización.*

* **Social Log In:** Consistía en permitir el inicio de sesión y registro mediante proveedores externos. El alcance incluía la creación automática de perfiles, la vinculación con correos electrónicos previamente registrados y la gestión de casos borde (como bloquear el flujo de "olvidé mi contraseña" para cuentas exclusivamente sociales).
* **Historial de chat de solo lectura con usuarios bloqueados:** La funcionalidad preveía que, al bloquear a un contacto, la conversación previa se mantuviera visible en la lista de chats pero inhabilitada para el envío de nuevos mensajes. Debido a que el sistema de bloqueos se desarrolló mucho antes que el módulo de mensajería, este criterio de integración fue omitido en la interfaz final (que actualmente filtra y muestra únicamente los chats con amigos activos). A nivel de base de datos los mensajes se conservan de forma correcta, pero se debería cambiar la forma en que se buscan los usuarios a que aparecen en la página que nos mustra nuestra lista de chats.
* **Sistema de Recomendación de Usuarios (IA):** Consistía en un motor de sugerencias de amistad basado en el análisis algorítmico de las biografías de los perfiles. Mediante técnicas de procesamiento de lenguaje natural (NLP), el sistema evaluaba los intereses compartidos y descripciones afines para calcular un "índice de similitud semántica" (similarity score) entre usuarios, recomendando proactivamente a aquellos con mayor afinidad.
* **Renderizado de Fotos de Perfil en Pines del Mapa:** Consistía en reemplazar los marcadores genéricos de ubicación en el mapa interactivo por la miniatura de la foto de perfil de cada amigo.

## 2. Análisis de Casos Borde y Posibles Riesgos
*Identificación de escenarios atípicos, flujos alternativos o condiciones extremas que no están completamente cubiertas por la implementación actual y que requieren atención en futuras iteraciones.*

* **Condiciones de Carrera (WebSockets vs. Bloqueo de Usuarios):** Riesgo de "Notificaciones Fantasma" y almacenamiento de datos no deseados. Actualmente, la validación de estado de una amistad ocurre únicamente en el handshake inicial de la conexión WebSocket. Si el Usuario A bloquea al Usuario B exactamente mientras B tiene el chat abierto, el túnel de B permanece activo. Esto permite que B siga guardando mensajes en la base de datos y disparando notificaciones Push hacia A, las cuales, al ser abiertas, llevarán a una vista vacía ya que la relación de amistad fue destruida. Requiere implementar validación continua por mensaje o cierre forzado del socket por eventos.

## 3. Deuda Técnica
*Registro de los atajos tomados a nivel de código, arquitectura o infraestructura que permitieron acelerar el desarrollo a corto plazo, pero que requerirán una refactorización o corrección futura para asegurar la mantenibilidad.*

* **Intermediario Web para Enlaces de Correo:** Refactorizar el flujo de enlaces en correos electrónicos mediante la implementación de una página web intermediaria (vía HTTPS). Actualmente se utiliza el esquema personalizado de la aplicación de forma directa, el cual es bloqueado proactivamente por los filtros de seguridad de clientes como Gmail.
* **Paginación y Escalabilidad en la Lista de Chats:** Implementar un mecanismo de paginación o scroll infinito en la pantalla de "Lista de Chats". En su estado actual, la vista está limitada a renderizar únicamente los 20 amigos con los que el usuario interactuó más recientemente, impidiendo el acceso a conversaciones más antiguas si se supera dicho límite.
* **Observabilidad de Logs en Grafana Cloud:** Se implementó exitosamente la observabilidad mediante OpenTelemetry para traces automáticos, métricas automáticas y métricas de negocio, todas visibles en Grafana Cloud. Como mejora adicional se intentó incorporar la recolección centralizada de logs mediante un segundo OpenTelemetry Collector desplegado como DaemonSet, encargado de capturar los logs de stdout/stderr de todos los pods y enviarlos a Grafana Loki. Si bien la implementación fue funcional a nivel de configuración y el collector logró procesar correctamente los logs en los nodos donde pudo ejecutarse, el despliegue no pudo completarse debido a limitaciones de la infraestructura provista por la cátedra (cuotas de recursos y restricciones de scheduling del cluster), que impedían ejecutar un collector en cada nodo del clúster. Dado que el resto de la plataforma de observabilidad ya se encontraba completamente operativa y estable, se decidió revertir esta implementación y mantener los logs estructurados accesibles mediante kubectl logs, dejando la integración completa con Loki como una mejora futura en un entorno con mayores recursos o permisos de infraestructura.

## 4. Lecciones Aprendidas

* La división en microservicios facilitó el trabajo en paralelo sin generar conflictos en los repositorios.
* La decisión de priorizar el estado local de la aplicación combinado con peticiones estratégicas al servidor (mediante useFocusEffect y Pull-to-Refresh) resultó ser un excelente balance para la experiencia de usuario (UX). Evitó el sobre-diseño y el alto consumo de recursos en el servidor que hubiera implicado el uso excesivo de WebSockets para pantallas estáticas.
* Automatización de Integración Continua (CI): La configuración de pipelines para la ejecución automática de tests y validación de cobertura de código actuó como una barrera de seguridad muy importante. Esta práctica nos garantizó que ningún código defectuoso o sin probar pudiera ser integrado (mergeado) a la rama principal.
