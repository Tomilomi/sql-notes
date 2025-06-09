# 🧾 Guía básica y avanzada de transacciones en SQL

---

## ¿Qué es una transacción?

Una **transacción** es un conjunto de operaciones SQL que se ejecutan como una sola unidad lógica. Deben cumplir las propiedades ACID:

- **Atomicidad**: todo se hace o nada se hace.
- **Consistencia**: la base de datos pasa de un estado válido a otro válido.
- **Aislamiento**: las transacciones no interfieren entre sí.
- **Durabilidad**: una vez confirmada, la información queda guardada.

---

## Comandos principales para controlar transacciones

### 1. **BEGIN TRANSACTION**

- Marca el inicio de una transacción.

```sql
BEGIN TRANSACTION;
```

### 2. **SAVE TRANSACTION / SAVEPOINT**

- Define un punto de guardado para rollback parcial.

```sql
SAVE TRANSACTION punto1;  -- SQL Server
-- o
SAVEPOINT punto1;         -- Otros motores
```

### 3. **COMMIT TRANSACTION**

- Confirma todos los cambios.

```sql
COMMIT TRANSACTION;
```

### 4. **ROLLBACK TRANSACTION**

- Revierte todos los cambios desde el inicio o último commit.

```sql
ROLLBACK TRANSACTION;
```

### 5. **ROLLBACK TO SAVEPOINT**

- Revierte hasta un punto guardado específico.

```sql
ROLLBACK TRANSACTION punto1;  -- SQL Server
-- o
ROLLBACK TO SAVEPOINT punto1; -- Otros sistemas
```

---

## Cerramiento (Closure) en SQL

El cerramiento se refiere a que la transacción debe ser una unidad completa y autónoma, donde todas las operaciones dependientes se ejecutan juntas, y no quede ninguna operación "pendiente" o incongruente.

### ¿Cómo se logra el cerramiento en SQL?

1. **Iniciando la transacción** con `BEGIN TRANSACTION` se asegura que todas las operaciones que siguen son parte de un bloque indivisible.

2. **Aislamiento adecuado**: Usar un nivel de aislamiento que garantice que las operaciones dentro de la transacción no se vean afectadas por otras transacciones (por ejemplo, `REPEATABLE READ` o `SERIALIZABLE`).

3. **Control de integridad**: Todas las validaciones de integridad (claves foráneas, restricciones CHECK, etc.) deben pasar antes de hacer commit.

4. **Commit o rollback completo**: Al finalizar la transacción, se debe hacer un `COMMIT` para aplicar todos los cambios juntos, o un `ROLLBACK` para deshacer todo si hubo error.

### Ejemplo de cerramiento completo

```sql
BEGIN TRANSACTION;

-- Inserto cliente
INSERT INTO Clientes (dni, nombre) VALUES (12345678, 'Ana');

-- Inserto pedido relacionado
INSERT INTO Pedidos (id, cliente_dni, descripcion) VALUES (1, 12345678, 'Pedido 1');

-- Validación o lógica adicional (opcional)

-- Si todo está bien, confirmo
COMMIT TRANSACTION;

-- Si hubo error, se hace rollback (ejemplo en código)
-- ROLLBACK TRANSACTION;
```

En este ejemplo, ambas operaciones quedan "cerradas" dentro de la misma transacción: o se hacen ambas o ninguna.

---

## Problemas comunes sin buen manejo de transacciones

### **Lectura sucia (Dirty Read)**

Ocurre cuando una transacción lee datos que otra transacción ha modificado pero **aún no ha confirmado** (no ha hecho COMMIT).

**Ejemplo:**
```sql
-- Transacción A
BEGIN TRANSACTION;
UPDATE Productos SET precio = 100 WHERE id = 1; -- Precio era 50
-- NO hace COMMIT todavía

-- Transacción B (al mismo tiempo)
SELECT precio FROM Productos WHERE id = 1; -- Lee 100 (dato "sucio")

-- Transacción A decide cancelar
ROLLBACK TRANSACTION; -- El precio vuelve a 50

-- Transacción B usó el valor 100 que nunca fue real
```

**Problema:** La transacción B tomó decisiones basada en datos que nunca existieron oficialmente.

### **Lectura no repetible (Non-repeatable Read)**

