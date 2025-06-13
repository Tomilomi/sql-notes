
# Guía de Repaso: Actualización de Bases de Datos

## 📌 Operaciones DML (Data Manipulation Language)
Las sentencias DML permiten modificar el contenido de las tablas:

- **INSERT**: Inserta nuevas filas.
- **UPDATE**: Modifica valores existentes.
- **DELETE**: Elimina registros.

---

## 🔒 Integridad de Datos

### Posibles intentos de violar la integridad al **INSERTAR**
- Insertar una **PK duplicada**.
- Insertar una **PK con valor NULL**.
- Insertar un **atributo duplicado** definido como `UNIQUE`.
- Insertar un **atributo NULL** definido como `NOT NULL`.
- Insertar una **clave foránea (FK)** sin concordancia con una **clave primaria (PK)**.
- Insertar un atributo **fuera de su dominio** (tipo o valor inválido).

### Posibles intentos de violar la integridad al **ACTUALIZAR**
- Actualizar una **PK con valor duplicado**.
- Actualizar una **PK con valor NULL**.
- Actualizar un atributo con **valor duplicado** (definido como `UNIQUE`).
- Asignar `NULL` a un atributo definido como `NOT NULL`.
- Modificar una **FK sin concordancia con una PK**.
- Asignar valores **fuera de dominio** (tipo o valor).
- Cambiar el valor de una **PK que tiene hijos**.

### Posibles intentos de violar la integridad al **ELIMINAR**
- Eliminar un registro con **PK referenciada** por otras tablas (tiene hijos).

---

## 📋 Tipos de Restricciones

| Tipo de Restricción         | Descripción |
|----------------------------|-------------|
| `NOT NULL`                 | El valor de la columna es obligatorio. |
| `CHECK`                    | Restringe los valores que se pueden insertar. |
| `PRIMARY KEY`              | Garantiza unicidad y no nulos. |
| `FOREIGN KEY`              | Asegura la integridad referencial entre tablas. |

---

## 🔁 Reglas de Actualización y Eliminación en Claves Foráneas - Regla de compensacion

Se definen al declarar una `FOREIGN KEY`. Determinan el comportamiento ante cambios en la tabla padre.

### Reglas disponibles:

- **RESTRICT / NO ACTION (por defecto)**: Impide la acción si hay hijos.
- **CASCADE**: Propaga los cambios a las filas hijas.
- **SET NULL**: Asigna `NULL` en la FK hija.
- **SET DEFAULT**: Asigna el valor por defecto definido para la columna.

---

## ⚠️ Ciclos Referenciales

### Problemas frecuentes:
- En **inserción**: puede haber dependencia circular entre tablas.
- En **eliminación**: puede requerir múltiples pasos o desactivación de restricciones temporales.

### Consecuencias:
- Dificultan los procedimientos comerciales.
- Generan duplicación de lógica y esfuerzo en el mantenimiento de programas.
