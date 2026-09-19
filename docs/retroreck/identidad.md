# Identidad, contraseñas y aislamiento de usuarios

Investigación: 2026-09-19, documentación oficial. Estado: recomendación para el primer spike; ningún framework instalado todavía.

## 1. Recomendación

**NestJS con adaptador Express + Better Auth + PostgreSQL + Prisma** para el servicio de producto; mantener Go/Libretro/WebRTC en el fork para ejecución.

Better Auth resolvería cuentas, credenciales, sesiones y recuperación. NestJS organizaría los módulos propios de RetroReck. PostgreSQL conservaría datos y restricciones; Prisma gestionaría acceso y migraciones revisables. No necesitamos crear un sistema propio de contraseñas ni modificar los algoritmos del proveedor.

Esta elección prioriza código TypeScript local, configuración versionada, módulos pequeños, contratos tipados y pruebas reproducibles. Son características que facilitan tanto el trabajo humano como el trabajo asistido por agentes; no eliminan la revisión de permisos ni las actualizaciones de seguridad.

## 2. Comparativa

| Opción | Capacidades verificadas | Ajuste a RetroReck y costo de integración |
|---|---|---|
| **Better Auth** | Email/contraseña, sesiones, recuperación; plugins de anónimos, organizaciones y 2FA | Encaja con invitados sin registro y backend TypeScript propio. Requiere implementar permisos de sala, aislamiento de recursos y operación de la API |
| **Supabase Auth** | Acceso anónimo convertible a cuenta, integración con PostgreSQL/RLS y self-hosting | Alternativa si priorizamos una plataforma integrada. Un anónimo recibe el rol de DB autenticado: sus permisos deben restringirse expresamente; limpiar identidades abandonadas requiere política propia |
| **Keycloak** | Servicio IAM con organizaciones, miembros, identidad federada y contexto multitenant | Candidato si aparecen SSO empresarial o varios productos. Mi valoración: añade una superficie operativa y de configuración mayor para este MVP; invitados de sala siguen siendo lógica del producto |
| **Insforge** | Email/contraseña, verificación, recuperación, OAuth/OIDC, sesiones JWT y RLS; también documenta integración con Better Auth | Mantenerlo es viable, pero la documentación consultada no establece por sí sola el flujo nativo completo de invitados/tenants que necesitamos. El puente con Better Auth añade otra frontera de identidad que habría que probar |

