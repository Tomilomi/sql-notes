# 📝 Preguntas de Práctica SQL - Teoría

## 🔢 Preguntas de Múltiple Opción

### **Pregunta 1: Orden de Ejecución**
¿Cuál es el orden correcto de ejecución de las siguientes cláusulas en SQL?

A) SELECT → FROM → WHERE → GROUP BY → HAVING → ORDER BY  
B) FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY  
C) WHERE → FROM → SELECT → GROUP BY → HAVING → ORDER BY  
D) FROM → SELECT → WHERE → GROUP BY → HAVING → ORDER BY

**Respuesta:** B

---

### **Pregunta 2: Subconsultas Correlacionadas**
¿Cuántas veces se ejecuta una subconsulta correlacionada?

A) Una sola vez, al inicio de la consulta  
B) Una vez por cada fila de la tabla principal  
C) Una vez por cada fila del resultado final  
D) Depende del optimizador de la base de datos

**Respuesta:** B

---

### **Pregunta 3: Cláusula HAVING**
¿En qué momento del orden de ejecución se procesa la cláusula HAVING?

A) Antes de GROUP BY  
B) Después de SELECT  
C) Entre GROUP BY y SELECT  
D) Al final, después de ORDER BY

**Respuesta:** C

---

### **Pregunta 4: Subconsultas no Correlacionadas**
Una subconsulta que usa `IN` sin referencias a la consulta externa:

A) Se ejecuta una vez por cada fila  
B) Se ejecuta una sola vez y se materializa  
C) No se puede optimizar  
D) Es más lenta que una correlacionada

**Respuesta:** B

---

### **Pregunta 5: JOIN vs WHERE**
En el orden de ejecución, ¿qué se procesa primero?

A) WHERE siempre se procesa antes que JOIN  
B) JOIN siempre se procesa antes que WHERE  
C) FROM → ON → JOIN → WHERE  
D) Depende del tipo de JOIN

**Respuesta:** C

---

### **Pregunta 6: DISTINCT**
¿Cuándo se aplica DISTINCT en el orden de ejecución?

A) Inmediatamente después de SELECT  
B) Antes de ORDER BY pero después de SELECT  
C) Al final, después de LIMIT  
D) Antes de GROUP BY

**Respuesta:** B

---

### **Pregunta 7: Subconsulta EXISTS**
En la siguiente consulta, ¿cuántas veces se ejecuta la subconsulta EXISTS?
```sql
SELECT * FROM Clientes C 
WHERE EXISTS (SELECT 1 FROM Pedidos P WHERE P.cliente_id = C.id)
```

A) Una sola vez  
B) Una vez por cada cliente  
C) Una vez por cada pedido  
D) Ninguna vez si no hay datos

**Respuesta:** B

---

### **Pregunta 8: Optimización**
¿Cuál de estas subconsultas es más eficiente generalmente?

A) `WHERE id IN (SELECT id FROM tabla WHERE condicion)`  
B) `WHERE EXISTS (SELECT 1 FROM tabla WHERE tabla.id = t.id AND condicion)`  
C) Ambas son equivalentes  
D) Depende de la cantidad de datos

**Respuesta:** D

---

## ✏️ Completar Comandos SQL

### **Ejercicio 1: Orden de Ejecución**
Completa los espacios para mostrar el orden correcto:

```sql
-- 1. _______ tabla1 t1
-- 2. _______ t1.id = t2.id  
-- 3. _______ tabla2 t2
-- 4. _______ t1.activo = 1
-- 5. _______ BY t1.categoria
-- 6. _______ COUNT(*) > 5
-- 7. _______ t1.nombre, COUNT(*)
-- 8. _______ BY t1.nombre
```

**Respuesta:**
```sql
-- 1. FROM tabla1 t1
-- 2. ON t1.id = t2.id  
-- 3. JOIN tabla2 t2
-- 4. WHERE t1.activo = 1
-- 5. GROUP BY t1.categoria
-- 6. HAVING COUNT(*) > 5
-- 7. SELECT t1.nombre, COUNT(*)
-- 8. ORDER BY t1.nombre
```

---

### **Ejercicio 2: Subconsulta No Correlacionada**
Completa la consulta para obtener empleados con salario mayor al promedio:

```sql
SELECT nombre, salario
FROM empleados
WHERE salario > (
    SELECT _______(salario) 
    FROM _______
);
```

**Respuesta:**
```sql
SELECT nombre, salario
FROM empleados
WHERE salario > (
    SELECT AVG(salario) 
    FROM empleados
);
```

---

### **Ejercicio 3: Subconsulta Correlacionada**
Completa para obtener empleados con salario mayor al promedio de su departamento:

