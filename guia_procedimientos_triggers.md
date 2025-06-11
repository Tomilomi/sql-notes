
# Guía de Procedimientos Almacenados y Disparadores en SQL

## 🧩 Procedimientos Almacenados (Stored Procedures)

### ¿Qué son?
Son bloques de código SQL precompilados que se almacenan en la base de datos y pueden ser ejecutados posteriormente con distintos parámetros. Se utilizan para encapsular lógica de negocio, automatizar tareas y mejorar la seguridad y el mantenimiento del sistema.

### Ventajas
- Modularidad y reutilización del código.
- Mejora del rendimiento al estar precompilados.
- Mayor control sobre la seguridad (se pueden otorgar permisos solo para ejecutar el procedimiento, sin acceso directo a las tablas).

### Sintaxis general (SQL Server)
```sql
CREATE PROCEDURE NombreProcedimiento
    @Parametro1 TipoDato,
    @Parametro2 TipoDato
AS
BEGIN
    -- Instrucciones SQL
END
```

### Ejemplo
```sql
CREATE PROCEDURE AgregarPais 
    @Cod VARCHAR(3), 
    @Nom VARCHAR(50)
AS
BEGIN
    INSERT INTO Paises (Codigo, Nombre)
    VALUES (@Cod, @Nom)
END
```

### Ejecución
```sql
EXEC AgregarPais 'AR', 'Argentina';
```

---

## 🔔 Disparadores (Triggers)

### ¿Qué son?
Son objetos de base de datos que se ejecutan automáticamente como respuesta a eventos DML (`INSERT`, `UPDATE`, `DELETE`) sobre una tabla. Su propósito es aplicar reglas de integridad adicional, auditoría o mantenimiento automático de datos relacionados.

### Ventajas
- Garantizan que ciertas acciones se realicen automáticamente.
- Refuerzan la integridad y trazabilidad de los datos.

### Consideraciones
- Aumentan la complejidad de la base de datos.
- Si no están bien documentados, pueden ser difíciles de mantener.

### Sintaxis general (SQL Server)
```sql
CREATE TRIGGER NombreTrigger
ON NombreTabla
AFTER INSERT | UPDATE | DELETE
AS
BEGIN
    -- Instrucciones a ejecutar
END
```

### Ejemplo
```sql
CREATE TRIGGER t_AltaEmpleados
ON Empleados
AFTER INSERT
AS
BEGIN
    INSERT INTO Auditoria (Tabla, Operacion, Usuario, Detalle)
    SELECT 
        'Empleados',
        'Alta',
        SESSION_USER,
        'Alta del Empleado ' + UPPER(Apellido) + ', ' + Nombre
    FROM inserted;
END
```

> RECORDAR QUE AFTER Y FOR FUNCIONAN PERO SE RECOMIENDA AFTER.

> En SQL Server, las pseudotablas `inserted` y `deleted` permiten acceder a los datos involucrados en la operación.

---

## 🔗 Reglas en Claves Foráneas (FOREIGN KEY)

### Definición
Al definir una clave foránea se puede especificar cómo debe comportarse ante operaciones de actualización o eliminación en la tabla referenciada.

### Sintaxis
```sql
FOREIGN KEY (ColumnaHija)
REFERENCES TablaPadre(ColumnaPadre)
    ON UPDATE [NO ACTION | CASCADE | SET NULL | SET DEFAULT]
    ON DELETE [NO ACTION | CASCADE | SET NULL | SET DEFAULT]
```

### Significado de las Reglas
- **NO ACTION / RESTRICT**: Impide la operación si hay referencias activas (comportamiento por defecto).
- **CASCADE**: Propaga el cambio o eliminación a las filas hijas.
- **SET NULL**: Asigna `NULL` en la columna hija si se elimina o modifica la fila padre.
- **SET DEFAULT**: Asigna el valor por defecto definido para la columna hija.
