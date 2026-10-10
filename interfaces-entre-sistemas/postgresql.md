# PostgreSQL

Almacenamiento primario y única fuente de verdad de todo el dominio (usuarios, obras, cartas, moderación, notificaciones, etc. — ver [modelo de tablas](/modelo-tablas/README.md)). Acceso exclusivamente vía TypeORM (`@nestjs/typeorm`), nunca SQL crudo fuera de los casos puntuales ya documentados en las clases de diseño (subqueries de filtro en `ArtworksService`).

Configuración (`src/config/database.config.ts`, variables `PG*`): `PGHOST`, `PGPORT`, `PGUSER`, `PGPASSWORD`, `PGDATABASE`, `PGSSLMODE`, `PGCHANNELBINDING`. `DB_SYNC` controla si TypeORM sincroniza el esquema automáticamente contra las entidades (`true` por defecto fuera de `NODE_ENV=production`) — en producción se espera manejar el esquema por migraciones versionadas en vez de `synchronize`, tal como recomienda [diseño de arquitectura, recomendación 10](/diseno-arquitectura/recomendaciones.md).

No hay *fallback*: si la conexión inicial falla, el proceso no llega a levantar el servidor HTTP.