```sql
SELECT e1.nombre, e1.salario, e1.departamento
FROM empleados e1
WHERE e1.salario > (
    SELECT AVG(e2.salario)
    FROM empleados e2
    WHERE e2.departamento = _______._______
);
```

**Respuesta:**
```sql
SELECT e1.nombre, e1.salario, e1.departamento
FROM empleados e1
WHERE e1.salario > (
    SELECT AVG(e2.salario)
    FROM empleados e2
    WHERE e2.departamento = e1.departamento
);
```

---

### **Ejercicio 4: NOT EXISTS**
Completa para obtener clientes sin pedidos:

```sql
SELECT c.nombre
FROM clientes c
WHERE _______ _______ (
    SELECT *
    FROM pedidos p
    WHERE p.cliente_id = c._______
);
```

**Respuesta:**
```sql
SELECT c.nombre
FROM clientes c
WHERE NOT EXISTS (
    SELECT *
    FROM pedidos p
    WHERE p.cliente_id = c.id
);
```

---

### **Ejercicio 5: Consulta Compleja**
Completa la consulta basada en el ejemplo del documento:

```sql
SELECT C.id_cli, C.Apellido, _______(F.totfac)
FROM Clientes C, Facturas F
WHERE C.id_cli = F.id_cli
  AND C.codpos _______ (
    SELECT L.codigo
    FROM Localida L
    WHERE L.region = 24
  )
  AND _______ _______ (
    SELECT *
    FROM Pedidos P
    WHERE P.Cli = C.id_cli
      AND P.totped > 5000
  )
_______ _______ C.id_cli, C.Apellido
_______ SUM(F.totfac) > 1000;
```

**Respuesta:**
```sql
SELECT C.id_cli, C.Apellido, SUM(F.totfac)
FROM Clientes C, Facturas F
WHERE C.id_cli = F.id_cli
  AND C.codpos IN (
    SELECT L.codigo
    FROM Localida L
    WHERE L.region = 24
  )
  AND NOT EXISTS (
    SELECT *
    FROM Pedidos P
    WHERE P.Cli = C.id_cli
      AND P.totped > 5000
  )
GROUP BY C.id_cli, C.Apellido
HAVING SUM(F.totfac) > 1000;
```

---

## 🧠 Preguntas de Análisis

### **Pregunta A1**
Explica por qué esta consulta puede ser ineficiente:
```sql
SELECT * FROM empleados e1
WHERE salario = (SELECT MAX(salario) FROM empleados e2 WHERE e2.dept = e1.dept)
```

**Respuesta:** Es una subconsulta correlacionada que se ejecuta una vez por cada empleado, calculando el MAX para cada departamento repetidamente. Sería más eficiente usar una ventana de función o una subconsulta no correlacionada con GROUP BY.

---

### **Pregunta A2**
¿Qué diferencia hay entre estas dos consultas en términos de ejecución?

**Consulta 1:**
```sql
SELECT * FROM productos WHERE precio > (SELECT AVG(precio) FROM productos)
```

**Consulta 2:**
```sql
SELECT * FROM productos p1 WHERE precio > (SELECT AVG(precio) FROM productos p2 WHERE p2.categoria = p1.categoria)
```

**Respuesta:** 
- **Consulta 1:** Subconsulta no correlacionada, se ejecuta UNA vez
- **Consulta 2:** Subconsulta correlacionada, se ejecuta UNA vez por CADA producto

---

## 📊 Casos Prácticos

### **Caso 1: Optimización**
Tienes una tabla con 1 millón de empleados y 100 departamentos. ¿Cuál consulta será más rápida?

A) `SELECT * FROM emp e WHERE sal > (SELECT AVG(sal) FROM emp e2 WHERE e2.dept = e.dept)`
B) `SELECT e.* FROM emp e JOIN (SELECT dept, AVG(sal) as avg_sal FROM emp GROUP BY dept) d ON e.dept = d.dept WHERE e.sal > d.avg_sal`

**Respuesta:** B es más eficiente porque calcula el promedio una vez por departamento, mientras que A lo calcula para cada empleado.

---

### **Caso 2: Orden de Filtros**
En esta consulta, ¿en qué orden se aplican los filtros?
```sql
SELECT dept, COUNT(*), AVG(salario)
FROM empleados
WHERE activo = 1 AND fecha_ingreso > '2020-01-01'
GROUP BY dept
HAVING COUNT(*) > 10
ORDER BY AVG(salario) DESC
LIMIT 5
```

**Respuesta:**
1. WHERE (activo = 1 AND fecha_ingreso > '2020-01-01')
2. GROUP BY dept
3. HAVING COUNT(*) > 10
4. SELECT dept, COUNT(*), AVG(salario)
5. ORDER BY AVG(salario) DESC
6. LIMIT 5
