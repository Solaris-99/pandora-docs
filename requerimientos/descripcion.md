# 1. Descripción del proyecto

Pandora es una plataforma que integra publicación de arte digital, curaduría comunitaria, coleccionismo de cartas y un juego de cartas automatizado. Los usuarios pueden publicar obras, explorar contenido, comentar, valorar obras durante un período limitado mediante estrellas y emociones, y participar en un sistema donde la percepción de la comunidad contribuye a determinar los atributos de una carta asociada a la obra.

Una obra publicada puede convertirse opcionalmente en una carta. La conversión sólo se realiza cuando el período de calificación termina y la obra está marcada para conversión. El procesamiento será realizado por un proceso CRON que detecta las obras pendientes y genera las cartas correspondientes.

Las cartas obtenidas pasan a formar parte de la colección del usuario mediante aperturas de paquetes. Las cartas pueden utilizarse en un autobattler completamente local al cliente. La plataforma también incluye perfiles públicos, seguimiento de usuarios, favoritos, comentarios, reportes, moderación y notificaciones.

La idea original define el ecosistema como una combinación de exposición artística, evaluación comunitaria y juego/coleccionismo. Esta guía mantiene ese enfoque, pero ajusta las partes que fueron posteriormente descartadas o modificadas.

Fuente base: la propuesta define web, backend centralizado y aplicación móvil, junto con publicación de obras, Q2Q, conversión de cartas, colección y módulo de juego.

La batalla se considera una funcionalidad local del cliente: el backend sólo interviene para seleccionar una carta rival. No existe PvP, sincronización de partida ni persistencia de victorias o derrotas.
