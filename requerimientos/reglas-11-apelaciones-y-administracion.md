# 5.11 Apelaciones y administración

1. Toda apelación remite a un único reporte ya resuelto con una sanción punitiva (`content_removed`, `comment_removed` o `user_banned`); un reporte resuelto como `dismissed` o `warning` no tiene nada que apelar ni que reabrir.
2. Un reporte sólo puede apelarse una vez: si la apelación es rechazada, no puede volver a apelarse la misma sanción.
3. Sólo el autor de la obra/comentario sancionado, o el propio usuario baneado, puede apelar esa sanción.
4. El moderador o admin que revisa una apelación no puede ser quien resolvió el reporte original, salvo que en ese momento exista exactamente un miembro de personal de moderación (`moderator` + `admin` contados juntos).
5. Aprobar una apelación, o reabrir un reporte directamente como admin, es una reversión de un solo sentido: deshace exactamente el efecto de la sanción original (restaura el contenido o desbanea al usuario). No permite reemplazar la sanción por otra distinta en el mismo paso.
6. Un usuario baneado puede autenticarse igual que cualquier otro usuario, pero sus operaciones de escritura quedan bloqueadas salvo apelar su propio baneo, consultar el estado de sus apelaciones y consultar/gestionar sus notificaciones.
7. La suspensión (`UserStatus.suspended`) no forma parte del sistema de apelaciones: una cuenta suspendida sigue sin poder autenticarse, sin excepción.
