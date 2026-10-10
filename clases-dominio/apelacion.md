# Apelación (Appeal)

La solicitud de un Usuario sancionado para que se revise y, potencialmente, revierta la sanción aplicada por un Reporte ya resuelto.

- **Atributos conceptuales:** reporte referenciado, apelante, motivo predefinido, texto explicativo opcional, estado (`pending`/`approved`/`rejected`), quién la revisó, cuándo.
- **Responsabilidades:** representar el único canal (junto con la reapertura directa de un admin) para revertir una sanción; garantizar que quien revisa no sea quien resolvió el reporte original, salvo que sea la única persona de moderación disponible.
- **Relaciones:** referencia exactamente 1 Reporte (y sólo puede existir una Apelación por Reporte) · presentada por 1 Usuario · revisada por 0..1 Usuario (moderador/admin).
