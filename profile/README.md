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
