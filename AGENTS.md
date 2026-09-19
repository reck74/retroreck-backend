# RetroReck Backend — instrucciones de implementación

Leer primero [el plan vigente](docs/retroreck/README.md). Este repositorio conserva el motor cloud-game; las funciones RetroReck descritas son requisitos y propuestas, no implementación terminada.

Toda interfaz de RetroReck, incluidas autenticación, soporte, administración y superadministración, usa exclusivamente el sistema visual aprobado. Leer [el contrato de frontend](docs/retroreck/frontend.md) antes de cualquier cambio UI. El frontend upstream es una referencia funcional, no una dirección gráfica aprobada.

La fuente visual está en `docs/design-system/` del workspace de producto RetroReck. En el checkout conjunto, ese directorio está en `../docs/design-system/` respecto de este repositorio. Abrir guía, referencia y PNG aplicable y reutilizar tokens. Si se trabaja con el fork aislado, obtener el paquete canónico del producto antes de implementar UI; no suplir su ausencia con una paleta, logo o tema nuevo.

El material `legacy/` del producto es histórico inerte. No extraer contratos, código ni recursos gráficos para la aplicación. Las bibliotecas elegidas deben adaptarse al sistema aprobado; un cambio de dirección artística requiere instrucción explícita del usuario.
