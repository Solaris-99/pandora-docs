# CU24 — Notificaciones

**Actor principal:** Usuario autenticado.

## Flujo principal

1. Ocurre un evento notificable.
2. El backend crea una notificación.
3. El usuario consulta el panel.
4. Se devuelven notificaciones paginadas.
5. El usuario marca una o varias como leídas.

## Eventos iniciales

- usuario seguido;
- nuevo upload de usuario seguido;
- fin de período de calificación;
- comentario realizado sobre una obra;
- comentario eliminado/baneado por moderación;
- resultado de una denuncia;
- actividad de votos/calificaciones relevante.

> El catálogo completo de tipos de notificación (incluyendo los agregados por moderación y apelaciones) vive en las [clases de dominio](/clases-dominio.md), clase `Notification`.
