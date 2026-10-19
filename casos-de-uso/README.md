# Pandora — Casos de Uso

Este documento cubre actores, permisos y el catálogo completo de casos de uso, con sus flujos principales, alternativos y observaciones.

---

## Índice

- [Diagrama](#diagrama)
- [Actores y permisos](actores.md)
  - [Invitado](actores.md#invitado)
  - [Usuario autenticado](actores.md#usuario-autenticado)
  - [Moderador](actores.md#moderador)
  - [Admin](actores.md#admin)
- [Catálogo de casos de uso](#catálogo-de-casos-de-uso) — cada fila enlaza al caso de uso
- [Flujo consolidado de moderación (CU21–CU23, CU26–CU28)](flujo-moderacion.md)
- [Documento completo](completo.md) — todos los casos de uso en una sola página

---
# Diagrama


### Actores

Admin hereda todo lo que puede hacer Moderador, y Moderador todo lo de Usuario autenticado; por eso los diagramas siguientes sólo enlazan cada caso de uso con el actor de menor nivel que lo ejecuta.

```mermaid
---
config:
  layout: dagre
---
usecase-beta
  direction LR
  accTitle: Actores de Pandora
  accDescr: Jerarquía de actores. Admin hereda de Moderador, que hereda de Usuario autenticado.

  actor invitado("Invitado")
  actor usuario("Usuario autenticado")
  actor moderador("Moderador")
  actor admin("Admin")
  actor sistema("CRON / sistema")@{ type: hollow }

  moderador --|> usuario
  admin --|> moderador
```

### Cuenta

```mermaid
---
config:
  layout: dagre
---
usecase-beta
  direction LR
  accTitle: Casos de uso de cuenta
  accDescr: Registro, verificación, login, recuperación y gestión del perfil propio.

  actor invitado("Invitado")
  actor usuario("Usuario autenticado")
  actor sistema("CRON / sistema")@{ type: hollow }

  systemBoundary cuenta("Cuenta")@{ type: package }
    cu01("CU01 · Registrarse")
    cu02("CU02 · Verificar email")
    cu03("CU03 · Login")
    cu04("CU04 · Recuperar contraseña")
    cu05("CU05 · Limpiar cuentas no verificadas")
    cu06("CU06 · Ver/editar perfil")
    cu07("CU07 · Cambiar email")
    cu08("CU08 · Cambiar contraseña")
    cu31("CU31 · Subir avatar")
  end

  invitado -- cu01
  invitado -- cu02
  invitado -- cu03
  invitado -- cu04
  usuario -- cu06
  usuario -- cu07
  usuario -- cu08
  usuario -- cu31
  sistema -- cu05

  cu01 ..> : include cu02
  cu07 ..> : include cu02
```

### Obras

```mermaid
---
config:
  layout: dagre
---
usecase-beta
  direction LR
  accTitle: Casos de uso de obras
  accDescr: Publicación, edición, exploración, comentarios y calificación de obras.

  actor usuario("Usuario autenticado")
  actor invitado("Invitado")

  systemBoundary obras("Obras")@{ type: package }
    cu09("CU09 · Subir obra")
    cu10("CU10 · Editar obra")
    cu11("CU11 · Eliminar obra")
    cu13("CU13 · Comentar obra")
    cu14("CU14 · Calificar obra (Q2Q)")
    cu12("CU12 · Explorar obras")
  end

  invitado -- cu12
  usuario -- cu09
  usuario -- cu10
  usuario -- cu11
  usuario -- cu12
  usuario -- cu13
  usuario -- cu14
```

### Cartas y juego

```mermaid
---
config:
  layout: dagre
---
usecase-beta
  direction LR
  accTitle: Casos de uso de cartas y juego
  accDescr: Conversión de obras a cartas, colección, paquetes y autobattler.

  actor invitado("Invitado")
  actor usuario("Usuario autenticado")
  actor sistema("CRON / sistema")@{ type: hollow }

  systemBoundary cartas("Cartas y juego")@{ type: package }
    cu15("CU15 · Convertir obra a carta")
    cu16("CU16 · Explorar colección/cartas")
    cu17("CU17 · Abrir paquete")
    cu18("CU18 · Jugar autobattler")
  end

  sistema -- cu15
  invitado -- cu16
  usuario -- cu16
  usuario -- cu17
  usuario -- cu18
```

### Social

```mermaid
---
config:
  layout: dagre
---
usecase-beta
  direction LR
  accTitle: Casos de uso sociales
  accDescr: Seguimiento de usuarios, destacados y notificaciones.

  actor usuario("Usuario autenticado")

  systemBoundary social("Social")@{ type: package }
    cu19("CU19 · Seguir usuario")
    cu20("CU20 · Destacar obra o carta")
    cu24("CU24 · Notificaciones")
  end

  usuario -- cu19
  usuario -- cu20
  usuario -- cu24
```

### Moderación

```mermaid
---
config:
  layout: dagre
---
usecase-beta
  direction LR
  accTitle: Casos de uso de moderación
  accDescr: Denuncias, su resolución por moderación, y apelaciones.

  actor usuario("Usuario autenticado")
  actor moderador("Moderador")

  systemBoundary moderacion("Moderación")@{ type: package }
    cu21("CU21 · Denunciar obra")
    cu22("CU22 · Denunciar comentario")
    cu23("CU23 · Administrar denuncias")
    cu27("CU27 · Apelar una sanción")
    cu28("CU28 · Revisar una apelación")
  end

  usuario -- cu21
  usuario -- cu22
  usuario -- cu27
  moderador -- cu23
  moderador -- cu28
```

### Administración

CU15 pertenece a [Cartas y juego](#cartas-y-juego); aparece acá porque CU30 dispara esa misma rutina.

```mermaid
---
config:
  layout: dagre
---
usecase-beta
  direction LR
  accTitle: Casos de uso de administración
  accDescr: Herramientas exclusivas del admin. CU30 dispara la misma rutina de conversión que CU15.

  actor admin("Admin")

  systemBoundary administracion("Administración")@{ type: package }
    cu25("CU25 · Gestionar rol de moderador")
    cu26("CU26 · Reabrir reporte resuelto")
    cu29("CU29 · Cargar carta directamente")
    cu30("CU30 · Disparar conversión manual")
  end

  cu15("CU15 · Convertir obra a carta")

  admin -- cu25
  admin -- cu26
  admin -- cu29
  admin -- cu30

  cu30 ..> : include cu15
```

[Diagrama (Legacy) en draw.io](https://drive.google.com/file/d/1IKb8TWgXZewPg7JQTgwWP-iWlpQOBo1r/view?usp=sharing)

---

## Catálogo de casos de uso

Los casos de uso cubren registro, verificación y recuperación de cuenta, perfil (incluido el avatar), obras (subir/editar/eliminar/comentar/calificar), conversión a carta, colección, moderación, seguimiento, batalla, paquetes, destacados, reportes, notificaciones y las herramientas de administración/apelaciones.

| ID | Nombre | Actor principal |
|---|---|---|
| [CU01](cu01-registrarse.md) | Registrarse | Invitado |
| [CU02](cu02-verificar-email.md) | Verificar email | Invitado / cuenta pendiente de verificación |
| [CU03](cu03-login.md) | Login | Invitado |
| [CU04](cu04-recuperar-contrasena-olvidada.md) | Recuperar contraseña olvidada | Invitado |
| [CU05](cu05-limpieza-de-cuentas-no-verificadas.md) | Limpieza de cuentas no verificadas | CRON / sistema |
| [CU06](cu06-ver-editar-perfil.md) | Ver/editar perfil | Usuario autenticado |
| [CU07](cu07-cambiar-email.md) | Cambiar email | Usuario autenticado |
| [CU08](cu08-cambiar-contrasena.md) | Cambiar contraseña | Usuario autenticado |
| [CU09](cu09-subir-obra.md) | Subir obra | Usuario autenticado |
| [CU10](cu10-editar-obra.md) | Editar obra | Usuario autenticado |
| [CU11](cu11-eliminar-obra.md) | Eliminar obra | Usuario autenticado |
| [CU12](cu12-explorar-obras.md) | Explorar obras | Invitado o usuario |
| [CU13](cu13-comentar-obra.md) | Comentar obra | Usuario autenticado |
| [CU14](cu14-calificar-obra-mediante-q2q.md) | Calificar obra mediante Q2Q | Usuario autenticado |
| [CU15](cu15-conversion-de-obra-a-carta.md) | Conversión de obra a carta | CRON / sistema |
| [CU16](cu16-explorar-coleccion-cartas.md) | Explorar colección/cartas | Invitado o usuario autenticado |
| [CU17](cu17-abrir-paquete.md) | Abrir paquete | Usuario autenticado |
| [CU18](cu18-jugar-autobattler.md) | Jugar autobattler | Usuario autenticado |
| [CU19](cu19-seguir-usuario.md) | Seguir usuario | Usuario autenticado |
| [CU20](cu20-destacar-obra-o-carta.md) | Destacar obra o carta | Usuario autenticado |
| [CU21](cu21-denunciar-obra.md) | Denunciar obra | Usuario autenticado |
| [CU22](cu22-denunciar-comentario.md) | Denunciar comentario | Usuario autenticado |
| [CU23](cu23-administrar-denuncias.md) | Administrar denuncias | Moderador |
| [CU24](cu24-notificaciones.md) | Notificaciones | Usuario autenticado |
| [CU25](cu25-gestionar-rol-de-moderador.md) | Gestionar rol de moderador | Admin |
| [CU26](cu26-reabrir-un-reporte-resuelto.md) | Reabrir un reporte resuelto | Admin |
| [CU27](cu27-apelar-una-sancion.md) | Apelar una sanción | Usuario autenticado afectado |
| [CU28](cu28-revisar-una-apelacion.md) | Revisar una apelación | Moderador o Admin |
| [CU29](cu29-cargar-carta-directamente.md) | Cargar carta directamente (Admin) | Admin |
| [CU30](cu30-disparar-el-ciclo-de-conversion-manualmente.md) | Disparar el ciclo de conversión manualmente (Admin) | Admin |
| [CU31](cu31-subir-avatar.md) | Subir avatar | Usuario autenticado, con email verificado |
