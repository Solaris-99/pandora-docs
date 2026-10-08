---
name: Pandora — Obsidian Violet
colors:
  dark:
    surface: '#14121a'
    surface-dim: '#14121a'
    surface-bright: '#3a3840'
    surface-container-lowest: '#0e0d14'
    surface-container-low: '#1c1b22'
    surface-container: '#201f26'
    surface-container-high: '#2a2931'
    surface-container-highest: '#35343c'
    on-surface: '#e5e0eb'
    on-surface-variant: '#cbc3d7'
    inverse-surface: '#e5e0eb'
    inverse-on-surface: '#312f37'
    outline: '#958ea0'
    outline-variant: '#494454'
    surface-tint: '#d0bcff'
    primary: '#d0bcff'
    on-primary: '#3c0091'
    primary-container: '#a078ff'
    on-primary-container: '#340080'
    inverse-primary: '#6d3bd7'
    secondary: '#ddb8ff'
    on-secondary: '#490081'
    secondary-container: '#62259b'
    on-secondary-container: '#d1a1ff'
    tertiary: '#abd600'
    on-tertiary: '#283500'
    tertiary-container: '#7c9c00'
    on-tertiary-container: '#222e00'
    error: '#ffb4ab'
    on-error: '#690005'
    error-container: '#93000a'
    on-error-container: '#ffdad6'
    background: '#14121a'
    on-background: '#e5e0eb'
    surface-variant: '#35343c'
  light:
    surface: '#fffbff'
    surface-dim: '#ded8e1'
    surface-bright: '#fffbff'
    surface-container-lowest: '#ffffff'
    surface-container-low: '#f7f2fa'
    surface-container: '#f1ecf4'
    surface-container-high: '#ece6ee'
    surface-container-highest: '#e6e0e9'
    on-surface: '#1c1b1f'
    on-surface-variant: '#49454e'
    inverse-surface: '#312f37'
    inverse-on-surface: '#f5eff7'
    outline: '#7a7580'
    outline-variant: '#cac4d0'
    surface-tint: '#6d3bd7'
    primary: '#6d3bd7'
    on-primary: '#ffffff'
    primary-container: '#eaddff'
    on-primary-container: '#22005d'
    inverse-primary: '#d0bcff'
    secondary: '#7a4fa8'
    on-secondary: '#ffffff'
    secondary-container: '#f6d8ff'
    on-secondary-container: '#2e0a47'
    tertiary: '#586300'
    on-tertiary: '#ffffff'
    tertiary-container: '#dcea7e'
    on-tertiary-container: '#1a1e00'
    error: '#ba1a1a'
    on-error: '#ffffff'
    error-container: '#ffdad6'
    on-error-container: '#410002'
    background: '#fffbff'
    on-background: '#1c1b1f'
    surface-variant: '#e6e0e9'
  fixed:
    primary-fixed: '#e9ddff'
    primary-fixed-dim: '#d0bcff'
    on-primary-fixed: '#23005c'
    on-primary-fixed-variant: '#5516be'
    secondary-fixed: '#f0dbff'
    secondary-fixed-dim: '#ddb8ff'
    on-secondary-fixed: '#2c0051'
    on-secondary-fixed-variant: '#62259b'
    tertiary-fixed: '#c3f400'
    tertiary-fixed-dim: '#abd600'
    on-tertiary-fixed: '#161e00'
    on-tertiary-fixed-variant: '#3c4d00'
