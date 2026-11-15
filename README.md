# Base de datos — Eliko Bodega

## Inicializar

```bash
sqlite3 eliko.db < eliko_db.sql
```

El archivo `eliko_db.sql` contiene:
1. **Schema** — CREATE TABLE para las 4 entidades (categoria, producto, inventario, movimiento_stock)
2. **Seed** — INSERT con los 767 productos reales del inventario (CSV 14/10/2026)
3. **Queries de negocio** — 5 consultas SQL comentadas (alertas, resumen por categoría, stock muerto, top 10, historial por SKU)

## Notas de calidad del CSV original
- 1 producto sin SKU → omitido (sin PK no puede insertarse)
- 73 productos sin nombre → nombre suplido como `"Producto {SKU}"` (completar manualmente)
- 23 productos sin categoría → inferida del prefijo SKU (ACC→Accesorios celular, etc.)
- `stock_minimo` NULL en 763/767 productos → decisión consciente, se completa progresivamente
