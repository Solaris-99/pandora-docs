# Lorem Picsum (`picsum.photos`)

**Rol:** imágenes placeholder para el catálogo "de sistema" que garantiza que siempre exista al menos una carta de cada rareza (`RARITY_SEED_CATALOG`, `src/cards/rarity-seed-catalog.ts`). **El backend nunca hace una petición HTTP a Picsum** — sólo construye una URL determinística (`https://picsum.photos/seed/pandora-<rareza>/600/800`) y la guarda como `image_original_url`/`image_medium_url`/`image_thumbnail_url` de esas obras semilla. Quien efectivamente descarga la imagen es el navegador/app del usuario final, al renderizar esa URL — igual que cualquier otra imagen servida por Cloudinary.

Dos consumidores comparten esta función:

- **`CardCatalogSeederService`** (`onApplicationBootstrap`): corre en **cada arranque del servidor, en cualquier ambiente** (no es un script manual) — autorepara el catálogo de rarezas bajo la cuenta de sistema `pandora` si falta alguna.
- **`scripts/seed-admin-catalog.ts`** (ejecución manual, sólo desarrollo): mismo mecanismo, bajo la cuenta admin real — ver `docs/scripts-inicializacion.md`.

**Por qué Picsum y no Cloudinary acá:** deliberado — el arranque del servidor (y el autoreparado del catálogo mínimo) no debe depender de que `CLOUDINARY_*` esté configurado. Usar un hotlink público y determinístico evita acoplar el arranque a credenciales externas y no consume cuota de Cloudinary en cada reinicio.

Si Picsum estuviera caído, el único efecto visible es que esas imágenes puntuales (obras semilla, no las de la comunidad) se ven rotas hasta que Picsum vuelva — no afecta la disponibilidad del backend ni de ninguna otra imagen.