typography:
  display-xl:
    fontFamily: Syne
    fontSize: 72px
    fontWeight: '800'
    lineHeight: 80px
    letterSpacing: -0.03em
  display-xl-mobile:
    fontFamily: Syne
    fontSize: 40px
    fontWeight: '800'
    lineHeight: 46px
    letterSpacing: -0.02em
  headline-lg:
    fontFamily: Syne
    fontSize: 48px
    fontWeight: '700'
    lineHeight: 56px
    letterSpacing: -0.02em
  headline-lg-mobile:
    fontFamily: Syne
    fontSize: 32px
    fontWeight: '700'
    lineHeight: 38px
    letterSpacing: -0.015em
  headline-md:
    fontFamily: Syne
    fontSize: 32px
    fontWeight: '700'
    lineHeight: 40px
    letterSpacing: -0.01em
  headline-md-mobile:
    fontFamily: Syne
    fontSize: 24px
    fontWeight: '600'
    lineHeight: 32px
    letterSpacing: -0.01em
  headline-sm:
    fontFamily: Syne
    fontSize: 20px
    fontWeight: '600'
    lineHeight: 28px
    letterSpacing: 0em
  body-lg:
    fontFamily: Plus Jakarta Sans
    fontSize: 18px
    fontWeight: '400'
    lineHeight: 28px
    letterSpacing: -0.01em
  body-md:
    fontFamily: Plus Jakarta Sans
    fontSize: 15px
    fontWeight: '400'
    lineHeight: 24px
    letterSpacing: 0em
  body-sm:
    fontFamily: Plus Jakarta Sans
    fontSize: 13px
    fontWeight: '400'
    lineHeight: 20px
    letterSpacing: 0.01em
  label-lg:
    fontFamily: JetBrains Mono
    fontSize: 14px
    fontWeight: '600'
    lineHeight: 20px
    letterSpacing: 0.06em
  label-md:
    fontFamily: JetBrains Mono
    fontSize: 12px
    fontWeight: '500'
    lineHeight: 16px
    letterSpacing: 0.08em
  label-sm:
    fontFamily: JetBrains Mono
    fontSize: 10px
    fontWeight: '500'
    lineHeight: 14px
    letterSpacing: 0.1em
rounded:
  sm: 0.125rem
  DEFAULT: 0.25rem
  md: 0.375rem
  lg: 0.5rem
  xl: 0.75rem
  full: 9999px
spacing:
  gutter: 1.5rem
  gutter-mobile: 0.75rem
  margin: 3rem
  margin-mobile: 1.25rem
  space-xs: 0.25rem
  space-sm: 0.5rem
  space-md: 1rem
  space-lg: 1.5rem
  space-xl: 2.5rem
  space-2xl: 4rem
---

> Fuente de verdad del sistema de diseño de Pandora, para que web y mobile compartan la misma identidad visual — igual que `battle-formula.md` para la mecánica de combate. Se escribió a partir de una plantilla de referencia (paleta M3 + especificación de componentes) adaptada al dominio real de Pandora (obras → calificación → cartas → colección → batalla), no una traducción literal.
>
> **Relación con la implementación web actual:** `src/index.css` ya tiene un acento violeta aplicado (ver `docs/frontend-todo.md`), pero sólo cubre los roles de shadcn (`primary`, `secondary`, `accent`, `muted`, `border`...) — no todavía la paleta M3 completa de acá (roles `tertiary`, niveles de `surface-container`, paleta clara derivada formalmente, tipografía Syne/Plus Jakarta Sans/JetBrains Mono, radios más angostos). Migrar el CSS del frontend para que coincida 1:1 con este documento es un paso aparte, todavía no hecho — ver tabla de mapeo al final.

## Marca y estilo

Pandora es una comunidad donde el arte se convierte en cartas coleccionables: se sube una obra, la comunidad la califica durante una semana (Q2Q), y al cierre se genera una carta con estadísticas de combate según la rareza obtenida. El sistema de diseño tiene que servir dos modos a la vez — **galería** (obras, perfiles, exploración) y **juego de cartas** (colección, rareza, batalla) — sin que ninguno de los dos se sienta como un añadido.

Rasgos clave:
- **Obsidiana nocturna:** superficies oscuras y profundas (`surface` / `surface-container-*`) que dejan que el arte subido por los usuarios sea el punto de mayor contraste de la pantalla — nunca compite cromáticamente con las miniaturas.
- **Precisión eléctrica:** el violeta (`primary`) y la lila (`secondary`) marcan estados interactivos e indicadores vivos con intención quirúrgica, no decorativa — se usan donde hay una acción o un estado, no como relleno.
- **Rigor de catálogo:** bordes definidos, líneas estructurales y tipografía editorial (Syne) le dan al catálogo de obras/cartas una sensación de archivo curado, no de red social genérica.
- **Señal de rareza:** el verde lima (`tertiary`) queda reservado a lo que de verdad importa destacar — rareza "Legendaria", insignias de verificación, indicadores en vivo — nunca para texto o superficies comunes.

## Colores

### Roles principales (M3)

