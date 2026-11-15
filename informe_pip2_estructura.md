# Informe PIP2 — Eliko Bodega
## Sistema de Gestión de Inventario
**ICN-292 · Sistemas de Información para la Gestión — UTFSM 2026-2**

---

## Portada
- Nombre del proyecto: Sistema de Gestión de Inventario — Eliko Bodega
- Integrantes del grupo
- Nombre del ayudante / sección
- Fecha de entrega
- Logo UTFSM

---

## 1. Descripción de la organización

### 1.1 Presentación de la empresa
Eliko Bodega es una empresa del rubro de accesorios electrónicos ubicada en Santiago, Chile. Opera principalmente a través de comercio electrónico, distribuyendo accesorios para celulares, cables y cargadores, audífonos, controles remotos y otros productos de electrónica de consumo.

### 1.2 Contexto del diagnóstico
La empresa fue visitada el 14 de octubre de 2026 con el objetivo de diagnosticar el estado de sus sistemas de información, específicamente el proceso de gestión de inventario en bodega.

### 1.3 Alcance del sistema
El sistema cubre exclusivamente el proceso de **gestión de inventario en bodega**. No incluye integración con la tienda online ni módulo de ventas, por decisión explícita de los dueños.

---

## 2. Diagnóstico del proceso actual (AS-IS)

### 2.1 Descripción del proceso
Describir cómo gestiona Eliko su inventario actualmente:
- Uso de planilla Excel / Google Sheets manual
- Sin sistema de alertas de stock mínimo
- Sin trazabilidad de movimientos (entradas/salidas)
- Etiquetas de código de barras generadas en HTML artesanal
- Categorías inconsistentes (23 productos sin categoría en el CSV)
- 73 productos sin nombre registrado

### 2.2 Diagrama BPMN — Proceso AS-IS
> Insertar diagrama BPMN del proceso de inventario actual.

### 2.3 Problemas identificados

| Problema | Impacto |
|---|---|
| Sin control de stock mínimo | No hay alertas de reposición; se detecta quiebre de stock solo cuando ya ocurrió |
| Sin historial de movimientos | Imposible auditar entradas/salidas ni calcular rotación |
| Datos incompletos en el catálogo | 73 productos sin nombre, 23 sin categoría dificultan búsquedas |
| Proceso manual y no centralizado | Alto riesgo de error humano y datos desactualizados |
| Sin indicadores de gestión (KPIs) | La gerencia no tiene visibilidad en tiempo real del estado del inventario |

---

## 3. Propuesta de mejora (TO-BE)

### 3.1 Solución propuesta
Sistema de gestión de inventario basado en base de datos relacional (SQLite) con dashboard de KPIs accesible en localhost, que reemplaza el manejo manual en planillas.

### 3.2 Diagrama BPMN — Proceso TO-BE
> Insertar diagrama BPMN del proceso mejorado con el sistema.

### 3.3 Beneficios esperados
- Alertas automáticas de reposición cuando el stock cae bajo el mínimo
- Trazabilidad completa de movimientos (quién, cuándo, cuánto, por qué)
- Dashboard de KPIs en tiempo real
- Búsqueda de productos por SKU o categoría
- Base para escalar hacia integración con tienda online

---

## 4. Modelo de datos

### 4.1 Diagrama Entidad-Relación (3FN)
> Insertar diagrama ER.

### 4.2 Descripción de entidades

| Entidad | PK | Descripción |
|---|---|---|
| `categoria` | `id_categoria` | 17 categorías de productos |
| `producto` | `sku` | 767 productos; SKU es identificador natural único |
| `inventario` | `id_registro` | Estado de stock; relación 1:1 con producto |
| `movimiento_stock` | `id_movimiento` | Historial de ENTRADA / SALIDA / AJUSTE |

### 4.3 Decisiones de diseño relevantes
- `stock_minimo` es nullable: solo 4 de 767 productos tienen mínimo definido; el campo se irá completando progresivamente.
- `ubicacion` eliminada: dato no disponible y bodega de tamaño reducido; se puede agregar sin romper el esquema.
- SKU como PK natural: estable, significativo y único en el negocio.

