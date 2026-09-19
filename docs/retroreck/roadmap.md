# Prioridades y roadmap de implementación

Fecha: 2026-09-19. Estado: plan de trabajo; solo fork, investigación y documentación completados.

Para iniciar una sesión de implementación, seguir [arranque y primeras pruebas](arranque.md): preflight, E0, spike E1a y formato de evidencia. No crear un segundo listado de casos de prueba.

## 1. Prioridades

| Prioridad | Resultado | Regla de salida |
|---|---|---|
| **P0 — Base y autoridad** | Motor reproducible, identidad, aislamiento de tenants, administradores y superadministradores | Debe funcionar antes de habilitar comunidad externa |
| **P1 — Experiencia completa** | Biblioteca privada, invitaciones, controles, espectadores, progreso y soporte integrado | Una partida completa con permisos y progreso verificables |
| **P2 — Operación y piloto** | Pool acotado, cola, recuperación, métricas, pruebas de carga y panel operativo | Capacidad y respuesta a fallos demostradas antes de lanzamiento |

P2 expresa orden de desarrollo, no opcionalidad: sin límites de recursos, recuperación y operación no se abre el piloto. El pool mínimo acotado se introduce en E3; su escalado y conciliación completa se cierran en E6.

El [sistema visual aprobado](frontend.md) es requisito transversal desde E1a: integrar su distribución y componentes al construir identidad, y aplicarlos a la consola en E1b/E1c. El diseño existe; no se planifica un rediseño.

## 2. Etapas con resultado demostrable

| Etapa | Prioridad | Entregables | Demostración de aceptación |
|---|---|---|---|
| E0 — Motor reproducible | P0 | Toolchain/imágenes/cores fijados; build y prueba upstream | Juego autorizado con video, audio, dos clientes y save/load |
| E1a — Identidad y tenants | P0 | Spike de framework, registro/verificación, recuperación, invitado, espacio personal y sesiones | Dos usuarios no acceden a recursos ajenos; invitado no crea tenant ni sala |
| E1b — Autoridad administrativa | P0 | Roles globales, perfiles, adaptador Admin, bootstrap del superadministrador, 2FA, revocación y auditoría | Superadmin gestiona administradores; admin no escala privilegios ni modifica personal |
| E1c — Consola inicial | P0 | Directorio paginado, ficha de cuenta/capacidades, casos de soporte y gestión del equipo | Administrador atiende un caso; superadmin cambia su perfil y se ve el efecto inmediato |
| E2 — Lobby e invitaciones | P1 | Invitación dirigida y externa, cupos, reservas, vencimiento y revocación | Amigo registrado e invitado temporal entran con permisos correctos |
| E3 — Partida privada | P1 | Biblioteca y su vista administrativa, ROM autorizada/cache, asignación exclusiva y tickets | Host juega; invitado no descarga su ROM; solo una ejecución por sala |
| E4 — Controles y espectadores | P1 | Asignación del host, transferencia, medios para espectadores y validación de inputs | Dos jugadores y un espectador; ceder control no deja botones presionados |
| E5 — Progreso y acceso vivo | P1 | Save/SRAM durable, autosave/cierre, gracia/reconexión, vista de saves y sanciones al motor | Restauración en otro worker; acceso revocado no sobrevive a reconexión |
| E6 — Pool y operación | P2 | Cola/reposición, máximo de workers, conciliación, drenaje, TURN y vista operativa | Sin doble asignación; reducir pool no termina salas activas; fallos reconciliados |
| E7 — Piloto controlado | P2 | Matriz de juego/navegador, carga, costos medidos y runbooks de soporte/recuperación | Equipo administra comunidad y responde a fallos con datos y procedimientos probados |

E1a prueba identidad y contratos con fixtures; el canje real se entrega en E2, el control de espectadores en E4 y la revocación contra juego vivo en E5. E1a–E1c pueden avanzar sin que el motor esté disponible; E3 requiere E0–E2. E4 depende de E3; E5, de E3/E4; E6 completa el ciclo de operación sobre E3/E5. E7 exige todos los anteriores.