| Rol | Uso en Pandora |
| --- | --- |
| `primary` / `on-primary` | Acción principal: botón de publicar, "Calificar", "Batallar", links activos, foco de inputs. |
| `primary-container` / `on-primary-container` | Superficie suave para resaltar sin gritar: fondo de insignia de tag activo, fondo de ícono de feature en `/about`, estado "seleccionado" de un filtro. |
| `secondary` / `on-secondary` | Acciones secundarias con intención propia (no un simple "botón gris"): "Seguir", contador de favoritos, chips de metadatos vivos. |
| `secondary-container` / `on-secondary-container` | Variante suave de lo anterior, mismo criterio que `primary-container`. |
| `tertiary` / `on-tertiary` | **Reservado.** Insignia de rareza Legendaria, verificación de cuenta, punto de notificación no leída, badge "Curador's Choice"-style (ej. "Obra destacada"). Nunca como color de texto de cuerpo ni fondo general. |
| `error` / `on-error` / `error-container` | Igual que `destructive` ya usado en el frontend (toasts de error, validación, eliminar/banear). |
| `surface` / `on-surface` | Fondo de página y texto principal. |
| `surface-container-low/DEFAULT/high/highest` | Jerarquía de superficies elevadas: tarjetas, popovers, modales, dropdowns, en ese orden creciente de elevación. |
| `on-surface-variant` | Texto secundario / metadatos (igual rol que `muted-foreground` hoy). |
| `outline` / `outline-variant` | Bordes con intención (`outline`) vs. bordes casi invisibles de separación (`outline-variant`). |
| `inverse-surface` / `inverse-on-surface` / `inverse-primary` | Elementos que necesitan "saltar" sobre el tema actual — tooltips, snackbars sobre un fondo ya oscuro. |
| `*-fixed` / `*-fixed-dim` / `on-*-fixed` | Mismos valores en claro y oscuro — para elementos que cruzan ambos temas en la misma pantalla (ej. una miniatura de carta con overlay que no debe cambiar con el tema). |

Los valores exactos (hex) están en el frontmatter de este documento, separados en `dark` (tema por defecto, ver "Elección de tema por defecto" abajo) y `light`. Los `fixed` son los mismos en ambos.

**Elección de tema por defecto:** a diferencia del resto de la app (que hoy respeta la preferencia del SO vía `next-themes`), la paleta oscura es la identidad "de marca" principal — está pensada primero para que el arte resalte. La paleta clara existe y tiene que ser igual de usable, pero cualquier superficie de marketing/landing (ej. el Hero de `/`) puede sesgarse hacia el tema oscuro si hace falta.

**Nota sobre la derivación de la paleta clara:** los valores de `light` están construidos para mantener el mismo rol y la misma familia de tono que `dark` (y dos de ellos, `primary` y `inverse-surface`, salen directamente de los campos `inverse-*` ya provistos por el generador de la paleta oscura, no son inventados). No corren el algoritmo HCT de Material exacto — si en algún momento mobile necesita paridad pixel-perfecta con `ColorScheme` nativo de Android/Material3, regenerar ambos temas juntos desde *Material Theme Builder* con semilla `#8B5CF6` y confirmar que coincide con lo documentado acá (y actualizar este archivo si no).

### Colores funcionales (fuera del rol M3, ya implementados)

Estos son convenciones del frontend que preceden a este documento y **se mantienen** — no las reemplaza la paleta de arriba, conviven con ella:

- **Stats de carta** (`CardDetailPage`, `CardTile`, `BattleArena`): ataque en rojo (`red-600`/`red-400`), defensa en ámbar (`amber-600`/`amber-400`), vida en esmeralda (`emerald-600`/`emerald-400`), velocidad en celeste (`sky-600`/`sky-400`). Son colores "de juego" (icono + número), no roles de marca — no se tocan.
- **Log de batalla**: turnos del jugador en celeste, del rival en rojo, resultado en esmeralda (victoria) o rojo (derrota) — ver `docs/frontend-todo.md`.
- **Rareza de carta** (`common` → `legendary`): hoy sólo usa 3 variantes genéricas de `Badge` (`outline`/`secondary`/`default`). Este documento propone — a confirmar antes de implementar — una escala propia de 5 pasos para que cada rareza se reconozca de un vistazo en la colección:

  | Rareza | Color sugerido |
  | --- | --- |
  | Común | `on-surface-variant` sobre `surface-container` (neutro) |
  | Poco común | verde — reusar el `emerald` ya usado para HP |
  | Rara | celeste — reusar el `sky` ya usado para velocidad |
  | Épica | `secondary` (lila) |
  | Legendaria | `tertiary` (lima) — único uso de `tertiary` compartido con el resto del sistema |

