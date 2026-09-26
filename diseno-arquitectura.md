# Pandora — Diseño de Arquitectura

Este documento cubre la vista general del sistema, la organización por módulos, las capas, el almacenamiento de imágenes, los procesos periódicos (conversión, paquetes, batalla), la observabilidad y las recomendaciones de diseño.

---

# 1. Vista general del sistema

```text
+----------------------+       +----------------------+
|     Portal Web       |       |    Android App       |
| React + TypeScript   |       | Java                 |
+----------+-----------+       +-----------+----------+
           |                               |
           +---------------+---------------+
                           |
                           v
                  +-------------------+
                  |    REST API       |
                  | NestJS + TypeScript|
                  +---------+---------+
                            |
             +--------------+--------------+
             |                             |
             v                             v
      +-------------+               +-------------+
      | PostgreSQL  |               |  Cloudinary |
      +-------------+               +-------------+

                    +-------------------+
                    | Scheduler / CRON  |
                    | Conversion Worker |
                    +-------------------+
```

El backend es la única fuente de autoridad para reglas de negocio: el frontend (web o Android) no debe reimplementar validaciones críticas, ya que ambos clientes comparten exactamente el mismo backend y contrato de API.

---

# 2. Organización del backend por módulos

Estructura de `src/` en NestJS, un módulo por dominio funcional:

```text
src/
├── auth/
├── users/
├── roles/
├── artworks/
├── ratings/
├── comments/
├── cards/
├── collections/
├── packages/
├── battle/
├── follows/
├── favorites/
├── notifications/
├── reports/
├── moderation/
├── tags/
├── storage/
├── conversion/
├── scheduler/
├── common/
└── database/
```

## 2.1 Capas recomendadas

```text
Controller
    ↓
Application / Service
    ↓
Domain logic
    ↓
Repository / ORM
    ↓
PostgreSQL
```

El detalle de qué controladores, servicios y entidades componen cada módulo vive en [clases de diseño](/clases-diseno.md) y [diseño de componentes](/diseno-componentes.md) (endpoints y guards).

---

# 3. Observabilidad / logging (RNF14)

- **Log de acceso HTTP** (`src/common/middleware/logging.middleware.ts`): un log por request (`método ruta status duración - ip`), a nivel `log` para 2xx/3xx y `warn` para 4xx/5xx. Deliberadamente implementado como **middleware de Express** (`app.use(...)` en `main.ts`), no como interceptor de Nest: los guards (`JwtAuthGuard`, `RolesGuard`, `AppThrottlerGuard`, etc.) corren *antes* que los interceptors en el pipeline de Nest, así que un interceptor nunca vería una request rechazada por un guard (401/403/429) — justo el tráfico más interesante de registrar para seguridad. El middleware corre primero de todos modos y se engancha a `res.on('finish')`, que se dispara sin importar en qué capa terminó resolviéndose la respuesta. Verificado en vivo: una request no autenticada (401) y una bloqueada por rate limiting (429) quedan registradas igual que una exitosa.
- **Errores no manejados:** `AllExceptionsFilter` registra cualquier excepción no capturada, con stack trace.
- **Ejecuciones del CRON:** `ConversionSchedulerService` registra un resumen por ciclo (procesadas/convertidas/fallidas/skipped).
- **Fallas de integraciones externas:** `MailService` (Resend) y `StorageService` (Cloudinary) registran sus fallos sin abortar el flujo que los llamó (best-effort — ver 4).

---

# 4. Almacenamiento de imágenes (Cloudinary)

Se utiliza **Cloudinary** para las imágenes de obras.

## 4.1 Flujo

```text
Cliente
  |
  v
Backend valida metadata
  |
  v
Subida a Cloudinary
  |
  +--> original
  +--> thumbnail
  +--> medium
  |
  v
Backend guarda URLs / public_id
  |
  v
Artwork disponible
```

## 4.2 Consideraciones

- No guardar blobs de imagen directamente en PostgreSQL.
- Guardar identificadores/URLs de Cloudinary.
- Utilizar transformaciones para thumbnails.
- Mantener acceso público sólo a los recursos que correspondan.
- Definir política de borrado de Cloudinary al eliminar una obra.
- No confiar sólo en la extensión del archivo: verificar MIME/type y contenido real (firma binaria).

---

# 5. Conversión de obras a cartas (proceso periódico)

## 5.1 Objetivo

