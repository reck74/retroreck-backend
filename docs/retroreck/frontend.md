# Frontend y sistema visual aprobado

19 de septiembre de 2026. Requisito transversal del plan; no implementación de interfaces.

## Autoridad única

El sistema gráfico de RetroReck es la única regla visual aprobada para landing, identidad, biblioteca, invitaciones, lobby, partida y consolas administrativas. La fuente canónica vive en el workspace de producto, bajo `docs/design-system/`: guía, referencia HTML, tokens CSS/JSON e inventario con hashes de los seis PNG y logo. El documento de frontend del producto relaciona esos diseños con los contratos funcionales.

El fork no mantiene una segunda guía ni paleta. En el checkout conjunto, consultar `../docs/design-system/README.md` desde la raíz del fork. Ese directorio no se distribuye todavía con un clon aislado de este repositorio: antes de su primer cambio UI debe recibir el paquete canónico, con versión y hashes comprobados. Integrar esa distribución en E1a; las copias para build serán generadas y verificadas, nunca editadas como fuentes independientes. Hasta disponer de la referencia se puede continuar el trabajo de API/motor, sin inventar una interfaz sustitutiva.

Las etiquetas O/D/P describen procedencia de valores observados, documentados o propuestos al definir el sistema. El sistema completo es normativo; esas etiquetas no autorizan estilos alternativos ni afirman que los estados móviles estén dibujados en los originales.

## Consola de comunidad

La consola de [administración](administracion.md) reutiliza el logo, tokens, navegación, formularios, paneles, tablas y diálogos del mismo sistema. No existe todavía un mockup específico de consola; su composición es una extensión revisable del sistema, no un nuevo tema aprobado. Adaptar cualquier librería de administración a estas reglas.

El administrador consulta usuarios, capacidades e inventario autorizado para soporte. El superadministrador dispone además de cuentas del equipo y perfiles de permisos. La API aplica las restricciones y devuelve permisos efectivos; los estilos o el estado visible de los botones no otorgan autoridad.

## Entregas y aceptación

- **E1a:** distribución versionada de tokens/assets, componentes base y flujos de identidad.
- **E1b/E1c:** gestión del equipo/permisos y consola de comunidad con estados auditados.
- **E2–E5:** invitaciones, biblioteca, controles, espectadores, partida y progreso según los seis diseños.
- **E6/E7:** vistas operativas y validación completa del piloto.

Cada entrega UI registra el mockup o patrón reutilizado, tokens/componentes, estados y tamaños revisados. Comprobar escritorio frente al original y anchos 1280, 768, 390 y 320, teclado, foco, contraste, zoom, movimiento reducido, textos largos y errores reales. No mostrar datos de demostración como capacidades ya implementadas.

El aspecto del frontend upstream no se adopta como tema. Sus assets funcionales se retiran solo al reemplazar sus dependencias; conservar la posibilidad de validar el baseline E0. Los contratos del motor se verifican por código/pruebas, nunca copiando textos o códigos de invitación de las imágenes.
