# 🔍 Guía de Condiciones de Búsqueda en SQL (Extendida)

## 📌 Comparadores básicos

| Operador | Significado         | Ejemplo                |
|----------|---------------------|------------------------|
| =        | Igual a             | `WHERE edad = 30`      |
| <> / !=  | Distinto de         | `WHERE nombre <> 'Ana'`|
| >        | Mayor que           | `WHERE salario > 50000`|
| <        | Menor que           | `WHERE edad < 18`      |
| >=       | Mayor o igual que   | `WHERE edad >= 65`     |
| <=       | Menor o igual que   | `WHERE edad <= 21`     |

---

## 🧠 Operadores lógicos

| Operador | Significado                           | Ejemplo                                |
|----------|----------------------------------------|----------------------------------------|
| AND      | Todas las condiciones se cumplen      | `WHERE edad > 18 AND ciudad = 'Córdoba'` |
| OR       | Al menos una condición se cumple      | `WHERE ciudad = 'Córdoba' OR ciudad = 'Rosario'` |
| NOT      | Niega una condición                   | `WHERE NOT (estado = 'Inactivo')`      |

---

## 🎯 Operadores especiales

### `BETWEEN`
Filtra valores dentro de un rango:
```sql
WHERE salario BETWEEN 30000 AND 60000
```

### `IN`
Compara contra una lista de valores:
```sql
WHERE pais IN ('Argentina', 'Chile', 'Uruguay')
```

### `LIKE`
Búsqueda por patrones:
```sql
WHERE nombre LIKE 'J%'      -- Empieza con J
WHERE nombre LIKE '%ez'     -- Termina con ez
WHERE nombre LIKE '_ana'    -- Cuatro letras, termina en "ana"
```

### `IS NULL / IS NOT NULL`
Evalúa si un campo está vacío o no:
```sql
WHERE fecha_baja IS NULL
WHERE fecha_baja IS NOT NULL
```

---

## 🧪 Tests de Subconsulta

### 1. Test de igualdad (subconsulta escalar)
```sql
SELECT nombre FROM Empleados
WHERE departamento_id = (
  SELECT id FROM Departamentos WHERE nombre = 'Ventas'
)
```

### 2. Test de pertenencia con IN
```sql
SELECT nombre FROM Empleados
WHERE departamento_id IN (
  SELECT id FROM Departamentos WHERE region = 'Sur'
)
```

### 3. Test de existencia con EXISTS
```sql
SELECT nombre FROM Departamentos D
WHERE EXISTS (
  SELECT 1 FROM Empleados E
  WHERE E.departamento_id = D.id
)
```

### 4. Test cuantificado: ALL, ANY, SOME

**ALL:**
```sql
SELECT nombre FROM Empleados
WHERE salario > ALL (
  SELECT salario FROM Empleados WHERE departamento_id = 3
)
```

**ANY / SOME:**
```sql
SELECT nombre FROM Empleados
WHERE salario > ANY (
  SELECT salario FROM Empleados WHERE departamento_id = 3
)
```

---

## ✅ Resumen

| Tipo de Test | Uso principal |
|--------------|---------------|
| Igualdad (=) | Comparar con un único valor |
| IN | Comparar con varios valores |
| EXISTS | Verifica si existen registros relacionados |
| ALL / ANY / SOME | Comparaciones con múltiples valores |

---

## 🧮 Ejemplo combinado

```sql
SELECT nombre FROM Empleados
WHERE salario > 50000
  AND departamento_id IN (
    SELECT id FROM Departamentos WHERE region = 'Centro'
  )
  AND EXISTS (
    SELECT 1 FROM Proyectos
    WHERE Proyectos.empleado_id = Empleados.id
  )
```