# Inicio de implementación y primeras pruebas

Revisión final: 19 de septiembre de 2026. Plan listo para iniciar trabajo local; **E0 y E1a no están ejecutados**. Este documento organiza el primer lote; el [roadmap](roadmap.md) conserva la secuencia completa y sus criterios de salida.

## 1. Estado de partida

- Fork: `reck74/retroreck-backend`; rama del plan: `codex/plan-backend-retroreck`. La propuesta sigue en borrador, sin fusionar. `master` todavía no contiene el plan.
- Base de motor analizada: `1fab9ef07a02cab0e0cdea169e95b5ec6e2e04ed`. Los cambios del fork respecto de esa base son documentación e instrucciones; el comportamiento del motor no ha sido modificado.
- Comprobado: análisis estático, estructura Compose, enlaces del plan. En el workspace de producto también están verificados tokens, originales y referencias gráficas.
- Pendiente: construir imágenes/binarios, ejecutar juegos y pruebas Go, instalar servicio de identidad, migrar base de producto y crear cuentas de prueba. No hay resultados de rendimiento ni capacidad medida.
- En el entorno revisado existen Docker, Node y Python; no hay `go` en PATH. Tener Docker CLI no demuestra disponibilidad del daemon. Volver a comprobar el entorno en la sesión de implementación.

## 2. Preparación de la sesión

1. Leer `AGENTS.md`, [plan](plan.md), [identidad](identidad.md), [administración](administracion.md), [workers](workers.md), [frontend](frontend.md) y [roadmap](roadmap.md). En el workspace completo, comenzar por su guía de inicio para localizar también el diseño aprobado.
2. Comprobar rama, remotos, cambios locales y último commit. Conservar el trabajo existente. Crear una rama `codex/` desde la revisión que contiene el plan; no comenzar desde `master` suponiendo que la propuesta ya está fusionada.
3. Registrar versiones y verificar daemon, espacio disponible y puertos. Elegir build nativo o contenedor reproducible. El Dockerfile menciona Go 1.27.1 y GStreamer 1.29.2; `go.mod` exige 1.26.0. Son valores del código, no disponibilidad demostrada. Verificar fuentes, compatibilidad y hashes antes de fijar la receta.
4. Preparar un juego de prueba y core con procedencia/licencia registradas. La presencia de demos en `assets/games/` no demuestra que cubran dos jugadores. Seleccionar un fixture válido para cada escenario; no descargar un catálogo comercial para probar.
5. Mantener el clon original como referencia. Los cambios de build/configuración RetroReck se documentan aparte y no se mezclan todavía con parches de permisos, saves o pool.

Comprobaciones iniciales desde la raíz del fork, sin iniciar servicios:

```bash
git status --short
git branch --show-current
git remote -v
git rev-parse HEAD
docker compose config --quiet
docker version
```

No ejecutar ciegamente el target `dev.run-docker`: retira un contenedor por nombre. Preparar un proyecto Compose de laboratorio identificado, puertos sin colisiones y volúmenes propios. Fijar digest de imágenes, hashes de cores/ROMs, codec y configuración efectiva. La sintaxis Compose correcta no sustituye un build.

## 3. Primer lote: E0, motor reproducible

Entregar una receta verificable y un registro de ejecución antes de modificar protocolos. Empezar por VP8/Opus según la configuración estudiada; no prometer H.264 ni migración transparente.

| Prueba | Preparación y acción | Resultado exigido |
|---|---|---|
| E0-01 · Build | Toolchain, imágenes y dependencias fijadas; construir Coordinator y worker | Binarios/imagen identificados, logs y versiones; sin cambios funcionales mezclados |
| E0-02 · Arranque | Core y ROM conocidos; abrir cliente upstream en laboratorio | Juego cargado, video y audio observados, input con efecto en emulación |
| E0-03 · Dos clientes | Dos sesiones de navegador independientes entran a la misma partida | Ambos reciben la misma ejecución; una sala/worker; identificar conexiones y puertos usados |
| E0-04 · Dos controles | Fixture con multijugador validado y mapeos distinguibles | Dos puestos independientes producen acciones observables; no confundir dos streams con dos controles |
| E0-05 · Save/load | Crear progreso identificable, guardar, alterarlo y cargar | Estado previo recuperado en ese juego/core; registrar archivos y comportamiento SRAM si aplica |
| E0-06 · Suite upstream | Con dependencias listas, ejecutar el target `make test` y pruebas pertinentes | Informe de éxitos, fallos y skips; un test omitido no cuenta como cobertura |

`TestStateConcurrency` y `TestLoad` estaban omitidos en la revisión estudiada. No quitar `t.Skip()` sin investigar sus requisitos. Una prueba manual de save local no demuestra persistencia cloud, aislamiento ni restauración en otro worker: eso pertenece a E5.

E0 se cierra con las evidencias anteriores y una repetición de la receta desde un entorno limpio equivalente. Si un fixture no soporta dos jugadores, registrar la limitación y seleccionar otro antes de declarar E0 completo. Un problema nativo de E0 no impide avanzar el spike E1a por separado.

## 4. Segundo lote: E1a, spike de identidad y contratos

Validar NestJS/Express + Better Auth + PostgreSQL/Prisma en `services/platform/`, sin adoptar versiones implícitas del sistema. Entregar runtime/dependencias fijados, lockfile, migraciones revisables y configuración de ejemplo sin secretos. Usar base y buzón de correo locales, cuentas sintéticas y pruebas contra PostgreSQL real para restricciones y transacciones.

