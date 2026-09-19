# Administración global y soporte de la comunidad

Fecha: 2026-09-19. Estado: requisito confirmado y diseño propuesto; aún no implementado.

## 1. Alcance fundamental

RetroReck debe administrar desde un panel central a todos sus usuarios. Habrá dos roles de equipo claramente separados:

- **Administrador:** consulta usuarios, sus capacidades y los recursos de sus cuentas para brindar soporte; realiza las operaciones que tenga concedidas.
- **Superadministrador:** crea, modifica y desactiva cuentas administrativas; define los permisos de los administradores y controla el acceso del equipo.

Estos roles son globales a la plataforma. El propietario de un tenant y el anfitrión de una sala siguen siendo usuarios del producto; esos papeles no conceden permisos administrativos. No se exige un tercer rol de moderación para lanzar: un administrador puede recibir un perfil limitado de soporte.

La identidad y el control de administradores forman parte de **E1, antes de abrir el producto a usuarios externos**. Las pantallas de partidas y workers se amplían al implementar esos servicios, manteniendo las mismas reglas de acceso.

## 2. Tenants e infraestructura compartida

Un tenant es un espacio lógico de propiedad: biblioteca, archivos, guardados, plan, cuota y consumo. Los usuarios comparten servicios y una base de datos inicial, con referencias de tenant, restricciones e índices; no se crea un servidor o una base por registro.

La API de usuarios opera siempre sobre un ámbito autorizado. La API administrativa puede consultar distintos tenants cuando el permiso lo autoriza. Cada consulta verifica identidad del operador, permiso, recurso objetivo y estado de su cuenta. No basta con enviar otro identificador de tenant desde el navegador.

El acceso administrativo global no convierte al operador en dueño de la cuenta consultada. Las acciones conservan separado quién actúa y sobre qué usuario/tenant actúa. El historial y las cuotas siguen perteneciendo al usuario original.

## 3. Matriz de permisos inicial

| Operación | Usuario del producto | Administrador | Superadministrador |
|---|---|---|---|
| Consultar perfil y recursos propios | Sí | Como usuario, si tiene espacio personal | Como usuario, si tiene espacio personal |
| Buscar usuarios en la comunidad | No | Según perfil asignado | Sí |
| Ver ficha de usuario, capacidades y recursos | Solo lo propio | Según perfil asignado | Sí, con auditoría |
| Ver sesiones y ayudar a recuperar acceso | Solo lo propio | Permiso de soporte | Sí |
| Suspender/reactivar usuarios ordinarios | No | Permiso específico | Sí |
| Ajustar cuota o restricciones de cuenta | No | Permiso específico y motivo | Sí |
| Cerrar una partida por soporte | Su sala | Permiso específico; disponible desde integración del motor | Sí, bajo el mismo procedimiento |
| Crear/modificar/desactivar administradores | No | No | Sí |
| Definir perfiles y conceder permisos administrativos | No | No | Sí |
| Cambiar roles o permisos de su propia cuenta administrativa | No | No | Solo operaciones permitidas, preservando el último superadministrador activo |
| Consultar auditoría global | No | Solo si tiene permiso de auditoría | Sí |

Propuesta: un administrador nuevo recibe un perfil mínimo de consulta/soporte; suspensiones, cuotas y cierre de partidas se conceden expresamente. No puede modificar a otro administrador ni a un superadministrador mediante endpoints genéricos de usuarios, aunque tenga permiso para modificar usuarios ordinarios.