La consola inicial muestra cuenta, permisos, cuotas y soporte. Las pestañas de biblioteca, guardados y partidas se implementan con sus respectivos módulos; no se presentan datos simulados como funcionalidad terminada.

## 3. Backlog inicial, en orden de ejecución

| Orden | Trabajo acotado | Entrega revisable | Condición para darlo por hecho |
|---|---|---|---|
| 1 | Entorno del motor | Configuración fijada y guía de ejecución | Build limpio y partida upstream demostrada |
| 2 | Base de API de producto | NestJS/Express, PostgreSQL/Prisma, configuración y CI | Arranque local y migraciones reproducibles |
| 3 | Integración de identidad | Better Auth con flujos elegidos y pruebas | Registro, recuperación, invitado y revocación reales |
| 4 | Propiedad y aislamiento | Tenant personal idempotente, consultas con ámbito | Pruebas cruzadas de dos tenants y rol invitado |
| 5 | Política del equipo | Superadmin/admin, capacidades registradas y perfiles | API deniega promoción propia y modificación de personal por admin |
| 6 | Provisionar y administrar personal | Bootstrap, invitación verificada, doble factor y revocación | Crear, editar, limitar y desactivar admin desde flujo probado |
| 7 | Auditoría y casos | Registro durable y contexto de soporte | Cada acción administrativa identifica actor, objetivo y resultado |
| 8 | Panel inicial | Usuarios, ficha, capacidades, casos y equipo | Caso de soporte resuelto; permisos del panel coinciden con servidor |
| 9 | Lobby e invitaciones | Contratos, reserva atómica y UI de entrada | Reintentos y concurrencia no duplican puestos ni participantes |
| 10 | Corte vertical jugable | Biblioteca privada, worker y tickets (E3), control de puestos y espectadores (E4) | Host + amigo + invitado con autoridad de motor comprobada; antes de E4 el corte se limita al laboratorio interno |

La política de auditoría se define en el trabajo 5 y se conecta desde las primeras operaciones; el trabajo 7 completa persistencia y consulta. No se liberan mutaciones de personal sin su registro correspondiente.

## 4. Criterios transversales de cada entrega

- Cambios pequeños con contrato, migración y prueba de comportamiento cuando corresponda.
- Toda UI sigue el sistema visual único, con evidencia de estados, adaptación y accesibilidad; incluye las pantallas administrativas.
- Fuente única para identidad y autoridad administrativa; cambios de permisos afectan sesiones abiertas.
- Las rutas del framework no pueden evitar las restricciones de las rutas propias.
- Soporte puede revisar capacidades e inventario autorizado de una cuenta; ver secretos no es parte de ninguna ficha.
- Operaciones concurrentes de reserva, permisos y cierre conservan sus invariantes.
- El panel distingue estado solicitado, aplicado, fallido y pendiente; no muestra éxito por una simple petición enviada.
- Todo límite o estimación de capacidad permanece identificado como provisional hasta medirse.

## 5. Fuera del primer lanzamiento

Varias partidas por proceso worker, transferencia de propiedad del host, organizaciones empresariales con SSO, directorio público de salas y acceso de soporte por suplantación de usuario. La administración global de usuarios y equipo **sí** forma parte del primer lanzamiento.

## 6. Punto de partida y próximas decisiones

El fork y los planes están publicados en una propuesta en borrador; no hay módulos de producto implementados ni cuentas administrativas creadas. El siguiente lote de implementación es **E0 + E1a**, seguido inmediatamente por **E1b/E1c**. La selección final del framework se cierra tras el spike, preservando los requisitos administrativos aunque cambie la biblioteca elegida.

Antes del piloto se fijarán operador inicial, proveedor de correo, infraestructura, cores autorizados y presupuesto del pool. No se requieren esas decisiones para continuar con el entorno local, contratos y pruebas. Las fechas de entrega se estimarán después de medir E0/E1a; este roadmap define dependencias y prioridades sin inventar plazos.