Evitar conversiones manuales y garantizar que una obra sólo sea convertida después de finalizar su semana de calificación.

## 5.2 Flujo

```text
CRON/Scheduler
  |
  v
Buscar artworks:
  qualification_started_at >= 7 dias
  conversion_request = true
  conversion_status = pending/failed reintentable
  |
  v
Tomar trabajo
  |
  v
Marcar processing
  |
  v
Obtener ratings + emociones + comentarios
  |
  v
Ejecutar CardConversionService
  |
  +--> attack
  +--> defense
  +--> hp
  +--> speed
  +--> rarity
  |
  v
Crear Card
  |
  v
Marcar converted
  |
  v
Notificación
```

## 5.3 Estados

```text
pending -> processing -> converted
                  \-> failed
pending -> skipped
```

## 5.4 Idempotencia

Debe existir una protección de base de datos mediante `UNIQUE(artwork_id)` en `cards`. El servicio debe utilizar transacciones para impedir que dos instancias del scheduler creen la misma carta.

## 5.5 Frecuencia del scheduler

El scheduler se dispara a las 00:00, diariamente, mediante el Scheduler nativo de NestJS (`@nestjs/schedule`).

## 5.6 Disparo manual (Admin, demo)

La rutina de conversión (5.2–5.4) se expone también como método invocable independientemente del disparador de Cron, y un endpoint de administración (`POST /conversion/run`, CU22) la ejecuta bajo demanda. No es una segunda implementación: es la misma lógica, sólo que disparada por una request en lugar de por el reloj — útil para demos donde no tiene sentido esperar a que termine una semana de calificación real.

---

# 6. Lógica de regeneración de paquetes

Se requiere un máximo de 10 paquetes y una regeneración de 1 paquete por minuto.

## 6.1 Estado mínimo

```text
amount
last_regeneration_at
```

## 6.2 Cálculo

Al consultar o abrir paquetes:

1. calcular minutos completos transcurridos;
2. determinar cuántos paquetes deben regenerarse;
3. limitar el resultado a 10;
4. actualizar `last_regeneration_at` de forma consistente;
5. si se llega a 10, evitar acumular tiempo pendiente de manera que al bajar a 9 aparezcan paquetes de golpe no previstos por el diseño.

La apertura debe ejecutarse con protección de concurrencia para que dos requests no consuman un único paquete dos veces.

---

# 7. Lógica de selección del rival (batalla)

El endpoint de rival debe cumplir las restricciones de negocio sin asumir que se trata de una partida multijugador.

```text
GET /battle/opponent
        |
        v
Seleccionar carta válida aleatoria
        |
        v
Devolver datos necesarios
```

El endpoint no debe crear una "partida" persistente. La separación entre backend y cliente es especialmente importante en el juego: el backend mantiene únicamente la autoridad necesaria sobre inventario y selección del rival, mientras la simulación de batalla no se considera información de negocio persistente.

---

# 8. Recomendaciones de diseño para evitar deuda técnica

1. Mantener la generación de cartas como servicio de dominio independiente del Scheduler.
2. No mezclar entidades TypeORM con DTOs de entrada.
3. No confiar en restricciones del frontend para autorización.
4. Aplicar ownership checks además de roles.
5. Usar transacciones para apertura de paquetes y conversión de cartas.
6. Registrar estados de procesos que puedan reintentarse.
7. Mantener soft delete en contenido moderado cuando la auditoría lo requiera.
8. Mantener la lógica de batalla fuera del backend mientras no haya persistencia ni interacción remota.
9. Evitar endpoints que existan sólo porque "podrían hacer falta" pero no tengan una operación de dominio real.
10. Versionar las migraciones de PostgreSQL desde el comienzo.

---

# 9. Resumen arquitectónico final

Pandora queda definido como un sistema cliente-servidor compuesto por un portal web y una aplicación Android que consumen una API NestJS. PostgreSQL centraliza usuarios, contenido, calificaciones, cartas, inventarios, relaciones, moderación y notificaciones. Cloudinary gestiona los assets gráficos.

La principal secuencia de negocio es:

```text
Usuario publica obra
        |
        v
Período Q2Q de 7 días
        |
        +--> comentarios continúan habilitados
        |
        v
Fin de período
        |
        v
Scheduler detecta obras elegibles
        |
        v
Conversión a carta
        |
        v
Carta disponible para colección
        |
        v
Paquetes
        |
        v
Colección del usuario
        |
        v
Autobattler local
```

