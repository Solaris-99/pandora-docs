# Pandora — Interfaces entre Sistemas

Este documento detalla las interacciones entre el backend de Pandora (como sistema completo) y los sistemas externos de los que depende: PostgreSQL, Cloudinary, Google (OAuth), Resend y Lorem Picsum. No cubre la interfaz entre el backend y sus propios clientes de primera parte (Portal Web, App Android) — eso vive en [diseño de arquitectura, vista general](/diseno-arquitectura/README.md#vista-general-del-sistema) (vista general) y en [convenciones de API](/api-conventions.md) (contrato REST). Tampoco redocumenta el detalle interno de cada integración (eso vive en [clases de diseño](/clases-diseno/README.md), en las clases `StorageService`, `MailService` y `AuthService`) — el foco acá es el protocolo, la dirección y el comportamiento ante fallos de cada interfaz externa.

---

## Índice

- [Resumen](#resumen) — cada fila enlaza al sistema
- [PostgreSQL](postgresql.md)
- [Cloudinary (almacenamiento de imágenes)](cloudinary.md)
  - [Subida (`uploadArtworkImage`)](cloudinary.md#subida-uploadartworkimage)
  - [Borrado (`deleteArtworkImage`)](cloudinary.md#borrado-deleteartworkimage)
  - [Límite conocido](cloudinary.md#límite-conocido)
- [Google (identidad federada / OAuth)](google.md)
  - [Flujo Web — authorization code (Passport)](google.md#flujo-web-authorization-code-passport)
  - [Flujo Android — verificación de ID token](google.md#flujo-android-verificación-de-id-token)
  - [Vinculación de cuenta (común a ambos flujos)](google.md#vinculación-de-cuenta-común-a-ambos-flujos)
- [Resend (correo transaccional)](resend.md)
  - [Correos enviados](resend.md#correos-enviados)
  - [Manejo de fallos](resend.md#manejo-de-fallos)
- [Lorem Picsum (`picsum.photos`)](lorem-picsum.md)
- [Consideraciones transversales](consideraciones-transversales.md)
- [Documento completo](completo.md) — todo el documento en una sola página

---

## Resumen

| Sistema | Rol | Protocolo | Dirección | ¿Obligatorio para arrancar? | Modo de fallo |
|---|---|---|---|---|---|
| [PostgreSQL](postgresql.md) | Almacenamiento primario de datos | TCP/SQL (TypeORM) | Backend → PostgreSQL | Sí | El proceso no arranca sin conexión a la base |
| [Cloudinary](cloudinary.md) | Almacenamiento/CDN de imágenes de obras | HTTPS REST (SDK oficial `cloudinary`) | Backend → Cloudinary | No (el proceso arranca igual) | Sin credenciales: cualquier subida responde `500 INTERNAL_SERVER_ERROR` explícito |
| [Google](google.md) | Identidad federada (login social) | OAuth2 authorization code (web) / verificación de ID token (Android) | Backend ↔ Google | No | Sin credenciales: las rutas `/auth/google*` fallan; login local no se ve afectado |
| [Resend](resend.md) | Envío de correo transaccional | HTTPS REST (SDK oficial `resend`) | Backend → Resend | No | Sin API key: modo *log* local (el correo se imprime en logs, no se bloquea el flujo) |
| [Lorem Picsum](lorem-picsum.md) | Imágenes placeholder para el catálogo semilla | URL hotlink — **el backend no llama a Picsum**, sólo construye la URL | Cliente (navegador/app) → Picsum, referenciado por el backend | No | Si Picsum cae, sólo se ven rotas esas imágenes puntuales; no afecta al backend |

Sólo PostgreSQL es una dependencia dura de arranque. Las otras cuatro son deliberadamente *fail-soft* a nivel de proceso: el servidor levanta igual sin credenciales configuradas y sólo falla (o degrada) la operación puntual que las necesita — ver [diseño de arquitectura, observabilidad](/diseno-arquitectura/observabilidad.md) para el detalle de logging de cada una.
