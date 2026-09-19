# RetroReck: diseño e implementación del backend

Fecha: 2026-09-19. Estado: propuesta técnica; funcionalidades de RetroReck todavía no implementadas.

Este repositorio es el fork de [giongto35/cloud-game](https://github.com/giongto35/cloud-game) para el backend de ejecución de RetroReck. Base estudiada: `1fab9ef07a02cab0e0cdea169e95b5ec6e2e04ed`.

El objetivo es conservar el motor Libretro/WebRTC y añadir cuentas, administración global de la comunidad, biblioteca privada, salas con invitados, control de puestos, espectadores y operación de un pool de workers.

| Documento | Contenido |
|---|---|
| [Plan de implementación](./plan.md) | Requisitos acordados, arquitectura, módulos, contratos, etapas y criterios de aceptación |
| [Identidad y tenants](./identidad.md) | Comparativa de frameworks, recomendación, permisos y modelo de datos |
| [Administración de la comunidad](./administracion.md) | Administradores, superadministradores, soporte, gestión de permisos y criterios de aceptación |
| [Pool de workers](./workers.md) | Asignación, escalado, recuperación, costos medibles y pruebas |
| [Prioridades y roadmap](./roadmap.md) | Orden de trabajo, backlog inicial y entregables necesarios para abrir el piloto |

Los requisitos de participación vienen de la definición del producto. Frameworks, límites iniciales y arquitectura de despliegue son recomendaciones de diseño, sujetas a los experimentos indicados; no son capacidades ya disponibles en este fork.

## Relación con upstream

- `origin`: fork de RetroReck; `upstream`: proyecto original.
- Mantener `master` como referencia inicial mientras se revisa esta propuesta.
- Introducir cambios mediante ramas y revisiones pequeñas; conservar licencia y atribuciones originales.
- Mantener inicialmente el módulo Go y sus rutas originales para reducir conflictos de actualización.
- Añadir el servicio de producto en un directorio separado, sin reorganizar de entrada el código del motor.
- Evaluar los cambios entrantes de upstream con pruebas de protocolo, emulación y guardados. No actualizar automáticamente cores o imágenes sin validación.

Esta entrega agrega documentación. No instala el framework de identidad, no levanta infraestructura y no modifica el comportamiento del motor.
