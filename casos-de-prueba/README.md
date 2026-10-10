# Pandora — Casos de Prueba

Este documento cubre la estrategia de pruebas y un caso de prueba por [caso de uso](/casos-de-uso/README.md), derivado de su flujo principal, su postcondición y sus flujos alternativos. La cobertura automatizada real (Jest) vive en el código fuente, bajo `src/**/*.spec.ts`, a la fecha de este documento, 50 suites con más de 330 pruebas unitarias.

> CP01–CP23 cubren los casos de uso originales (ahora renumerados, ver [casos de uso](/casos-de-uso/README.md)). Los casos de uso agregados después de esa primera versión ([CU02](/casos-de-uso/cu02-verificar-email.md), [CU05](/casos-de-uso/cu05-limpieza-de-cuentas-no-verificadas.md), [CU07](/casos-de-uso/cu07-cambiar-email.md), [CU08](/casos-de-uso/cu08-cambiar-contrasena.md), [CU10](/casos-de-uso/cu10-editar-obra.md), [CU11](/casos-de-uso/cu11-eliminar-obra.md), [CU13](/casos-de-uso/cu13-comentar-obra.md)) todavía no tienen un caso de prueba dedicado en este documento.

---

## Índice

- [Estrategia de pruebas](estrategia.md)
  - [Pruebas unitarias (backend)](estrategia.md#pruebas-unitarias-backend)
  - [Pruebas de integración](estrategia.md#pruebas-de-integración)
  - [Pruebas end-to-end (flujos completos prioritarios)](estrategia.md#pruebas-end-to-end-flujos-completos-prioritarios)
- [Catálogo de casos de prueba](#catálogo-de-casos-de-prueba) — cada fila enlaza al caso de prueba
- [Documento completo](completo.md) — todos los casos de prueba en una sola página

---

## Catálogo de casos de prueba

Cada caso de prueba usa como **pasos** el flujo principal del caso de uso correspondiente, como **resultado esperado** su postcondición, y como **casos negativos/alternativos** sus flujos `A*`.

| ID | Caso de prueba | Caso de uso |
|---|---|---|
| [CP01](cp01-registro-exitoso.md) | Registro exitoso | [CU01](/casos-de-uso/cu01-registrarse.md) |
| [CP02](cp02-login.md) | Login | [CU03](/casos-de-uso/cu03-login.md) |
| [CP03](cp03-subir-obra.md) | Subir obra | [CU09](/casos-de-uso/cu09-subir-obra.md) |
| [CP04](cp04-explorar-obras.md) | Explorar obras | [CU12](/casos-de-uso/cu12-explorar-obras.md) |
| [CP05](cp05-calificar-obra-mediante-q2q.md) | Calificar obra mediante Q2Q | [CU14](/casos-de-uso/cu14-calificar-obra-mediante-q2q.md) |
| [CP06](cp06-conversion-de-obra-a-carta.md) | Conversión de obra a carta | [CU15](/casos-de-uso/cu15-conversion-de-obra-a-carta.md) |
| [CP07](cp07-explorar-coleccion-cartas.md) | Explorar colección/cartas | [CU16](/casos-de-uso/cu16-explorar-coleccion-cartas.md) |
| [CP08](cp08-abrir-paquete.md) | Abrir paquete | [CU17](/casos-de-uso/cu17-abrir-paquete.md) |
| [CP09](cp09-jugar-autobattler.md) | Jugar autobattler | [CU18](/casos-de-uso/cu18-jugar-autobattler.md) |
| [CP10](cp10-seguir-usuario.md) | Seguir usuario | [CU19](/casos-de-uso/cu19-seguir-usuario.md) |
| [CP11](cp11-destacar-obra-o-carta.md) | Destacar obra o carta | [CU20](/casos-de-uso/cu20-destacar-obra-o-carta.md) |
| [CP12](cp12-denunciar-obra.md) | Denunciar obra | [CU21](/casos-de-uso/cu21-denunciar-obra.md) |
| [CP13](cp13-denunciar-comentario.md) | Denunciar comentario | [CU22](/casos-de-uso/cu22-denunciar-comentario.md) |
| [CP14](cp14-administrar-denuncias.md) | Administrar denuncias | [CU23](/casos-de-uso/cu23-administrar-denuncias.md) |
| [CP15](cp15-notificaciones.md) | Notificaciones | [CU24](/casos-de-uso/cu24-notificaciones.md) |
| [CP16](cp16-ver-editar-perfil.md) | Ver/editar perfil | [CU06](/casos-de-uso/cu06-ver-editar-perfil.md) |
| [CP17](cp17-gestionar-rol-de-moderador.md) | Gestionar rol de moderador | [CU25](/casos-de-uso/cu25-gestionar-rol-de-moderador.md) |
| [CP18](cp18-reabrir-un-reporte-resuelto.md) | Reabrir un reporte resuelto | [CU26](/casos-de-uso/cu26-reabrir-un-reporte-resuelto.md) |
| [CP19](cp19-apelar-una-sancion.md) | Apelar una sanción | [CU27](/casos-de-uso/cu27-apelar-una-sancion.md) |
| [CP20](cp20-revisar-una-apelacion.md) | Revisar una apelación | [CU28](/casos-de-uso/cu28-revisar-una-apelacion.md) |
| [CP21](cp21-cargar-carta-directamente.md) | Cargar carta directamente (Admin) | [CU29](/casos-de-uso/cu29-cargar-carta-directamente.md) |
| [CP22](cp22-disparar-el-ciclo-de-conversion-manualmente.md) | Disparar el ciclo de conversión manualmente (Admin) | [CU30](/casos-de-uso/cu30-disparar-el-ciclo-de-conversion-manualmente.md) |
| [CP23](cp23-recuperar-contrasena-olvidada.md) | Recuperar contraseña olvidada | [CU04](/casos-de-uso/cu04-recuperar-contrasena-olvidada.md) |
