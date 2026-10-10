# Pandora — Clases de Dominio y Diagrama

Este documento presenta el **modelo conceptual de dominio**: las clases del negocio, su responsabilidad y sus relaciones, independientemente de cómo se implementan o persisten.

---

## Índice

- [Diagrama](#diagrama)
- [Catálogo de clases de dominio](#catálogo-de-clases-de-dominio) — cada fila enlaza a la clase
- [Relaciones entre clases de dominio](#relaciones-entre-clases-de-dominio)
- [Documento completo](completo.md) — todas las clases en una sola página

---

## Diagrama

[Diagrama en Mermaid Live](https://mermaid.live/edit#pako:eNqVVs1u2zgQfhWBpxZrG5Ycu7YOCxROgfbQtEjSHha-TMiJQ5QiVYp03QZ5qj5CX6wjWaKlWLazvnhIzffN_4CPjBuBLGVcQVFcSlhbyFZ6paOouom-FB6sNNFjeRVF__gCrYYM6yNmIFUt56T_w1jxHoqH5nPhQJilR-2gvrqTppZgAw7sF6vaXF_RynvJCRWl0Z0xCqHy5qnt1LVRwSFtsjuLByqf7iwEHSedV41ZgQW3MufS6PpGZrBG_cnKtdTQeFMYJbl0sDR6g7Yg7a5D--iCQkOnCWiWoKpAWnZQyYwsCDgR2i2sz4W2NFmZz3ZVHG6deWblVAK_gjK2ci1QUDAWFX0lWDwaTUNRTBNB1wmwbp9gKuR3jyHB96iLpt4bKYKIisgEiPpsweIvOKD-QPncxXeJn0teh0WwxEG7kuItdx4U-ToejeJx05zKUS2vkaqJFnr9vsbcWIeBLzNObprU8ZDZToUbd5F6wrfqSRcelTPvDs28zVF189sxJNCBUthvZSML6pEe0iuiaFqq1du56WtzbrRDLQOvQnmy8ZYWBWouQd1iRjkC1TFxa75h6ONtLm27sf0Rh3eD_27ryp0R6HJrNojC2GZexOfmZqdqDohusOj06nMHKGmG9w9WKTRLLFqxeMWi4fBfkq5IqrZE-KUReLdz6yiiNXwvRLRmrUJgJl1V-WD8uI2Tanvio2rlcJSE1bDWMfJ6X_35_Tw3V10v9qAKt6S8cl7Colfc5BKK10fjLqW-MT5pMHyoDd7g2tMqIxJzEve8iJflPPF6ol4cXgfVSmYXRev5ZLHbI9oh6itgvYtKvU5bndDr5OiIXhUOzbIvp1n_T9xuyVFPR6_K5nl9Fny4N85D2mvhrPZu9nvUupWhN8FZqv1WpsEt5Zdkpw2qlnMnN_vc9Y9eQK80G7C1lYKlznocsAwtPXjoyKq1tmLuAeldxVISKad-OxRgvw25oSlfsZV-InwO-j9jsobCGr9-YOk9qIJOPhfgsH7DBRXUAu3SeO1YmswqCpY-si1LJ_FodjFOLuJZkiySi_l4OmA_WTqcJ4tR_GaWzBbTSZxMnwbsV2UzHs0n03gxn84vkvFiNpkMGApJrfKxfkKWf09_AegKK50)

``` mermaid
classDiagram

  class Usuario {
    +username
    +email
    +passwordHash
    +estadoCuenta
    +bio
    +avatarUrl
    +emailVerificado : boolean
  }

  class Rol {
    +nombre
  }

  class Obra {
    +titulo
    +descripcion
    +imagenOriginal
    +solicitaConversion : boolean
    +estadoConversion
    +inicioCalificacion
    +eliminada : boolean
  }

  class Tag {
    +nombre
  }

  class Comentario {
    +texto
    +eliminado : boolean
  }

  class Valoracion {
    +estrellas : 1..5
    +emocion
  }

  class Carta {
    +ataque
    +defensa
    +vida
    +velocidad
    +rareza
  }

  class InventarioDePaquetes {
    +cantidadActual : 0..10
    +ultimaRegeneracion
  }

  class Reporte {
    +motivo
    +comentario
    +estado
    +resolucion
    +resueltoEn
  }

  class Apelacion {
    +motivo
    +detalle
    +estado
    +revisadaEn
  }

  class Notificacion {
    +tipo
    +titulo
    +contenido
    +leida : boolean
  }

  class CredencialTemporal {
    +tipoToken
    +expiracion
    +usadaEn
  }

  class CuentaExterna {
    +proveedor
    +idProveedorExterno
  }

  class Sesion {
    +expiracion
    +revocada : boolean
  }


  Usuario  "1" --> "N" Obra          : autor
  Usuario  "1" --> "N" Comentario    : autor
  Usuario  "1" --> "N" Valoracion    : emite
  Obra     "1" --> "N" Comentario
  Obra     "1" --> "N" Valoracion
  Obra     "1" --> "0..1" Carta      : conversión
  Usuario  "N" --> "N" Carta         : Colección (copias)
  Usuario  "1" --> "1" InventarioDePaquetes
  Usuario  "N" --> "N" Usuario       : Seguimiento
  Usuario  "N" --> "N" Obra          : Destacado
  Usuario  "N" --> "N" Carta         : Destacado
  Obra     "N" --> "N" Tag
  Usuario  "1" --> "N" Notificacion
  Obra         "1" --> "N" Reporte
  Comentario   "1" --> "N" Reporte
  Usuario      "1" --> "N" Reporte   : denunciante
  Usuario      "1" --> "N" Reporte   : resolutor (0..1)
  Usuario      "1" --> "N" CredencialTemporal
  Usuario      "1" --> "N" CuentaExterna
  Usuario      "1" --> "N" Sesion
  Usuario      "N" --> "N" Rol
  Usuario      "1" --> "N" Apelacion : apelante
  Usuario      "1" --> "N" Apelacion : revisor (0..1)
  Reporte      "1" --> "0..1" Apelacion

```

---

## Catálogo de clases de dominio

| Clase | Descripción |
|---|---|
| [Usuario](usuario.md) | Representa a cualquier persona registrada en la plataforma. |
| [Rol](rol.md) | Etiqueta de autorización que un Usuario puede poseer, independiente de su identidad. |
| [Obra (Artwork)](obra.md) | La unidad central de contenido de la plataforma. |
| [Tag](tag.md) | Etiqueta libre que clasifica una Obra para búsqueda y filtrado. |
| [Comentario](comentario.md) | Texto libre que un Usuario deja sobre una Obra. |
| [Valoración (Rating / Q2Q)](valoracion.md) | Un voto que un Usuario emite sobre una Obra durante su período de calificación. |
| [Carta (Card)](carta.md) | El resultado de convertir una Obra al finalizar su período de calificación (o de crearla directamente como admin). |
| [Colección (relación Usuario–Carta)](coleccion.md) | No es una entidad autónoma sino la relación de posesión entre un Usuario y una Carta, con un contador de copias. |
| [Inventario de Paquetes](inventario-de-paquetes.md) | Representa la cantidad de paquetes de cartas disponibles para un Usuario y el estado de su regeneración periódica. |
| [Seguimiento (Follow)](seguimiento.md) | Relación asimétrica entre dos Usuarios. |
| [Destacado (Favorite)](destacado.md) | Relación entre un Usuario y una Obra propia, o entre un Usuario y una Carta que posee, usada para la sección de "destacados" de su perfil. |
| [Reporte (Report)](reporte.md) | Una denuncia sobre una Obra o un Comentario, presentada por un Usuario y eventualmente resuelta por un moderador o admin. |
| [Apelación (Appeal)](apelacion.md) | La solicitud de un Usuario sancionado para que se revise y, potencialmente, revierta la sanción aplicada por un Reporte ya resuelto. |
| [Notificación (Notification)](notificacion.md) | Un registro persistente de un acontecimiento relevante para un Usuario. |
| [Credencial temporal (Token de verificación / recuperación)](credencial-temporal.md) | Un secreto de un solo uso, con expiración, emitido para completar una operación sensible (verificar el email o recuperar la contraseña). |
| [Cuenta externa (OAuthAccount)](cuenta-externa.md) | El vínculo entre un Usuario de Pandora y su identidad en un proveedor externo (actualmente sólo Google). |
| [Sesión (Refresh Token)](sesion.md) | Representa una sesión de larga duración que permite renovar el token de acceso sin volver a autenticarse con credenciales. |
| [Batalla (concepto no persistido)](batalla.md) | Simulación que ocurre enteramente en el cliente a partir de dos Cartas; **no es una clase de dominio persistente**. |

---

## Relaciones entre clases de dominio

```text
Usuario 1 ─── N Obra                       (autoría)
Usuario 1 ─── N Comentario                 (autoría)
Usuario 1 ─── N Valoración                 (autoría)
Obra     1 ─── N Comentario
Obra     1 ─── N Valoración
Obra     1 ─── 0..1 Carta                  (conversión)
Usuario  N ─── N Carta                     (Colección, con copias)
Usuario  1 ─── 1 Inventario de Paquetes
Usuario  N ─── N Usuario                   (Seguimiento)
Usuario  N ─── N Obra                      (Destacado de obra)
Usuario  N ─── N Carta                     (Destacado de carta)
Obra     N ─── N Tag
Usuario  1 ─── N Notificación
Obra         1 ─── N Reporte
Comentario   1 ─── N Reporte
Usuario      1 ─── N Reporte               (como denunciante)
Usuario      1 ─── N Reporte               (como resolutor, opcional)
Usuario      1 ─── N Credencial temporal   (verificación / recuperación)
Usuario      1 ─── N Cuenta externa
Usuario      1 ─── N Sesión
Usuario      N ─── N Rol
Usuario      1 ─── N Apelación             (como apelante)
Usuario      1 ─── N Apelación             (como revisor, opcional)
Reporte      1 ─── 0..1 Apelación
```
