# Convención general

Cada módulo de NestJS agrupa tres tipos de clase de diseño:

- **Entidad** (`*.entity.ts`): clase TypeORM que mapea una tabla; define atributos tipados y relaciones (`@OneToMany`, `@ManyToOne`, etc.).
- **Servicio** (`*.service.ts`): clase inyectable que concentra la lógica de dominio y orquesta repositorios; es la única capa que accede a los repositorios directamente. Expone las operaciones que el/los controlador(es) del módulo invocan.
- **DTO** (`dto/*.dto.ts`): clases de contrato — de entrada (validadas con `class-validator`) o de salida (serializadas con `class-transformer`/`@Expose()`). Nunca se reutiliza una entidad como DTO de entrada.

Controladores y guards concretos (qué expone cada endpoint) viven en [diseño de componentes](/diseno-componentes/README.md); acá el foco es la clase y sus operaciones.