Fuentes de la comparación: [Better Auth: anónimos](https://better-auth.com/docs/plugins/anonymous), [organizaciones](https://better-auth.com/docs/plugins/organization), [2FA](https://better-auth.com/docs/plugins/2fa); [Supabase: anónimos](https://supabase.com/docs/guides/auth/auth-anonymous), [self-hosting](https://supabase.com/docs/guides/self-hosting); [Keycloak: administración y organizaciones](https://www.keycloak.org/docs/latest/server_admin/); [Insforge: autenticación](https://docs.insforge.dev/core-concepts/authentication/overview), [integración Better Auth](https://docs.insforge.dev/integrations/better-auth).

La valoración de ajuste/costo de integración es nuestra inferencia arquitectónica, no un benchmark entre productos. No se afirma que uno sea universalmente más seguro.

### Por qué no conservar Insforge por inercia

Todavía no hay una aplicación RetroReck implementada que migrar: lo existente es el plan. Los módulos de admisión, cuotas, puestos, leases y recuperación necesitan reglas propias incluso usando un BaaS. La propuesta concentra esas reglas en una API modular, con un solo proveedor de identidad.

Insforge puede reconsiderarse como infraestructura o mantenerse si el spike demuestra una ventaja concreta. Su integración con Better Auth usa un puente de tokens y tablas de identidad diferentes de su auth nativo; no basta con reutilizar las referencias antiguas a `auth.users`.

### Integración NestJS

NestJS ofrece estructura modular sobre Node/TypeScript. La integración NestJS documentada por Better Auth usa un paquete **mantenido por la comunidad** y señala soporte Fastify en beta. Por eso proponemos Express inicialmente y un spike que pruebe montaje del handler, cookies y guards. Si el adaptador añade fragilidad, montar el handler Node/Express documentado y un guard pequeño que consuma la API oficial de sesiones; no reimplementar la autenticación. Fuentes: [NestJS](https://docs.nestjs.com/), [integración NestJS](https://better-auth.com/docs/integrations/nestjs), [integración Express](https://better-auth.com/docs/integrations/express).

El adaptador Prisma está documentado por Better Auth; la generación de esquema y la aplicación de migraciones son pasos distintos. Revisar y versionar las migraciones con Prisma. [Fuente](https://better-auth.com/docs/adapters/prisma).

## 3. Qué significa tenant en RetroReck

Propuesta: al registrar una cuenta permanente se crea un **espacio personal propietario**. Ese tenant posee biblioteca, ROMs, guardados, cuota y consumo. Inicialmente tiene un solo dueño; no requiere una pantalla de «organizaciones» para el jugador.

Una sala pertenece al tenant de su anfitrión. Un amigo registrado conserva su propio tenant, pero al entrar recibe acceso exclusivamente a esa sala. Un invitado temporal no necesita tenant personal. **Invitar a jugar no equivale a añadir a alguien como miembro de la organización ni darle acceso a sus archivos.**

El plugin de organizaciones puede representar estos espacios y sus miembros permanentes. La creación del espacio personal debe ser exclusivamente de servidor e idempotente; invitados anónimos no pueden crear organizaciones. Los endpoints generales de invitar miembros de organización no se exponen como «invitar a jugar».

El aislamiento no lo garantiza automáticamente ese plugin: consultas, descargas, WebSockets y operaciones del motor deben comprobar tenant y permiso de sala. Para el MVP, la base no se expone directamente al navegador; repositorios de datos con ámbito obligatorio y tests cruzados. Si se introduce acceso directo o RLS, diseñar el rol de aplicación y la propagación segura del contexto; no asumir que Prisma activa RLS por sí solo.

## 4. Identidades y roles

| Dimensión | Valores | Significado |
|---|---|---|
| Identidad | Permanente / anónima | Cómo se autentica la persona |
| Propiedad | Dueño del tenant / visitante de sala | Qué recursos privados administra |
| Autoridad en sala | Anfitrión / participante | Quién invita, expulsa y asigna controles |
| Modo de participación | Jugador / espectador | Si puede enviar controles |
| Puesto | 0–3 o ninguno | Puerto interno; interfaz muestra Jugador 1–4 |

El anfitrión también puede observar y asignar todos los puestos a sus invitados. La autoridad de anfitrión no depende de ocupar el primer control.

Better Auth anónimo crea un registro interno y una sesión aunque no pida correo ni contraseña. Eso satisface «sin crear cuenta» en la experiencia del usuario, pero debe explicarse en el diseño de datos y retención. La admisión exige además una invitación válida; obtener sesión anónima no concede acceso a ninguna sala. [Fuente](https://better-auth.com/docs/plugins/anonymous).

## 5. Modelo de datos conceptual

| Entidad | Campos/relaciones principales | Invariante |
|---|---|---|
| Identidad del framework | Usuario, sesión, credenciales y verificaciones | Framework como única autoridad de credenciales |
| Tenant / membresía | Organización personal, dueño permanente | No conceder membresía por invitación a sala |
| ROM | Tenant, objeto privado, hash, sistema/core permitido | Solo propietario o permiso explícito de ejecución |
| Sala | Tenant, host, ROM, estado, límites | Separar lobby persistente de ejecución de la partida |
| Participante | Sala, identidad, modo, estado de admisión | Una identidad admitida por sala; apodo no identifica permisos |
| Invitación | Hash del secreto, sala, destinatario opcional, expiración, usos, puesto reservado | Canje atómico, destinatario verificado y revocación |
| Puesto | Sala, puerto, participante o reserva, versión | Un propietario/reserva por puerto; participante no ocupa dos puestos activos |
| Partida | Sala, worker, generación, estado, lease | Como máximo una ejecución activa por sala/worker |
| Guardado | Tenant, ROM/hash, core, slot, versión, resultado | Invitados no adquieren propiedad del progreso |
| Evento/auditoría | Operación, actor, sala, versión, resultado | Deduplicación y trazabilidad de cambios de autoridad |

Usar IDs de identidad como valores opacos compatibles con el framework; no asumir que todos son UUID. Las claves de producto pueden usar UUID separados.

## 6. Configuración y pruebas de identidad

Better Auth documenta hashing `scrypt`, sesiones revocables, cookies protegidas, comprobaciones de origen y mitigaciones CSRF. Mantener sus mecanismos y configurar dominios/proxies explícitos. Su documentación de email/contraseña señala que revocar otras sesiones al restablecer contraseña requiere activar esa opción. Fuentes: [seguridad](https://better-auth.com/docs/reference/security), [contraseñas y recuperación](https://better-auth.com/docs/authentication/email-password).

Configuración propuesta para validar en el spike:

- Registro con email y verificación antes de crear salas; recuperación con proveedor SMTP transaccional y pruebas en buzón local de desarrollo.
- Cookies de sesión HttpOnly/Secure bajo HTTPS, origen compartido mediante proxy y lista cerrada de redirecciones.
- Revocación de sesiones al recuperar contraseña; revocación de acceso vivo al motor al expulsar o cerrar la sala.
- Rate limits compartidos entre réplicas, políticas de creación de invitados y limpieza de identidades temporales sin sesiones/partidas activas.
- 2FA para administración; no activar SSO, SCIM, proveedor OAuth propio ni plugins de facturación sin necesidad.
- Sesión de navegador separada del ticket breve de juego. Ticket de un uso por conexión, ligado a audiencia, participante, sala, worker y generación; firma mediante biblioteca estándar. Nunca usar el enlace de invitación como credencial permanente del motor.
- Enlace externo con secreto aleatorio de alta entropía, almacenado como hash; no registrar ese secreto en logs. Un código corto de sala no es autorización suficiente.
- Cuenta registrada que acepta invitación dirigida debe coincidir con su destinatario. Un enlace externo puede permitir varios usos hasta el límite configurado; un enlace de puesto reservado es individual.

No se ha efectuado una auditoría integral del framework. Se revisaron avisos oficiales recientes: existen correcciones para flujos de magic-link/email OTP y SSO. La versión final debe fijarse con sus dependencias y comprobarse contra los avisos aplicables al configurar los plugins; no usar una versión antigua por copiar un ejemplo. Fuentes: [avisos del proyecto](https://github.com/better-auth/better-auth/security/advisories), [corrección de acceso por correo](https://github.com/better-auth/better-auth/security/advisories/GHSA-qq9h-g4jm-xgf3), [corrección del plugin SSO](https://github.com/better-auth/better-auth/security/advisories/GHSA-8c5h-wx78-2cfg).

### Salida exigida del spike

Registro, verificación, login/logout, recuperación, revocación y 2FA administrativo deben funcionar; un invitado debe entrar sin correo, mantener su sesión al recargar y quedar bloqueado tras expulsión. Probar dos tenants, invitación dirigida a tercero, replay del enlace consumido, escalamiento de espectador y creación concurrente del espacio personal. Comprobar aislamiento con llamadas directas a API, no solo con botones ocultos.

Entregar versiones fijadas, migraciones, configuración de ejemplo sin secretos, pruebas y una decisión final de adopción. La seguridad de RetroReck depende de esa integración además del framework elegido.
