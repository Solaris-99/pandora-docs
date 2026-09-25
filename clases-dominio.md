# Pandora — Clases de Dominio y Diagrama

> Documento derivado de `pandora-diseño-general.md` (que se mantiene como referencia consolidada). Presenta el **modelo conceptual de dominio**: las clases del negocio, su responsabilidad y sus relaciones, independientemente de cómo se implementan o persisten. La realización concreta (tipos de dato, entidades TypeORM, DTOs, servicios) vive en `pandora-clases-diseno.md`; el esquema físico de base de datos en `pandora-modelo-tablas.md`.
>
> No se incluye diagrama gráfico en esta versión — el catálogo de clases y la lista de relaciones al final del documento son su equivalente textual.

---

# 1. Catálogo de clases de dominio

## 1.1 Usuario

Representa a cualquier persona registrada en la plataforma. Un mismo Usuario es simultáneamente artista potencial y coleccionista potencial — no existen subtipos de cuenta.

- **Atributos conceptuales:** nombre de usuario, email, contraseña (sólo si se registró localmente), estado de la cuenta (activo/suspendido/baneado/pendiente de verificación), biografía, avatar, email verificado (sí/no).
- **Responsabilidades:** autenticarse; publicar Obras; comentar; calificar; seguir a otros Usuarios; destacar Obras/Cartas propias o poseídas; poseer una Colección de Cartas y un Inventario de Paquetes; recibir Notificaciones; (si tiene rol de moderación) resolver Reportes y revisar Apelaciones; (si es admin) gestionar Roles y disparar herramientas administrativas.
- **Relaciones:** autor de N Obras · autor de N Comentarios · emisor de N Valoraciones · posee N Cartas (a través de la Colección, con contador de copias) · posee 1 Inventario de Paquetes · sigue a N Usuarios / es seguido por N Usuarios · destaca N Obras y N Cartas · recibe N Notificaciones · tiene N Roles · presenta N Apelaciones (como apelante) · revisa N Apelaciones (como moderador/admin) · resuelve N Reportes (como moderador/admin).

## 1.2 Rol

Etiqueta de autorización que un Usuario puede poseer, independiente de su identidad. Un Usuario puede tener más de un Rol a la vez (p. ej. `user` + `moderator`).

- **Valores:** `user` (rol por defecto), `moderator`, `admin`.
- **Responsabilidades:** habilitar o restringir funcionalidad según el conjunto de roles del Usuario que la solicita.
- **Relaciones:** N Usuarios ↔ N Roles.
- **Nota de negocio:** el rol `admin` es de confianza y se asigna fuera de la aplicación; nunca se otorga ni se revoca a través de un flujo propio del sistema.

## 1.3 Obra (Artwork)

La unidad central de contenido de la plataforma: una pieza de arte digital publicada por un Usuario, que opcionalmente evoluciona a una Carta.

- **Atributos conceptuales:** título, descripción/lore (opcional), imagen (original + versiones optimizadas), si solicita conversión a carta, estado de conversión, fecha de inicio del período de calificación, si fue eliminada.
- **Responsabilidades:** exponerse en el explorador público mientras no esté eliminada; permanecer calificable durante exactamente una semana desde su publicación; permanecer comentable indefinidamente; convertirse en, como máximo, una Carta.
- **Relaciones:** pertenece a 1 Usuario (autor) · tiene N Comentarios · tiene N Valoraciones · está asociada a N Tags · genera 0..1 Carta · es destacada por N Usuarios · es denunciada mediante N Reportes.
- **Regla de estado clave:** una Obra eliminada nunca aparece como contenido público activo, pero conserva su historial (eliminación lógica).

## 1.4 Tag

Etiqueta libre que clasifica una Obra para búsqueda y filtrado.

- **Atributos conceptuales:** nombre (normalizado, p. ej. minúsculas).
- **Relaciones:** N Obras ↔ N Tags.

## 1.5 Comentario

Texto libre que un Usuario deja sobre una Obra. No expira junto con el período de calificación.