El acceso administrativo exige segundo factor satisfecho en la sesión actual, no solo activado en el perfil. Las sesiones previas a una promoción o abiertas por otra vía de login no heredan esa comprobación. Ver la [política de métodos de acceso](identidad.md#segundo-factor-y-métodos-de-acceso); probar OAuth, recuperación y retirada de 2FA para evitar rutas alternativas.

Los superadministradores también se autentican, usan doble factor y dejan auditoría. No se representan con una clave secreta compartida ni con una ruta pública de creación de cuentas privilegiadas.

## 4. Ficha de soporte: qué podrá revisar el administrador

El panel reunirá información paginada y filtrada por los permisos efectivos del operador:

| Sección | Información y acciones previstas |
|---|---|
| Cuenta | Perfil, correo, verificación, estado, fechas de alta/actividad y restricciones |
| Capacidades del usuario | Plan, cuotas, consumo, límites de salas/participantes y explicación de por qué una acción está permitida o denegada |
| Biblioteca | Inventario de juegos/ROMs, sistema, tamaño, integridad y estado de procesamiento |
| Guardados | Juego, slot, fecha, versión/core, tamaño y resultado de sincronización; errores de recuperación |
| Salas y sesiones | Participantes y roles, estado, errores recientes y vinculación con worker cuando exista esa integración |
| Soporte | Caso, motivo de consulta, notas del equipo y acciones realizadas |
| Acceso | Sesiones de autenticación resumidas y revocación autorizada; iniciar recuperación por el flujo del framework |

Consultar inventario permite saber qué tiene la cuenta sin exponer automáticamente descargas de ROMs o guardados. Si diagnosticar un caso requiere inspección de contenido, utilizar un permiso separado, ámbito limitado al recurso/caso y registro de acceso. Las respuestas del panel no incluyen contraseñas, hashes, cookies, tokens ni claves de storage.

En la primera versión no se necesita hacerse pasar por el usuario: la ficha y las operaciones administrativas conservan la identidad del operador. Recuperar una contraseña utiliza el flujo verificado del framework; el panel nunca muestra la contraseña actual.

## 5. Panel del superadministrador

Debe permitir:

1. Crear una invitación de personal administrativo vinculada a su destinatario y perfil; no enviar contraseñas en claro. La aceptación exige verificación y alta de doble factor antes de habilitar funciones de equipo.
2. Consultar administradores, estado de acceso, perfiles efectivos y actividad administrativa.
3. Modificar datos permitidos, asignar/quitar perfiles y suspender/reactivar una cuenta administrativa.
4. Crear y editar perfiles de permisos combinando un catálogo de capacidades implementadas: consulta de usuarios, inventario, soporte, revocación de sesiones, suspensión, cuotas y auditoría.
5. Revocar sesiones y permisos de personal inmediatamente; una pestaña abierta no conserva autoridad retirada.
6. Consultar quién concedió o retiró cada permiso y el resultado de la operación.

Las capacidades nuevas requieren implementación y pruebas; crear un perfil desde el panel solo combina capacidades existentes. Los permisos que permiten administrar personal son exclusivos de superadministradores y no pueden introducirse indirectamente en un perfil de soporte.

El primer superadministrador se provisiona mediante un procedimiento de despliegue autenticado, único e idempotente. La identidad exacta se configura al desplegar. No hay credenciales predeterminadas. Impedir desactivar, borrar o degradar al último superadministrador activo, incluso mediante dos solicitudes concurrentes. Definir y ensayar un procedimiento de recuperación administrativa antes del piloto.

## 6. Framework y frontera de autorización

Better Auth ofrece un [plugin Admin](https://better-auth.com/docs/plugins/admin) con consulta de usuarios, roles, bloqueo/desbloqueo y revocación de sesiones, además de permisos personalizados. Se reutilizan esas operaciones para identidad. El plugin no entrega por sí solo el panel de comunidad ni los permisos de biblioteca, cuotas, salas y workers de RetroReck.

Los roles predeterminados del plugin no sustituyen la matriz anterior. Su administrador predeterminado tiene facultades amplias; un administrador de soporte de RetroReck no debe recibir automáticamente ese conjunto. La jerarquía de cuentas objetivo y la reserva de facultades del superadministrador se comprueban en servidor.

Diseño propuesto:

- `StaffAccessService` será la autoridad de las asignaciones/perfiles de equipo en PostgreSQL. Mantendrá una versión de autorización por operador; las operaciones sensibles comprobarán permisos vigentes, sin confiar solo en claims antiguos del navegador.
- Un adaptador encapsulará las operaciones del plugin. Las rutas nativas administrativas del framework deben tener restricciones equivalentes o no estar expuestas directamente; no dejar un endpoint alternativo que evite las comprobaciones de RetroReck.
- Si se refleja un rol en Better Auth por compatibilidad, será un dato derivado de la asignación canónica, no otra fuente independiente editable. Solo el servicio autorizado podrá cambiarlo.
- Los permisos combinan acción y objetivo: editar un usuario ordinario nunca implica editar una cuenta de equipo. La validación se comparte entre API, procesos internos y acciones del panel.
- Suspender al usuario o retirar acceso a una sala revoca sesiones/tickets y envía una orden al motor con versión/generación; la interfaz muestra su confirmación. Suspender un administrador invalida sus sesiones, impide nuevas operaciones y obliga a reevaluar las órdenes privilegiadas pendientes que aún no se hayan aplicado.
- Las lecturas de ficha/inventario y las modificaciones administrativas generan auditoría: operador, objetivo, acción, caso/motivo, instante, resultado y cambios no secretos. Si una mutación requiere auditoría y no puede registrarse de forma durable, no se confirma silenciosamente como completada.

## 7. Modelo de datos adicional

| Entidad conceptual | Responsabilidad |
|---|---|
| Asignación de equipo | Identidad, rol global, estado, versión y quién lo concedió |
| Perfil de permisos / concesión | Catálogo conocido, perfiles versionados y asignaciones; validar cambios en transacción |
| Invitación administrativa | Destinatario, emisor superadministrador, perfil, hash del secreto, expiración y canje único |
| Caso de soporte | Usuario/tenant, motivo, estado, operador y notas |
| Evento administrativo | Actor y objetivo separados, acción, versión, resultado y correlación con operación interna |

La lista de usuarios usa búsqueda/paginación del framework donde corresponda; biblioteca y consumo usan consultas paginadas e índices de producto. Exportaciones o tareas masivas requieren permiso específico y ejecución asíncrona, no cargar toda la comunidad en el navegador. Las vistas administrativas no evitan el control de autorización por usar conexiones de base de datos privilegiadas.

## 8. Criterios de aceptación obligatorios

1. Un usuario normal, invitado o dueño de tenant no accede a la API administrativa ni enumerando rutas directamente.
2. Un administrador consulta usuarios/inventario conforme a su perfil y puede explicar restricciones de cuenta desde la ficha.
3. Un administrador no se promueve, no cambia perfiles y no modifica a personal privilegiado a través de rutas de usuario genéricas o del plugin.
4. Un superadministrador crea y modifica un administrador, ajusta sus permisos y lo desactiva. Las sesiones abiertas pierden acceso tras la revocación.
5. La aceptación de invitación administrativa verifica destinatario, expiración, uso único y segundo factor. Las sesiones previas o de otros métodos de acceso no ejercen autoridad sin satisfacer también el segundo factor vigente.
6. Se preserva al último superadministrador activo con acciones concurrentes; se prueba la recuperación.
7. Las consultas y cambios auditados identifican al operador real y no contienen secretos ni contenido innecesario de archivos.
8. Un permiso de inventario no habilita descarga de contenido; un acceso de diagnóstico autorizado queda limitado y registrado.
9. Desde E5/E6, suspender una cuenta termina o restringe su acceso vivo al juego conforme a la política, con confirmación o estado pendiente visible.
10. Listas de usuarios, recursos y auditoría siguen siendo paginadas y medibles con el volumen de datos del piloto.

El [roadmap](./roadmap.md) integra estos entregables en el orden de implementación. No se crean cuentas administrativas reales ni se activan permisos con esta actualización documental.

## Sistema visual de la consola

La consola de usuarios, soporte y gestión del equipo se implementa con el [mismo sistema visual aprobado](frontend.md) del producto desde E1b/E1c. No se adopta otro tema administrativo. Las pantallas sin mockup extienden los componentes y tokens existentes, con revisión visual y permisos verificados contra la API.