## Tipografía

Tres familias con roles bien separados — igual criterio que la plantilla de referencia:

- **Syne** (`display-*`, `headline-*`): expresiva, geométrica, con personalidad — títulos de página, nombre de obra/carta en su detalle, hero.
- **Plus Jakarta Sans** (`body-*`): lectura continua — descripciones, comentarios, texto de formularios.
- **JetBrains Mono** (`label-*`): metadatos y datos de precisión — nombres de usuario (`@handle`), timestamps, contadores, stats de carta, códigos de rareza. Reemplaza el uso ad-hoc de `font-medium` en metadatos por una familia dedicada que comunica "este es un dato, no prosa".

El frontend web hoy sólo carga **Geist Variable** (`@fontsource-variable/geist`) para todo. Adoptar esta escala de tres familias es parte del trabajo pendiente de aplicar este documento al código (ver nota al principio) — hay que sumar `Syne`, `Plus Jakarta Sans` y `JetBrains Mono` vía `@fontsource` o Google Fonts.

Las escalas `*-mobile` son los tamaños a usar por debajo de 768px — mismo criterio de "achicar pero no perder jerarquía" que ya sigue el breakpoint `sm:`/`md:` de Tailwind en el proyecto.

## Layout y espaciado

Grilla de 12 columnas con `margin`/`gutter` fijos en vez de espaciado arbitrario:

- **Desktop (1200px+):** 12 columnas, `margin: 3rem`, `gutter: 1.5rem`. El explorador de obras/cartas usa una grilla de 4 columnas (`xl:grid-cols-4`, ya implementado); el detalle de obra/carta puede usar proporciones 7:5 (imagen:metadata), como en el Hero de referencia.
- **Tablet (768–1199px):** 8 columnas, 2rem de margen, 1rem de gutter. Las grillas de 4 columnas bajan a 2.
- **Mobile (<768px):** 4 columnas, `margin-mobile: 1.25rem`, `gutter-mobile: 0.75rem`. Todo colapsa a 1–2 columnas (ya el criterio actual de `grid-cols-2` en `ExplorePage`/`CollectionPage`).

La escala `space-xs` → `space-2xl` reemplaza los valores sueltos de Tailwind (`gap-2`, `gap-3`, `gap-4`...) una vez que se aplique al código — ver tabla de mapeo.

## Elevación y profundidad

Igual que la plantilla: **sin sombras difusas**. La jerarquía se construye con superficie + borde, no con `box-shadow` suave:

- **Capa 0 (lienzo):** `surface` / `background`.
- **Capa 1 (paneles, tarjetas de obra/carta):** `surface-container-low`, borde 1px `outline-variant`.
- **Capa 2 (overlays flotantes — modales, dropdowns, popovers):** `surface-container-high`, borde 1px en `primary/40`.
- **Resplandor de foco eléctrico** — reservado a estados interactivos activos (no decorativo): `box-shadow: 0 0 0 1px var(--primary), 0 0 24px -4px color-mix(in oklch, var(--primary) 45%, transparent)`.

## Formas

Radio **más angosto y estructural** que el actual (`--radius: 0.625rem` en `index.css` hoy, bastante más redondeado que esto):

- Controles interactivos core (inputs, botones, chips): `rounded` (0.25rem / 4px).
- Tarjetas, paneles, modales: `rounded-lg` (0.5rem / 8px) — un paso más que hoy en `card.tsx` (`rounded-xl`), a achicar si se aplica este documento.
- Insignias/badges: esquinas filosas, `rounded-sm` (2px) en vez del `rounded-full` (píldora) que usa `badge.tsx` hoy — cambio de forma deliberado, no sólo de color, para reforzar el carácter "de catálogo" en vez de "red social".
- Círculo completo (`rounded-full`) reservado a avatares y controles circulares (ej. el picker de batalla) — como ya es hoy.

## Componentes

### Botones
- **Primario:** fondo `primary`, texto `on-primary`, tipografía `label-md` en mayúsculas. Hover: `color-mix` hacia un tono más oscuro + resplandor eléctrico (ver Elevación). Mapea al `Button` default de shadcn.
- **Secundario / ghost:** fondo transparente, texto `on-surface`, borde `outline-variant`. Hover: borde y texto pasan a `secondary`. Mapea a `variant="outline"`/`variant="ghost"`.
- **Acción de alto compromiso:** fondo `tertiary`, texto `on-tertiary` — reservado a muy pocas acciones por pantalla (ej. "Abrir paquete", "Confirmar batalla"), nunca el botón por defecto de un formulario.