- **Atributos conceptuales:** texto, autor, obra referenciada, si fue eliminado.
- **Relaciones:** pertenece a 1 Obra · escrito por 1 Usuario · es denunciado mediante N Reportes · puede tener 0..1 Apelación (si fue eliminado por moderación).

## 1.6 Valoración (Rating / Q2Q)

Un voto que un Usuario emite sobre una Obra durante su período de calificación: una cantidad de estrellas (1 a 5) y una emoción.

- **Atributos conceptuales:** estrellas, emoción, obra, votante.
- **Responsabilidades:** contribuir, junto con el resto de las Valoraciones de una Obra, al cálculo de las estadísticas y la rareza de la futura Carta.
- **Relaciones:** pertenece a 1 Obra · emitida por 1 Usuario.
- **Regla de unicidad:** un Usuario tiene como máximo una Valoración activa por Obra; volver a calificar actualiza la existente en vez de crear una nueva.

## 1.7 Carta (Card)

El resultado de convertir una Obra al finalizar su período de calificación (o de crearla directamente como admin). Representa la pieza coleccionable y jugable.

- **Atributos conceptuales:** ataque, defensa, vida, velocidad, rareza, obra de origen.
- **Responsabilidades:** ser coleccionada (con copias) por los Usuarios que la obtienen; participar como carta propia o como rival en una Batalla; ocultar sus estadísticas ante quien no la posee.
- **Relaciones:** se genera a partir de exactamente 1 Obra (relación 1 a 1, máximo una Carta por Obra) · es poseída por N Usuarios, cada uno con su propio contador de copias · puede ser destacada por N Usuarios.

## 1.8 Colección (relación Usuario–Carta)

No es una entidad autónoma sino la relación de posesión entre un Usuario y una Carta, con un contador de copias.

- **Atributos conceptuales:** usuario, carta, cantidad de copias poseídas.
- **Responsabilidades:** determinar si las estadísticas completas de una Carta son visibles para ese Usuario (visible sólo si copias > 0); incrementarse al recibir una copia repetida desde un Paquete.

## 1.9 Inventario de Paquetes

Representa la cantidad de paquetes de cartas disponibles para un Usuario y el estado de su regeneración periódica.

- **Atributos conceptuales:** usuario propietario, cantidad actual, momento de la última regeneración.
- **Responsabilidades:** regenerar un paquete por minuto hasta un máximo de 10, sin acumular deuda una vez alcanzado el máximo; permitir el consumo atómico de un paquete, que entrega 5 Cartas nuevas al Usuario.
- **Relaciones:** pertenece a exactamente 1 Usuario (1 a 1).

## 1.10 Seguimiento (Follow)

Relación asimétrica entre dos Usuarios.

- **Atributos conceptuales:** usuario que sigue, usuario seguido.
- **Responsabilidades:** determinar quién recibe una Notificación cuando el usuario seguido publica una Obra nueva.
- **Relaciones:** N Usuarios ↔ N Usuarios. Un Usuario no puede seguirse a sí mismo.

## 1.11 Destacado (Favorite)

Relación entre un Usuario y una Obra propia, o entre un Usuario y una Carta que posee, usada para la sección de "destacados" de su perfil.

- **Atributos conceptuales:** usuario, obra o carta destacada.
- **Regla de negocio:** sólo puede destacarse una Obra propia o una Carta efectivamente poseída.
- **Relaciones:** N Usuarios ↔ N Obras (favoritas) · N Usuarios ↔ N Cartas (favoritas).

## 1.12 Reporte (Report)

Una denuncia sobre una Obra o un Comentario, presentada por un Usuario y eventualmente resuelta por un moderador o admin. Conceptualmente una única clase, aunque a nivel de datos se distingue por el tipo de objetivo (ver `pandora-modelo-tablas.md`).

- **Atributos conceptuales:** objetivo denunciado (Obra o Comentario), denunciante, motivo, comentario opcional, estado (`pending`/`in_review`/`resolved`/`rejected`/`overturned`), resolución aplicada (si fue resuelto), quién lo resolvió, cuándo.
- **Responsabilidades:** disparar el flujo de moderación; registrar auditoría de quién tomó qué acción y cuándo; habilitar, si su resolución fue punitiva, que el afectado presente una Apelación o que un admin lo reabra directamente.
- **Relaciones:** referencia 1 Obra o 1 Comentario · presentado por 1 Usuario (denunciante) · resuelto por 0..1 Usuario (moderador/admin) · referenciado por 0..1 Apelación.

