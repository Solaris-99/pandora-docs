# Validaciones

## Validación general

Toda entrada debe:

1. validarse en el backend aunque ya se valide en frontend;
2. normalizarse cuando corresponda;
3. limitar longitud y tamaño;
4. rechazar tipos inesperados;
5. **sanitizar contenido apropiadamente** — implementado: todo campo de texto libre (título/descripción de obra, tags, comentarios, bio, `comment`/`detail` de reportes y apelaciones) remueve marcado HTML antes de persistir, defensa contra XSS almacenado independiente del cliente que lo renderice (detalle en [convenciones de API](/api-conventions.md) §4.2);
6. evitar concatenación directa en consultas SQL — garantizado en todo el proyecto vía TypeORM (queries parametrizadas / query builder, sin interpolación directa de strings).

## Usuario

```text
username: 1–30 caracteres, sólo letras/números/guión bajo, único
email: formato válido, máximo 50 según RNF adoptado, único
password: política de complejidad definida por implementación (mínimo 6, máximo 50)
```

La contraseña debe almacenarse siempre como hash.

## Obras

```text
title: requerido, <= 50
description: opcional, <= 300
tags: lista acotada y normalizada
image: <= 25 MB, PNG/JPG/JPEG/WEBP
```

## Comentarios

```text
comment: requerido después de trim, <= 300
```

Deben aplicarse medidas anti-spam y rate limiting (ver [rate limiting](guards.md#rate-limiting)).

## Ratings

```text
stars: integer 1..5
emotion: valor perteneciente al enum
```

Además debe verificarse que la obra esté dentro de su período activo.

## Reportes

```text
reason: enum
comment: <= 300
```

No aceptar resolución enviada desde un cliente normal. Las resoluciones son acciones administrativas.

## DTOs

Los DTOs representan contratos explícitos de entrada. Catálogo (no exhaustivo) por módulo:

```text
RegisterDto, LoginDto, VerifyEmailDto, ResendVerificationDto,
ForgotPasswordDto, ResetPasswordDto, RefreshTokenDto, GoogleLoginDto
UpdateProfileDto
CreateArtworkDto, UpdateArtworkDto, AdminUploadCardDto, QueryArtworksDto
CreateCommentDto
RateArtworkDto
QueryCardsDto
QueryTagsDto
CreateReportDto, ResolveReportDto, ReopenReportDto
CreateAppealDto, ResolveAppealDto, QueryAppealsDto
```

No se reutilizan automáticamente entidades TypeORM como DTOs de entrada. Esto ayuda a:

- controlar qué campos puede modificar el cliente;
- aplicar validaciones;
- evitar mass assignment;
- separar modelo de persistencia y contrato de API.
