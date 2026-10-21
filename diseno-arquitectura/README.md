# Pandora — Diseño de Arquitectura

Este documento cubre la vista general del sistema, la organización por módulos, las capas, el almacenamiento de imágenes, los procesos periódicos (conversión, paquetes, batalla), la observabilidad y las recomendaciones de diseño.

---

## Índice

- [Vista general del sistema](#vista-general-del-sistema)
- [Arquitectura del Portal Web (Frontend)](#arquitectura-del-portal-web-frontend)
- [Organización del backend por módulos](organizacion-modulos.md)
  - [Capas recomendadas](organizacion-modulos.md#capas-recomendadas)
- [Observabilidad / logging (RNF14)](observabilidad.md)
- [Almacenamiento de imágenes (Cloudinary)](almacenamiento-imagenes.md)
  - [Flujo](almacenamiento-imagenes.md#flujo)
  - [Consideraciones](almacenamiento-imagenes.md#consideraciones)
- [Conversión de obras a cartas (proceso periódico)](conversion.md)
  - [Objetivo](conversion.md#objetivo)
  - [Flujo](conversion.md#flujo)
  - [Estados](conversion.md#estados)
  - [Idempotencia](conversion.md#idempotencia)
  - [Frecuencia del scheduler](conversion.md#frecuencia-del-scheduler)
  - [Disparo manual (Admin, demo)](conversion.md#disparo-manual-admin-demo)
- [Lógica de regeneración de paquetes](regeneracion-paquetes.md)
  - [Estado mínimo](regeneracion-paquetes.md#estado-mínimo)
  - [Cálculo](regeneracion-paquetes.md#cálculo)
- [Lógica de selección del rival (batalla)](seleccion-rival.md)
- [Recomendaciones de diseño para evitar deuda técnica](recomendaciones.md)
- [Resumen arquitectónico final](#resumen-arquitectónico-final)
- [Documento completo](completo.md) — todo el documento en una sola página

---

## Vista general del sistema

```mermaid
flowchart TB
    Web["Portal Web<br/>React + TypeScript"]
    App["App Android<br/>Java"]

    subgraph Backend["Backend — NestJS"]
        direction TB
        Guards["Guards / Middleware<br/>(auth, roles, rate limit)"]
        Controllers["Controllers"]
        Services["Services de dominio"]
        Repos["Repository / ORM"]
        Scheduler["Scheduler / CRON"]

        Guards --> Controllers --> Services --> Repos
        Scheduler --> Services
    end

    DB[("PostgreSQL")]
    Cloudinary[("Cloudinary")]
    Mail[("Resend / Mail")]

    Web --> Guards
    App --> Guards
    Repos --> DB
    Services --> Cloudinary
    Services --> Mail
```

Elementos genéricos a propósito: qué controller/service/guard concreto compone cada módulo vive en [diagramas de clases de diseño](/clases-diseno/README.md#diagramas-de-clases-de-diseño) y [diseño de componentes](/diseno-componentes/README.md). El detalle interno de Portal Web se desarrolla a continuación; el de App Android queda pendiente del lado de mobile.

El backend es la única fuente de autoridad para reglas de negocio: el frontend (web o Android) no debe reimplementar validaciones críticas, ya que ambos clientes comparten exactamente el mismo backend y contrato de API.

---

## Arquitectura del Portal Web (Frontend)

Detalle interno del nodo "Portal Web" del diagrama anterior. Es una SPA React + TypeScript (Vite) organizada por feature (`src/features/<dominio>/{api,hooks,store?,components,pages,types}`), sin lógica de negocio propia: toda regla de negocio vive en el backend, el cliente sólo la consume.

```mermaid
flowchart TB
    User(["Usuario"])

    subgraph WebApp["Portal Web — React + TypeScript"]
        direction TB
        Router["React Router<br/>(rutas públicas / protegidas por rol)"]
        Pages["Páginas y componentes<br/>(organizados por feature)"]
        State["Estado<br/>TanStack Query (caché de servidor) + Zustand (sesión/UI)"]
        ApiClient["Cliente HTTP<br/>Axios + interceptores (JWT, refresh automático)"]

        Router --> Pages --> State --> ApiClient
    end

    Backend[("Backend API<br/>NestJS")]

    User --> Router
    ApiClient --> Backend
```

Capas, de afuera hacia adentro:

- **React Router**: define las rutas públicas (explorar, detalle de obra/carta, login) y las protegidas (`RequireAuth`/`RequireVerified`/`RequireRole`), redirigiendo según el estado de sesión.
- **Páginas y componentes**: una carpeta por feature (`artworks`, `cards`, `battle`, `users`, etc.), cada una con sus propias páginas y componentes de UI (shadcn/ui + Tailwind).
- **Estado**: TanStack Query cachea y sincroniza todo dato que viene del servidor (listados, detalle, mutaciones); Zustand sólo guarda lo que no es caché de servidor — sesión (`accessToken`/`refreshToken`/usuario) y estado efímero de UI (p. ej. la simulación de batalla en curso).
- **Cliente HTTP**: una única instancia de Axios con interceptores — adjunta el header `Authorization`, normaliza todo error al formato de [convenciones de API §2](/api-conventions.md#2), y reintenta con refresh automático de token ante un `401`.

El frontend nunca recalcula reglas de negocio (rareza, estadísticas de carta, resultado de un reporte, etc.): las consume tal como las devuelve el backend. La única excepción documentada es el simulador de batalla, que corre 100% en el cliente por diseño (ver [convenciones de API §15](/api-conventions.md#15) y `battle-formula.md`).

---

## Resumen arquitectónico final

Pandora queda definido como un sistema cliente-servidor compuesto por un portal web y una aplicación Android que consumen una API NestJS. PostgreSQL centraliza usuarios, contenido, calificaciones, cartas, inventarios, relaciones, moderación y notificaciones. Cloudinary gestiona los assets gráficos.

La principal secuencia de negocio es:

```mermaid
flowchart TD
    A["Usuario publica obra"] --> B["Período Q2Q de 7 días"]
    B -.-> B1["Comentarios continúan habilitados"]
    B --> C["Fin de período"]
    C --> D["Scheduler detecta obras elegibles"]
    D --> E["Conversión a carta"]
    E --> F["Carta disponible para colección"]
    F --> G["Paquetes"]
    G --> H["Colección del usuario"]
    H --> I["Autobattler local"]
```