## 1.13 Apelación (Appeal)

La solicitud de un Usuario sancionado para que se revise y, potencialmente, revierta la sanción aplicada por un Reporte ya resuelto.

- **Atributos conceptuales:** reporte referenciado, apelante, motivo predefinido, texto explicativo opcional, estado (`pending`/`approved`/`rejected`), quién la revisó, cuándo.
- **Responsabilidades:** representar el único canal (junto con la reapertura directa de un admin) para revertir una sanción; garantizar que quien revisa no sea quien resolvió el reporte original, salvo que sea la única persona de moderación disponible.
- **Relaciones:** referencia exactamente 1 Reporte (y sólo puede existir una Apelación por Reporte) · presentada por 1 Usuario · revisada por 0..1 Usuario (moderador/admin).

## 1.14 Notificación (Notification)

Un registro persistente de un acontecimiento relevante para un Usuario.

- **Atributos conceptuales:** destinatario, tipo de evento, título, contenido, si fue leída.
- **Eventos que la generan:** usuario seguido; nuevo upload de un usuario seguido; fin de período de calificación; obra convertida; comentario nuevo sobre una obra; comentario/obra eliminados por moderación (y su reversión); resumen de actividad de votos; advertencia o baneo; resultado de una apelación. El catálogo no es cerrado y puede ampliarse.
- **Relaciones:** pertenece a exactamente 1 Usuario (destinatario).

## 1.15 Credencial temporal (Token de verificación / recuperación)

Dos variantes de un mismo concepto: un secreto de un solo uso, con expiración, emitido para completar una operación sensible sin exponer la contraseña del Usuario.

- **Token de verificación de email:** confirma que el Usuario controla la dirección de correo con la que se registró.
- **Token de recuperación de contraseña:** habilita, por única vez, establecer una contraseña nueva sin conocer la anterior.
- **Responsabilidades comunes:** expirar automáticamente; invalidarse tras un único uso; invalidar cualquier token anterior sin usar del mismo Usuario al emitir uno nuevo.
- **Relaciones:** cada instancia pertenece a exactamente 1 Usuario.

## 1.16 Cuenta externa (OAuthAccount)

El vínculo entre un Usuario de Pandora y su identidad en un proveedor externo (actualmente sólo Google).

- **Atributos conceptuales:** usuario, proveedor, identificador del usuario en ese proveedor.
- **Relaciones:** pertenece a 1 Usuario. Un mismo Usuario puede tener, a futuro, más de una cuenta externa vinculada (uno por proveedor).

## 1.17 Sesión (Refresh Token)

Representa una sesión de larga duración que permite renovar el token de acceso sin volver a autenticarse con credenciales.

- **Atributos conceptuales:** usuario, secreto opaco, expiración, si fue revocada.
- **Responsabilidades:** rotarse en cada uso (el token usado queda revocado y se emite uno nuevo); revocarse íntegramente al cerrar sesión o al restablecer la contraseña del Usuario.
- **Relaciones:** pertenece a 1 Usuario. Un Usuario puede tener múltiples sesiones activas simultáneamente (p. ej. web + Android).

## 1.18 Batalla (concepto no persistido)

La Batalla/autobattler **no es una clase de dominio persistente**: es una simulación que ocurre enteramente en el cliente a partir de dos Cartas (la propia y una rival elegida al azar por el backend). No se modela como entidad porque no existe registro de partidas, turnos ni resultados en el servidor.

---

# 2. Relaciones entre clases de dominio

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

---

## Documentos relacionados

- `pandora-requerimientos.md` — reglas de negocio que definen el comportamiento de estas clases.
- `pandora-clases-diseno.md` — realización de estas clases en entidades, servicios y DTOs concretos.
- `pandora-modelo-tablas.md` — esquema físico de base de datos que persiste estas clases.
- `pandora-casos-de-uso.md` — casos de uso donde estas clases participan.
