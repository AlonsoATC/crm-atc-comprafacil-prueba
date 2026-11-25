# Compra Fácil USA · CRM ATC

Panel privado de atención al cliente con registros persistentes y trazabilidad por cuenta autenticada.

**Versión actual: V9.5 · 24/11/2026.** Canal rojo tiene acceso directo y registro manual por OC con categoría propia, prioridad crítica y trazabilidad. DNI continúa registrándose manualmente a solicitud expresa de Compras; los avisos generales de la base central no crean solicitudes. Se mantienen las reglas de vigencia. [Vista simplificada para agentes](docs/VISTA_AGENTES_V9_4.md): resumen de Mi día con accesos directos, filas compactas, fichas con detalles e historial desplegables y diseño adaptable a móvil. Mantiene la regla de DNI por solicitud expresa de Compras de V9.3, la trazabilidad multianual y los accesos privados vigentes. La API real y la fusión con el CRM central continúan pendientes. Las secciones por versión al final documentan la evolución.

## Operación

- **Mi día** muestra los casos vigentes que requieren contacto. Se recalcula al abrir, al volver a la pestaña y cada minuto mientras está visible.
- El radar toma pedidos creados en los últimos 7 días o con fecha límite entre 7 días vencidos y 2 días futuros, siempre con una señal activa.
- Entregados, recibidos y anulados quedan excluidos. Los pedidos históricos sin vigencia tampoco generan tareas.
- Los recojos son responsabilidad de Logística y no generan alertas ni tareas de ATC. Los casos antiguos de recojo quedan archivados para consulta, conservando su historial.
- Las señales generales de una orden se agrupan en un caso. Una nueva señal puede reabrirlo; una importación idéntica conserva el estado atendido o cerrado.
- Atendido y cerrado salen de Mi día. Los seguimientos programados regresan en su día calendario en America/Lima.
- Cerrar requiere un motivo estructurado y una observación. Se guarda agente autenticado, identificador de cuenta, fecha y motivo. La cuenta no se elige en un selector.
- Las garantías se registran expresamente y se consultan en su sección, incluso para pedidos entregados; no forman parte del radar proactivo.
- Saga Falabella utiliza el canal seller. El panel prepara borradores y no envía comunicaciones.

## Fuente y datos

La demostración usa datos ficticios y una base independiente del espacio de pedidos reales. Los registros se guardan en D1, con control de versión para impedir que dos acciones simultáneas sobrescriban cambios. El historial de gestiones es acumulativo.

La API de Compra Fácil aún no está conectada. La operación admite importación incremental CSV o JSON, con hasta 1000 pedidos por archivo y 1,5 MB. Supervisión revisa una comparación en servidor y confirma ese mismo lote antes de guardar. Un número de pedido por fila; para varios productos se utiliza `items` en JSON. Se actualiza por número de pedido, preservando el historial y las órdenes ausentes del archivo.

Cabeceras CSV: `NUMERO_PEDIDO`, `CLIENTE`, `CANAL_SELLER_ORIGEN`, `PRODUCTO`, `FECHA_PEDIDO`, `FECHA_LIMITE_ENVIO`, `ESTADO_RESUMEN`, `METODO_ENVIO`, `TIENE_INCIDENCIA`, `CANAL_ROJO`. Las fechas admiten YYYY-MM-DD o DD/MM/YYYY.

JSON normalizado: `number`, `customer`, `channel`, `createdAt`, `dueAt`, `deliveryMethod`, `red`, `incidence` e `items: [{id, product, state, quantity}]`. La importación valida todos los registros antes de guardar. Un archivo inválido no aplica cambios.

V8.5.1: la OC debe ser texto también en JSON (`"number": "00000123456"`). Los valores numéricos se rechazan para impedir pérdida de dígitos o ceros; no se intenta reconstruir códigos ya alterados por Excel. No se admiten duplicados por diferencias de mayúsculas/minúsculas. Para actualizar una orden se usa exactamente el código ya cargado.

## Acceso

Se conserva el proyecto y su audiencia privada. El servidor exige identidad de ChatGPT en cada operación. El usuario local simulado existe únicamente en desarrollo y no está en el Worker de producción. Ningún token se incluye en el código cliente.

V9 añade permisos operativos de Agente y Supervisor comprobados en servidor. Agente gestiona casos; Supervisión administra importaciones, conexión, respaldos, exportación, revisión de Compras y roles. El propietario se vincula una sola vez al ID autenticado usando `CRM_OWNER_EMAIL` como arranque. El acceso nativo al sitio y el rol operativo son controles distintos; asignar un rol no invita personas.

## Validación