Una transacción lee los mismos datos dos veces y obtiene **valores diferentes** porque otra transacción los modificó entre las dos lecturas.

**Ejemplo:**
```sql
-- Transacción A
BEGIN TRANSACTION;
SELECT saldo FROM Cuentas WHERE id = 123; -- Lee 1000

-- Transacción B (mientras A sigue activa)
BEGIN TRANSACTION;
UPDATE Cuentas SET saldo = 500 WHERE id = 123;
COMMIT TRANSACTION;

-- Transacción A lee otra vez
SELECT saldo FROM Cuentas WHERE id = 123; -- Ahora lee 500
COMMIT TRANSACTION;
```

**Problema:** La transacción A esperaba que el saldo fuera consistente durante toda su ejecución, pero cambió a mitad de camino.

### **Inserción fantasma (Phantom Read)**

Una transacción ejecuta la misma consulta dos veces y la segunda vez **aparecen filas nuevas** que no estaban antes.

**Ejemplo:**
```sql
-- Transacción A
BEGIN TRANSACTION;
SELECT COUNT(*) FROM Empleados WHERE departamento = 'Ventas'; -- Cuenta 10

-- Transacción B
BEGIN TRANSACTION;
INSERT INTO Empleados (nombre, departamento) VALUES ('Juan', 'Ventas');
COMMIT TRANSACTION;

-- Transacción A ejecuta la misma consulta
SELECT COUNT(*) FROM Empleados WHERE departamento = 'Ventas'; -- Ahora cuenta 11
COMMIT TRANSACTION;
```

**Problema:** Aparecieron registros "fantasma" que no existían en la primera consulta.

### **Pérdida de actualizaciones (Lost Update)**

Dos transacciones leen el mismo valor, lo modifican por separado, y una sobrescribe los cambios de la otra.

**Ejemplo:**
```sql
-- Ambas transacciones leen saldo = 1000

-- Transacción A
BEGIN TRANSACTION;
SELECT saldo FROM Cuentas WHERE id = 123; -- Lee 1000
-- Calcula: 1000 - 200 = 800
UPDATE Cuentas SET saldo = 800 WHERE id = 123;
COMMIT TRANSACTION;

-- Transacción B (casi al mismo tiempo)
BEGIN TRANSACTION;
SELECT saldo FROM Cuentas WHERE id = 123; -- También lee 1000 (valor original)
-- Calcula: 1000 + 300 = 1300
UPDATE Cuentas SET saldo = 1300 WHERE id = 123; -- Sobrescribe el 800 de A
COMMIT TRANSACTION;

-- Resultado: saldo = 1300, pero debería ser 1100 (1000 - 200 + 300)
```

**Problema:** Se perdió la actualización de la transacción A. El depósito de 300 sobrescribió completamente el retiro de 200.

### **Interbloqueo (Deadlock)**

Dos o más transacciones esperan recursos mutuamente, bloqueándose para siempre.

**Ejemplo:**
```sql
-- Transacción A
BEGIN TRANSACTION;
UPDATE Tabla1 SET campo = 'A' WHERE id = 1; -- Bloquea Tabla1
-- Ahora necesita Tabla2...
UPDATE Tabla2 SET campo = 'A' WHERE id = 1; -- Espera porque B la tiene bloqueada

-- Transacción B (al mismo tiempo)
BEGIN TRANSACTION;
UPDATE Tabla2 SET campo = 'B' WHERE id = 1; -- Bloquea Tabla2
-- Ahora necesita Tabla1...
UPDATE Tabla1 SET campo = 'B' WHERE id = 1; -- Espera porque A la tiene bloqueada

-- Resultado: A espera a B, B espera a A = DEADLOCK
```

**Solución:** El motor de base de datos detecta esto y aborta una de las transacciones automáticamente.

**Prevención:**
- Acceder siempre a las tablas en el mismo orden (primero Tabla1, después Tabla2)
- Mantener transacciones cortas
- Usar niveles de aislamiento apropiados

---

## Niveles de aislamiento

| Nivel de aislamiento | Evita lectura sucia | Evita lectura no repetible | Evita inserción fantasma |
|---------------------|---------------------|---------------------------|-------------------------|
| READ UNCOMMITTED    | No                  | No                        | No                      |
| READ COMMITTED      | Sí                  | No                        | No                      |
| REPEATABLE READ     | Sí                  | Sí                        | No                      |
| SNAPSHOT            | Sí                  | Sí                        | Sí                      |
| SERIALIZABLE        | Sí                  | Sí                        | Sí                      |