### Chips e insignias
- Radio `rounded-sm`, tipografía `label-sm`, borde 1px.
  - Estándar (tags, filtros inactivos): fondo `surface-container`, borde `outline-variant`, texto `on-surface-variant`.
  - Activo/seleccionado: fondo `primary-container`, texto `on-primary-container`.
  - Señal (rareza Legendaria, verificado, "en vivo"): fondo `tertiary` al 8–10% de opacidad, borde `tertiary`, texto `tertiary`.

### Tarjetas de obra/carta
- Superficie `surface-container-low`, borde 1px `outline-variant`, imagen a ancho completo con separador de 1px antes de los metadatos — mismo patrón que `ArtworkCard`/`CardTile` ya siguen, sólo migrando los colores concretos a los roles de este documento.
- Hover: borde pasa a `primary`, título pasa a `primary` (ya es el comportamiento de `CardTile`/`ArtworkCard` con `group-hover:text-primary`, se mantiene).

### Inputs y búsqueda
- Fondo `surface-container-low`, borde 1px `outline-variant`, texto `on-surface`, placeholder `outline`.
- Foco: borde `primary` 1px, sin outline de navegador — ya el comportamiento de `Input`/`Combobox` de shadcn, sólo cambia la variable de color subyacente.

### Checkboxes y toggles
- Checkbox cuadrado, radio `sm`. Sin marcar: borde 1px `outline`. Marcado: fondo `primary`, check en `on-primary`.
- Toggle: track `surface-container-high` con borde `outline-variant`; thumb pasa a `secondary` cuando está activo.

### Listas indexadas (ranking, cola de moderación, notificaciones)
- Filas de ancho completo con borde inferior 1px `outline-variant`. Numeral/índice en `label-sm` monoespaciado, color `outline`. Hover: fondo `primary` al 6% de opacidad.

## Tabla de mapeo: roles M3 → variables actuales de `index.css`

Para cuando se encare la migración del CSS del frontend — qué variable de shadcn/Tailwind le corresponde a cada rol nuevo:

| Rol M3 | Variable actual (`index.css`) | Nota |
| --- | --- | --- |
| `background` / `surface` | `--background` | Ya existen y coinciden en significado. |
| `on-background` / `on-surface` | `--foreground` | — |
| `on-surface-variant` | `--muted-foreground` | — |
| `surface-container` | `--card` / `--popover` | Hoy comparten un solo tono; este documento pide 4 niveles (`low`/`DEFAULT`/`high`/`highest`) — variables nuevas a agregar. |
| `outline` | `--border` | — |
| `outline-variant` | *(no existe)* | Nueva variable — hoy no hay una versión "más tenue" de `--border`. |
| `primary` / `on-primary` | `--primary` / `--primary-foreground` | Ya aplicado (ver nota al principio). |
| `primary-container` / `on-primary-container` | `--accent` / `--accent-foreground` | El acento ya tintado de violeta hoy cumple casi exactamente este rol. |
| `secondary` / `on-secondary` | `--secondary` / `--secondary-foreground` | Hoy siguen siendo neutros (gris) — pendiente de tintar a lila. |
| `tertiary` / `on-tertiary` / `*-container` | *(no existe)* | Rol nuevo — variables a agregar, sin equivalente hoy. |
| `error` / `on-error` | `--destructive` / *(implícito)* | — |
| `radius` (`rounded`) | `--radius` | Valor actual más grande que lo que pide este documento — a achicar. |

## Fuera de alcance de esta versión

- Íconos: la plantilla de referencia usa Material Symbols; el proyecto ya usa `lucide-react` en todos lados — **no se cambia la librería de íconos**, sólo su color sigue los roles de arriba.
- Modo claro para mobile nativo (Android `ColorScheme`/Compose Material3): los valores de `light`/`dark`/`fixed` de este documento son compatibles directamente si mobile usa Jetpack Compose + Material3, pero no se generó ni se validó un `Theme.kt` — queda para cuando se encare esa implementación.
- Migrar el CSS/componentes del frontend web a este sistema completo (tipografía de tres familias, radios angostos, niveles de `surface-container`, rol `tertiary`) es trabajo aparte, no incluido en este documento — ver nota al principio y la tabla de mapeo.