- `npm test`: reglas, calendario de Lima, exclusiones, seguimiento, trazabilidad, deduplicación y CSV.
- `npm run typecheck`: tipos.
- Build mediante el flujo de Sites.
- `node tests/api-integration.mjs`: tras el build, ejecuta el Worker en una base efímera y prueba identidad obligatoria, importación, concurrencia, cierres con otro agente, historial, espacios separados y origen de las peticiones.
- `node tests/pilot-integration.mjs`: 12 escenarios completos con 12 pedidos ficticios y un lote adicional de 1000, en Worker/D1 efímero. Incluye rechazos sin cambios parciales, DNI por producto, cierre por otro agente y exportación estable. No usa la URL publicada ni envía mensajes.
- `node tests/readiness-integration.mjs`: permisos efectivos, importación revisada, versiones de origen, revisión de Compras, respaldo/restauración con auditoría preservada y conector paginado GET/POST contra una API simulada. No equivale a validar la API de la empresa.
- `node tests/years-integration.mjs`: archivo multianual con 1105 pedidos y más de 10000 eventos en D1 efímero; prueba paginación, años de Lima, OCs reutilizadas y consultas históricas.
- Prueba de navegador: atención, cierre, persistencia tras recarga y programación de seguimiento.

Referencia funcional: `CRM_ATC_CompraFacil_V7_6_TrazabilidadAgentes(2).txt`. La base del panel previo se conserva en el historial de Git.


## Ampliación V8.1

- Fichas con categoría, responsable, prioridad manual, canal preferido y referencias de caso/conversación del CRM central.
- Reasignación con motivo e identidad del ejecutor. Atender un caso asignado a otra persona conserva su responsable; el ejecutor queda registrado en cada gestión.
- Intento sin respuesta con canal, fecha futura, dependencia y próximo paso obligatorios.
- Seguimientos vencidos, de hoy y próximos, incluyendo garantías registradas expresamente.
- Equipo y control con cargas por responsable y motivos de cierre.
- Laboratorio de seis escenarios ficticios, de carga incremental e idempotente. Solo disponible en Demostración.
- Exportación autenticada y versionada `cf-atc/1.0` para una futura integración. No existe conexión automática con el CRM central.

Ver `docs/INTEGRACION_CRM_ATC.md` para el contrato propuesto y las responsabilidades de cada sistema. Las cuentas visibles en Equipo son cuentas que ya ingresaron o perfiles ficticios; la asignación no concede acceso al sitio.

Todo registro manual (consulta, seguimiento ATC o garantía) requiere ingresar el número completo de orden de compra y validarlo para revisar cliente, producto y estado antes de crear el caso. El servidor resuelve la orden por ese número, rechaza IDs discordantes y nunca elige el primer pedido por defecto. Si la orden no existe en el espacio, se solicita importarla antes de continuar.

Los códigos de OC se tratan como texto. Se admiten formatos como CFPE194636, v18433394plzv-01, 1674564935500-01 y #2000019578082614, sin convertirlos a números, retirar símbolos ni usar coincidencias parciales. No hay una orden preseleccionada.

## V8.4 · colaboración y operación

- Persistencia separada en `crm_orders`, `crm_cases`, `crm_agents` y `crm_store_meta`. La versión de la ficha y de su pedido permite gestionar casos distintos simultáneamente; la edición del mismo caso o un cambio en su fuente exige revisar la ficha. Importaciones, altas y reinicios conservan un control global de snapshot.
- Migración de datos transaccional e idempotente al primer acceso a cada espacio. La fila anterior de `crm_datasets` queda como checkpoint, sin eliminarla ni reescribirla. Las migraciones SQL solo crean el esquema.
- La reserva de formularios DNI es única por espacio en D1 y comparte la transacción con el caso y la auditoría. Un conflicto revierte toda la gestión.
- Vistas rápidas para pendientes propios, sin responsable, urgencias, seguimientos de hoy y DNI.
- Tablero: pendiente, esperando al cliente, esperando a otra área, atendido y cerrado. Las tarjetas abren la ficha; no cambian estados sin una gestión documentada.
- Plantillas de seguimiento y contacto con la OC completa, cliente y producto. Seller usa su canal; DNI mantiene exclusivamente el mensaje y el formulario confirmado por producto.
- Reportes CSV con periodo inclusivo en calendario de Lima. Atenciones y cierres se cuentan por evento y ejecutor; la carga actual se cuenta por responsable. Recepciones parciales de DNI no cuentan como cierre de toda la solicitud.
- Garantías con tipo de producto y fechas de entrega/reporte. Referencia de 30 días para reacondicionados; fechas desconocidas quedan por confirmar y el sistema nunca decide cobertura automáticamente.
- La exportación `cf-atc/1.0` conserva sus campos previos y añade opcionalmente `caseRevision` y `warranty`. Las referencias centrales existentes se mantienen. API de pedidos, login TI, SLA y presencia en vivo siguen pendientes.