---

## Configuraciones avanzadas para transacciones

### 1. **SET DEADLOCK_PRIORITY {NORMAL | HIGH | LOW}**

- Controla la prioridad de la transacción cuando ocurre un deadlock (interbloqueo).
- SQL Server usa esta prioridad para decidir qué transacción abortar si hay bloqueo mutuo.

| Prioridad | Qué significa |
|-----------|---------------|
| NORMAL    | Prioridad por defecto |
| HIGH      | Esta transacción tiene mayor prioridad, se abortarán otras antes que esta |
| LOW       | Esta transacción será la primera en abortarse en caso de deadlock |

**Ejemplo:**

```sql
SET DEADLOCK_PRIORITY HIGH;
BEGIN TRANSACTION;
-- operaciones
COMMIT TRANSACTION;
```

### 2. **SET LOCK_TIMEOUT período**

- Establece el tiempo (en milisegundos) que una transacción esperará para obtener un bloqueo antes de devolver un error.
- Evita que la transacción quede bloqueada indefinidamente esperando un recurso.

**Ejemplo:**

```sql
SET LOCK_TIMEOUT 5000; -- Espera máximo 5 segundos
BEGIN TRANSACTION;
-- operaciones que necesitan bloqueo
COMMIT TRANSACTION;
```

Si pasados 5 segundos el bloqueo no se libera, la transacción recibe error y puede manejarse para reintentar o abortar.

### 3. **SET TRANSACTION ISOLATION LEVEL**

- Define el nivel de aislamiento para la transacción actual, que controla cómo y cuándo se ven los cambios hechos por otras transacciones concurrentes.
- Afecta la posibilidad de lecturas sucias, no repetibles y fantasmas.

| Nivel de aislamiento | Descripción breve |
|---------------------|-------------------|
| READ UNCOMMITTED    | Permite leer datos sin confirmar (lecturas sucias posibles) |
| READ COMMITTED      | Sólo lee datos confirmados (nivel por defecto en muchos DBMS) |
| REPEATABLE READ     | Bloquea filas leídas para evitar cambios hasta el fin de la transacción |
| SNAPSHOT            | Usa versiones para evitar bloqueos (implementación MVCC) |
| SERIALIZABLE        | Nivel más estricto, bloquea rangos para evitar inserciones fantasmas |

**Ejemplo:**

```sql
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
BEGIN TRANSACTION;
-- operaciones
COMMIT TRANSACTION;
```

### Cómo usar estas configuraciones juntas

```sql
SET DEADLOCK_PRIORITY LOW;
SET LOCK_TIMEOUT 10000;  -- 10 segundos de espera
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;

BEGIN TRANSACTION;

-- operaciones SQL aquí

COMMIT TRANSACTION;
```

Con esta configuración:

- La transacción esperará máximo 10 segundos por bloqueos.
- Tiene prioridad baja para ser abortada en deadlocks.
- Usa nivel de aislamiento que evita lecturas no repetibles.

### ¿Por qué es importante esto?

- **Controlar prioridades** ayuda a manejar mejor recursos en sistemas con muchas transacciones concurrentes.
- **Definir tiempo máximo** para esperas evita bloqueos largos que perjudican el rendimiento.
- **Ajustar nivel de aislamiento** equilibra entre integridad y concurrencia.

---

## Resumen comandos

| Comando | Función |
|---------|---------|
| BEGIN TRANSACTION | Inicia una transacción |
| SAVE TRANSACTION / SAVEPOINT | Marca un punto para rollback parcial |
| COMMIT TRANSACTION | Confirma todos los cambios |
| ROLLBACK TRANSACTION | Deshace todos los cambios desde el inicio o último commit |
| ROLLBACK TO SAVEPOINT | Deshace cambios hasta un punto guardado específico |
| SET DEADLOCK_PRIORITY | Define prioridad para abortar transacciones en deadlocks |
| SET LOCK_TIMEOUT | Establece tiempo máximo de espera para bloqueos |
| SET TRANSACTION ISOLATION LEVEL | Configura nivel de aislamiento de la transacción |