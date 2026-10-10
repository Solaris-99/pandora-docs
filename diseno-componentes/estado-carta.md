# Estado de una carta en el cliente

La API de colección permite distinguir como mínimo:

```text
owned = true/false
copies = integer
stats = objeto completo | null
```

Esto permite que el frontend renderice:

- carta completa si es poseída;
- imagen/obra visible con estadísticas desconocidas (`stats: null`) si no es poseída.

La ocultación es una regla de presentación que además protege el backend: la API no envía información privada de forma innecesaria cuando el usuario no posee la carta (ver [Carta](/clases-dominio/carta.md) y [Colección](/clases-dominio/coleccion.md) en las clases de dominio para la regla de negocio, y [clases de diseño, módulo `cards`](/clases-diseno/cards-collections.md) para `toCardView`).