Validación adicional: `node tests/storage-integration.mjs` comprueba backfill simultáneo, conservación de historial, conflictos por caso, rollback de formulario duplicado, protección de pedido cambiado y checkpoint inalterado, en D1 efímero.

## V8.5 · ficha única por OC y atención previa

`Ficha por OC` y `Buscar OC` reúnen la orden por su código completo, productos/estados, casos ATC, garantías, solicitudes DNI y una cronología común. La consulta autenticada `/api/order?mode=demo|live&number=...` no crea casos ni marca gestiones. No se agrupan órdenes por nombre del cliente ni por coincidencias parciales; se preservan #, prefijos, guiones y ceros iniciales.

La ficha muestra quién realizó el último contacto o intento registrado, quién gestiona los casos abiertos y sus próximos pasos. Las notas internas y el guardado de enlaces no se presentan como un contacto al cliente. Las fichas de atención, el registro manual y el flujo individual DNI muestran el aviso y permiten revisar el contexto completo.

Si ya existe un caso abierto del mismo tipo (consulta manual o garantía), se exige confirmar que corresponde otro caso distinto; el servidor valida la confirmación y la registra en el historial. Se pueden consultar entregados y casos antiguos sin reactivarlos: no se habilita un nuevo caso ATC para pedidos fuera de vigencia ni una solicitud DNI para finalizados. Las garantías expresas mantienen su recorrido.

El resumen DNI solo incluye metadatos por producto, sin URLs ni tokens. Los enlaces permanecen en su ficha protegida de DNI. La audiencia y las reglas de acceso no cambian. No hay migraciones de base de datos en V8.5.

Resultados del piloto y opciones pendientes: [PRUEBAS_Y_OPCIONES_V8_5_1.md](docs/PRUEBAS_Y_OPCIONES_V8_5_1.md).

## V8.6 · espacio de trabajo para agentes

Cinco accesos diarios: Mi día, Casos, Seguimientos, DNI / importación y Garantías. Las consultas, el análisis y las herramientas se agrupan en menús desplegables. El buscador por OC y el registro de casos quedan en la cabecera. Mi día usa una sola bandeja amplia, con filtros que se abren bajo demanda y un aviso del número de filtros activos. Los reportes mantienen sus métricas en su propia sección.

DNI muestra un resumen compacto e instrucciones desplegables; sus formularios, mensajes oficiales y comprobaciones por producto conservan el flujo anterior. En pantalla estrecha la navegación se abre como menú y las filas operativas se presentan de forma vertical. La reorganización es de interfaz: no modifica datos, permisos, contratos de integración ni reglas de contacto.

Ver [ESPACIO_DE_TRABAJO_V8_6.md](docs/ESPACIO_DE_TRABAJO_V8_6.md).

## V9.0 · operación y preparación de la API

- Fecha de carga y modificación de origen por OC. Aviso de antigüedad configurable de 1 a 168 horas; los registros antiguos sin fecha se muestran como desconocidos.
- Importación con comparación antes de guardar, revisión global y protección contra versiones antiguas. Un refresco idéntico solo actualiza la procedencia; no invalida la ficha del agente.
- Revisión de Compras por formulario/producto: pendiente, validado u observado, con agente, fecha y referencia. Una observación reabre solo el producto afectado si el pedido sigue activo.
- Respaldos completos con auditoría, comprobación SHA-256, carga de archivo, revisión antes de restaurar y copia previa automática dentro de la misma transacción. Los permisos y las credenciales quedan fuera de la restauración.
- Conector de lectura de pedidos con URL y credencial exclusivamente en servidor, rutas de campos, equivalencias de estados, cursor y GET o POST de consulta. Hasta 1000 pedidos y 20 páginas por consulta; sin redirecciones ni escrituras parciales.
- Herramientas de Supervisión agrupadas; los cinco accesos diarios de los agentes se mantienen. Piloto ficticio de 12 órdenes revisable desde Conexión de pedidos.
- Migración aditiva `drizzle/0003_far_starbolt.sql`: controles, miembros operativos y respaldos fragmentados. No contiene datos ni modifica registros de pedidos.

Faltan la URL/método de consulta, credencial de lectura y respuesta real acordada con TI; después se deben probar estados, paginación, permisos y un piloto real. La actualización desde API se ejecuta por Supervisión con revisión y confirmación; no hay sincronización programada, webhook DNI ni envío automático de mensajes. Para grandes volúmenes queda pendiente paginar la lectura operativa completa y medirla en alojamiento.
