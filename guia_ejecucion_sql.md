# 📋 Orden de ejecución en SQL

En SQL, aunque escribimos la consulta en un orden lógico (`SELECT ... FROM ... WHERE ...`), el motor de base de datos procesa las cláusulas en un orden diferente, que es clave para entender cómo funcionan las consultas y cómo se optimizan.

---

## Orden típico de ejecución de una consulta SQL

1. **FROM**  
   Se identifican y combinan las tablas involucradas. Aquí se aplican los `JOIN`.

2. **ON**  
   Se aplican las condiciones de los `JOIN` para filtrar las combinaciones.

3. **JOIN**  
   Se unen las tablas según la condición del `ON`.

4. **WHERE**  
   Se filtran las filas según las condiciones indicadas.

5. **GROUP BY**  
   Se agrupan las filas en grupos para funciones agregadas.

6. **HAVING**  
   Se filtran los grupos según la condición dada.

7. **SELECT**  
   Se seleccionan las columnas o expresiones a mostrar.

8. **DISTINCT**  
   Se eliminan filas duplicadas del resultado.

9. **ORDER BY**  
   Se ordenan las filas según las columnas indicadas.

10. **LIMIT / OFFSET**  
    Se limitan la cantidad de filas devueltas o se saltan las primeras filas.

---

## Subconsultas: ¿Cuándo y cuántas veces se ejecutan?

### Subconsulta sin referencia externa (subconsulta no correlacionada)

- Se ejecuta **una sola vez**.
- Su resultado es independiente de la fila actual de la consulta externa.
- El motor puede guardar ese resultado en memoria para reutilizarlo (materialización).
- Ejemplo:

  ```sql
  SELECT nombre
  FROM empleados
  WHERE salario > (SELECT AVG(salario) FROM empleados);
  ```

  La subconsulta `(SELECT AVG(salario) FROM empleados)` se calcula una sola vez y luego se compara con cada fila.

### Subconsulta con referencia externa (subconsulta correlacionada)

- Se ejecuta **una vez por cada fila** que procesa la consulta externa.
- Depende del valor de la fila externa, por lo que no puede precalcularse.
- Ejemplo:

  ```sql
  SELECT nombre
  FROM empleados e1
  WHERE salario > (SELECT AVG(salario) FROM empleados e2 WHERE e2.departamento = e1.departamento);
  ```

  Aquí la subconsulta usa `e1.departamento`, por lo que se evalúa para cada empleado.

---

## Ejemplo completo con subconsultas y explicación

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

### Explicación paso a paso

1. **FROM Clientes C, Facturas F**  
   Se combinan las tablas Clientes y Facturas en un producto cartesiano, pero el filtro del WHERE limita la combinación.

2. **WHERE C.id_cli = F.id_cli**  
   Filtra la combinación para obtener solo las facturas relacionadas con cada cliente.

3. **Subconsulta en IN (no correlacionada)**
   ```sql
   SELECT L.codigo
   FROM Localida L
   WHERE L.region = 24
   ```
   Esta subconsulta se ejecuta una sola vez y devuelve todos los códigos postales (codigo) de localidades de la región 24. Luego, se filtra para que sólo los clientes cuyo código postal esté en esa lista sean considerados.

4. **Subconsulta en NOT EXISTS (correlacionada)**
   ```sql
   SELECT *
   FROM Pedidos P
   WHERE P.Cli = C.id_cli
     AND P.totped > 5000
   ```
   Esta subconsulta se ejecuta una vez por cada cliente (C.id_cli) y verifica que no existan pedidos de ese cliente con un total mayor a 5000. Si existe alguno, ese cliente queda excluido.

5. **GROUP BY C.id_cli, C.Apellido**  
   Agrupa resultados por cliente para sumar el total de sus facturas.

6. **HAVING SUM(F.totfac) > 1000**  
   Se filtran grupos para mostrar sólo los clientes cuya suma de facturas supera los 1000.

---

## Resumen

| Paso | Descripción |
|------|-------------|
| 1. FROM | Identificación y combinación de tablas |
| 2. ON | Aplicación de condiciones de unión |
| 3. JOIN | Unión efectiva de tablas |
| 4. WHERE | Filtrado de filas |
| 5. GROUP BY | Agrupamiento de filas para agregados |
| 6. HAVING | Filtrado de grupos |
| 7. SELECT | Selección de columnas o expresiones |
| 8. DISTINCT | Eliminación de duplicados |
| 9. ORDER BY | Ordenamiento del resultado |
| 10. LIMIT / OFFSET | Limitación y desplazamiento de filas |
