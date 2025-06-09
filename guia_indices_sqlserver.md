# 📘 Guía de Índices en SQL Server

## 📌 ¿Qué es un índice?
Un **índice** es una estructura que permite acceder más rápido a los datos de una tabla, similar al índice de un libro.

---

## 🧠 Tipos de Índices

### 🔷 Índice Primario (Clustered Index)
- Ordena **físicamente** los datos de la tabla.
- Solo puede haber **uno por tabla**.
- Se crea automáticamente con la **clave primaria** (`PRIMARY KEY`).
- Ejemplo:
```sql
CREATE UNIQUE CLUSTERED INDEX empleadosID_ind ON empleados (ID);
```

---

### 🔷 Índice Secundario (NonClustered Index)
- **No cambia el orden físico** de los datos.
- Es una estructura separada con punteros a las filas.
- Se pueden crear **hasta 249 por tabla**.
- Ejemplo:
```sql
CREATE INDEX autor_ind ON autor (id);
```

---

### 🔷 Índice Compuesto
- Usa **2 o más columnas** en la clave del índice.
- Puede ser **clustered o nonclustered**.
- Soporta hasta **16 columnas** y un tamaño total de hasta **900 bytes**.
- El **orden de las columnas importa**.
- Ejemplo:
```sql
CREATE INDEX seccion_emp_ind ON seccionxemp (seccionID, ID);
```

---

## 🔄 Comparación Analógica

| Tipo Clásico     | SQL Server              | Descripción                                                                      |
|------------------|--------------------------|----------------------------------------------------------------------------------|
| Índice Primario  | Clustered Index          | Ordena físicamente los datos. Se crea con la clave primaria.                    |
| Índice Secundario| NonClustered Index       | No cambia el orden físico. Apunta a los datos reales.                           |
| Índice Compuesto | Clustered o NonClustered | Usa varias columnas. El orden en la definición del índice es importante.        |

---

## 📚 Analogía: Enciclopedia

- **Índice Primario**: como si las páginas estuvieran ordenadas alfabéticamente (orden físico).
- **Índice Secundario**: como una hoja adicional que apunta a páginas concretas (marcador).
- **Índice Compuesto**: como una estructura por categoría y subcategoría (Animal > Mamífero).

---

## 💡 Nota Final

> ✅ Un índice primario suele ser **clustered**.  
> ✅ Los índices secundarios suelen ser **nonclustered**.  
> ✅ Elegí bien qué columnas indexar para mejorar el rendimiento sin sobrecargar la base.

