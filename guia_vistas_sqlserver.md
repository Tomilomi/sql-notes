# 👁️‍🗨️ Guía Teórico-Práctica sobre Vistas en SQL Server

## 📌 ¿Qué es una vista?
Una **vista** es una **consulta guardada** en la base de datos que actúa como una **tabla virtual**. No almacena datos físicamente, sino que muestra resultados en tiempo real basados en otras tablas.

---

## 🎯 ¿Para qué se usan las vistas?

- Simplificar consultas complejas.
- Ocultar detalles de estructura de tablas.
- Restringir el acceso a ciertos datos (seguridad).
- Reutilizar lógica de consulta.

---

## 🔧 Sintaxis básica

```sql
CREATE VIEW nombre_vista AS
SELECT columnas
FROM tabla
WHERE condiciones;
```

✅ Ejemplo:

```sql
CREATE VIEW Vista_EmpleadosActivos AS
SELECT ID, nombre, puesto
FROM Empleados
WHERE estado = 'Activo';
```

---

## 📤 Usar una vista

Una vez creada, se consulta igual que una tabla:

```sql
SELECT * FROM Vista_EmpleadosActivos;
```

---

## 🛠️ Actualizar o eliminar una vista

- **Actualizar:**

```sql
ALTER VIEW Vista_EmpleadosActivos AS
SELECT ID, nombre
FROM Empleados
WHERE estado = 'Activo' AND puesto = 'Ingeniero';
```

- **Eliminar:**

```sql
DROP VIEW Vista_EmpleadosActivos;
```

---

## ⚠️ Limitaciones de las vistas

- No pueden tener `ORDER BY` (excepto con `TOP`).
- Algunas vistas **no son actualizables** (por ejemplo, si usan `GROUP BY`, `DISTINCT`, `UNION`, etc.).
- No almacenan datos, por lo que cada uso ejecuta la consulta original.

---

## 🔐 Vistas y seguridad

Podés crear vistas para dar acceso a ciertas columnas o filas, sin exponer toda la tabla:

```sql
CREATE VIEW Vista_Publica AS
SELECT nombre, ciudad FROM Clientes;
-- El usuario puede ver esta vista, pero no la tabla original.
```

---

## 🧪 Vista práctica avanzada

```sql
CREATE VIEW Vista_ResumenVentas AS
SELECT v.clienteID, c.nombre, SUM(v.monto) AS TotalGastado
FROM Ventas v
JOIN Clientes c ON v.clienteID = c.ID
GROUP BY v.clienteID, c.nombre;
```

Permite ver el total gastado por cliente, sin repetir toda la lógica cada vez.

---

## ✅ Conclusión

- Las vistas **simplifican**, **protegen** y **reutilizan** lógica de consulta.
- Se comportan como **tablas virtuales**.
- No deben confundirse con tablas reales: **no almacenan datos por sí solas**.