La decisión de integración incluye ESM, orden del handler respecto del parser y compatibilidad de versiones; ver [investigación actualizada](identidad.md). Primero demostrar el framework y el ámbito personal; después ampliar el dominio. No construir criptografía ni recuperación de contraseña propias.

| Prueba | Acción | Resultado exigido |
|---|---|---|
| E1a-01 · Cuenta | Registro, verificación, login y logout con buzón local | Usuario persistido por el framework, acceso según verificación y sesión revocada al salir |
| E1a-02 · Recuperación | Restablecer contraseña y reutilizar token/sesión anterior | Token consumido no reutilizable; sesiones anteriores invalidadas según configuración explícita |
| E1a-03 · Invitado | Crear sesión anónima, recargar y llamar rutas de cuenta/tenant | Mantiene identidad temporal; no crea tenant, sala ni autoridad administrativa |
| E1a-04 · Aislamiento | Usuarios A/B, recursos de prueba con ámbito y cambio de identificador | Acceso cruzado denegado desde API; alta concurrente crea un solo tenant personal |
| E1a-05 · Frontera Admin | Cuenta ordinaria/invitado llaman rutas propias y nativas del plugin | Ningún camino concede funciones del equipo; roles amplios del plugin no quedan expuestos |
| E1a-06 · Factibilidad 2FA | Cuenta de prueba inicia sesión con desafío pendiente/completado | Se distingue factor configurado de factor satisfecho; el acceso administrativo exige ambos |
| E1a-07 · Cookies y origen | Peticiones con sesión ausente/revocada y origen no permitido | Rechazo observable; flujo legítimo funciona detrás del proxy/configuración elegidos |
| E1a-08 · Contratos | Validar fixtures iniciales Go/TypeScript de IDs, errores y capacidades | Mismo contrato interpretado por ambos consumidores; no reutilizar códigos ocupados del motor |

Los fixtures prueban contratos aislados, no salas ni juego real. La aceptación/canje de invitaciones se demuestra en E2; controles y espectador en E4; expulsión/revocación contra conexión viva y recuperación en E5. E1a demuestra factibilidad de Admin/2FA, mientras E1b/E1c entregan autoridad completa y consola.

El spike termina con una decisión de adopción o ajuste que explique evidencia, versiones y límites. Si falla el adaptador comunitario, probar el handler oficial Express dentro de NestJS antes de reconsiderar el stack completo. Los criterios de aislamiento y autoridad no se relajan para aceptar una biblioteca.

## 5. Continuación inmediata: E1b/E1c

Antes de abrir acceso externo, comprobar con API real y después con consola:

- Bootstrap idempotente del primer superadministrador sin credenciales predeterminadas; alta y desafío de segundo factor comprobados.
- Superadmin invita, limita y desactiva administradores. Admin no se promueve ni modifica personal mediante ninguna ruta.
- Revocar un perfil afecta la pestaña abierta; acciones pendientes reevalúan autoridad vigente. Proteger al último superadmin incluso con cambios concurrentes.
- Auditar consultas y mutaciones con actor/objetivo separados. Si falla el registro durable requerido, no reportar una mutación como completada.
- Atender un caso de soporte con datos sintéticos, listas paginadas, capacidades efectivas y errores reales. Las pestañas de recursos todavía no implementados no muestran datos ficticios como reales.
- Comprobar la consola y autenticación con el único sistema visual aprobado. En E1a se prepara su distribución verificable; E1c no crea una marca administrativa diferente.

Superar E1 no abre automáticamente el piloto: la aceptación externa requiere también E2–E7 y sus controles de operación.

## 6. Evidencia y organización de entregas

Un PR por resultado acotado: entorno E0; base/spike E1a; autoridad E1b; consola E1c. Los directorios del plan se crean cuando tengan implementación real, sin esqueletos vacíos para aparentar avance. La ubicación final de la app de jugadores y el mecanismo de distribución gráfica se registran en E1a; `apps/admin/` es la ubicación propuesta de la consola.

Guardar informes en `docs/retroreck/evidencias/` al ejecutar, con fecha, commit probado, comandos, entorno, fixture/core/hash, esperado, observado y estado **PASS / FAIL / BLOQUEADO / NO EJECUTADO**. Enlazar logs/capturas depurados cuando aporten evidencia; no versionar ROMs adicionales, secretos, cookies ni volcados de usuarios. Si se cambian configuración, core o binario, identificar la nueva revisión y repetir las pruebas afectadas.

Al cerrar cada sesión: actualizar evidencia y estado de etapa, listar bloqueos reales y dejar una siguiente acción concreta. Mantener separados cambios documentales y parches del motor. El [roadmap](roadmap.md) cubre después salas, ROM privada, controles, saves, pool y piloto; este documento no crea un roadmap paralelo.

## 7. Decisiones que se pueden resolver durante el trabajo

| Cuándo | Decisión y evidencia requerida |
|---|---|
| Inicio E0 | Toolchain/build, daemon, fixture multijugador y cores reproducibles |
| Cierre E1a | Adopción del framework, modelo de tenant, métodos de acceso/2FA, contratos iniciales, ubicación frontend y distribución visual |
| E3–E6 | Storage privado, red/ICE/TURN, límites, recursos y costo medidos por perfil |
| Antes de E7 externo | Operador inicial real, correo, infraestructura, presupuesto, recuperación y revisión aplicable de privacidad/contenido |

No se necesitan proveedor de nube ni credenciales de producción para el primer laboratorio. Si falta material imprescindible para una prueba, registrar el impedimento y avanzar las pruebas independientes; no sustituirlo por un resultado inventado.
