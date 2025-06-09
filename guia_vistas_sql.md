## 📘 4.2.5. Actualización de Vistas (pág. 192)

En las operaciones de actualización realizadas sobre vistas, se presenta el siguiente problema:  
si se especifica una sentencia `INSERT`, `DELETE` o `UPDATE` sobre una vista,  
las actualizaciones correspondientes **deben traducirse a operaciones equivalentes** sobre las **tablas reales** que componen la vista.  
Esto **no siempre es posible**.

---

### ❌ Problemas con funciones de columna

Por ejemplo, si una vista tiene funciones de columna como `AVG` (que calcula el promedio de un conjunto de datos)  
y se intenta actualizar dicha vista, el **DBMS no permitirá la operación**.  
Esto se debe a que **no hay una manera clara de reflejar la actualización** en los valores originales usados para calcular el promedio.

> Cada fila de una **vista agrupada** representa un grupo de filas de la tabla real,  
> por lo que **no se puede trasladar directamente una actualización** desde la vista hacia las tablas subyacentes.

---

### ⚠️ Casos en los que una vista **no es actualizable**

Actualmente, las vistas **no pueden actualizarse** si cumplen con **alguna** de las siguientes condiciones:

- 🔸 Usan `DISTINCT` en la cláusula `SELECT` (filas no repetidas).
- 🔸 Utilizan `GROUP BY` o `HAVING` (agrupamiento de datos).
- 🔸 Incluyen funciones de columna como:
  - `AVG()`
  - `SUM()`
  - `MAX()`
  - `MIN()`, etc.
- 🔸 Se basan en otra vista **que no es actualizable**.
- 🔸 La cláusula `WHERE` incluye **subconsultas**.

---

### 📌 Nota adicional

También se presenta el mismo problema al intentar actualizar una vista con cláusula `ORDER BY`,  
ya que esta también implica un tipo de agrupamiento que impide reflejar adecuadamente la modificación en las tablas base.