### 4.4 Normalización (3FN)
Verificar que el modelo cumple 1FN, 2FN y 3FN:
- **1FN**: todos los atributos son atómicos. ✓
- **2FN**: no hay dependencias parciales (todas las PK son simples). ✓
- **3FN**: no hay dependencias transitivas. `nombre_categoria` depende de `id_categoria`, no de `sku`. ✓

---

## 5. Base de datos y SQL

### 5.1 Schema SQL
> Incluir el DDL (CREATE TABLE) de las 4 tablas con sus constraints y foreign keys.
> (Ver archivo `db/eliko_db.sql`)

### 5.2 Queries de negocio
Presentar al menos 3 queries con su propósito:

**Q1 — Alertas de reposición:**
```sql
SELECT p.sku, p.nombre_producto, c.nombre_categoria,
       i.stock_actual, i.stock_minimo,
       (i.stock_minimo - i.stock_actual) AS unidades_a_reponer
FROM inventario i
JOIN producto  p ON p.sku = i.sku
JOIN categoria c ON c.id_categoria = p.id_categoria
WHERE i.stock_minimo IS NOT NULL
  AND i.stock_actual < i.stock_minimo
ORDER BY unidades_a_reponer DESC;
```

**Q2 — Resumen por categoría (KPI principal):**
```sql
SELECT c.nombre_categoria,
       COUNT(p.sku) AS total_productos,
       SUM(i.stock_actual) AS total_unidades
FROM categoria c
JOIN producto  p ON p.id_categoria = c.id_categoria
JOIN inventario i ON i.sku = p.sku
GROUP BY c.nombre_categoria
ORDER BY total_unidades DESC;
```

**Q3 — Detección de stock muerto:**
```sql
SELECT p.sku, p.nombre_producto, i.stock_actual
FROM producto p
JOIN inventario i ON i.sku = p.sku
WHERE p.sku NOT IN (
    SELECT DISTINCT sku FROM movimiento_stock
    WHERE fecha_movimiento >= datetime('now', '-28 days')
)
  AND i.stock_actual > 0
ORDER BY i.stock_actual DESC;
```

---

## 6. Sistema desarrollado

### 6.1 Arquitectura
- **Lenguaje**: Python 3.11+
- **Base de datos**: SQLite (portable, sin servidor)
- **Dashboard**: Streamlit + Pandas + Plotly
- **Acceso**: localhost:8501

### 6.2 Pantallas del sistema
> Insertar capturas de pantalla del dashboard con:
> - Tiles KPI (resumen general)
> - Gráfico de stock por categoría
> - Tabla de alertas de reposición
> - Buscador por SKU

### 6.3 Instrucciones de instalación
```bash
# 1. Crear la base de datos
sqlite3 db/eliko.db < db/eliko_db.sql

# 2. Instalar dependencias
pip install -r app/requirements.txt

# 3. Ejecutar el dashboard
streamlit run app/app.py
```

---

## 7. Conclusiones

### 7.1 Resultados obtenidos
- Se digitalizó el inventario completo de 767 productos en 17 categorías
- Se construyó un modelo de datos en 3FN que permite escalar el sistema
- Se implementó un dashboard de KPIs operacional en tiempo real
- Se estableció la base para trazabilidad de movimientos

### 7.2 Limitaciones y trabajo futuro
- `stock_minimo` requiere ser completado por el equipo de Eliko para activar las alertas de reposición en la mayoría de los productos
- El módulo de movimientos se alimenta con el uso del sistema; no hay historial previo
- Integración con tienda online: fuera del alcance actual, posible en una segunda etapa

### 7.3 Reflexión sobre el proceso
> Describir brevemente qué aprendieron del diagnóstico, del diseño del modelo y de la implementación. ¿Qué harían distinto?

---

## 8. Repositorio GitHub

> Incluir URL del repositorio con el código fuente, el SQL y los diagramas.

---

## Anexos

### A. Diccionario de datos
Tabla con todos los atributos, tipos, constraints y descripción.

### B. Diagrama BPMN completo

### C. Capturas adicionales del sistema

### D. Acta de visita a la empresa (14/10/2026)
