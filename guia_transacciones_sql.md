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

Marca el inicio de una transacción.

```sql
BEGIN TRANSACTION;
```

### 2. **SAVE TRANSACTION / SAVEPOINT**

Define un punto de guardado para rollback parcial.

```sql
SAVE TRANSACTION punto1;  -- SQL Server
-- o
SAVEPOINT punto1;         -- Otros motores
```

### 3. **COMMIT TRANSACTION**

Confirma todos los cambios.

```sql
COMMIT TRANSACTION;
```

### 4. **ROLLBACK TRANSACTION**

Revierte todos los cambios desde el inicio o último commit.

```sql
ROLLBACK TRANSACTION;
```

### 5. **ROLLBACK TO SAVEPOINT**

Revierte hasta un punto guardado específico.

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

### ⚠️ Actualización perdida

Ocurre cuando dos transacciones modifican el mismo dato y una sobrescribe los cambios de la otra sin saberlo.

```sql
-- Ambas transacciones leen saldo = 1000

-- Transacción A
BEGIN TRANSACTION;
SELECT saldo FROM Cuenta WHERE id = 1; -- Lee 1000
UPDATE Cuenta SET saldo = 800 WHERE id = 1; -- Resta 200
COMMIT TRANSACTION;

-- Transacción B
BEGIN TRANSACTION;
SELECT saldo FROM Cuenta WHERE id = 1; -- También lee 1000
UPDATE Cuenta SET saldo = 1300 WHERE id = 1; -- Suma 300
COMMIT TRANSACTION;

-- Resultado final: saldo = 1300 (se perdió la resta de A)
```

**❗ Problema**: Se pierden cambios porque ambas transacciones trabajaron sobre la misma base sin coordinación.

### ⚠️ Datos no confirmados

Una transacción lee datos que otra modificó, pero aún no confirmó (no hizo COMMIT). Si luego se hace un ROLLBACK, los datos eran inválidos.

```sql
-- Transacción A
BEGIN TRANSACTION;
UPDATE Productos SET precio = 200 WHERE id = 5;

-- Transacción B
SELECT precio FROM Productos WHERE id = 5; -- Lee 200 (dato no confirmado)

-- Transacción A decide cancelar
ROLLBACK TRANSACTION;
```

**❗ Problema**: La transacción B basó decisiones en datos que nunca existieron oficialmente.

### ⚠️ Datos inconsistentes

Se leen dos veces los mismos datos dentro de una transacción, pero los valores cambian porque otra transacción los modificó entre lecturas.

```sql
-- Transacción A
BEGIN TRANSACTION;
SELECT stock FROM Inventario WHERE producto = 'X'; -- Lee 50

-- Transacción B
BEGIN TRANSACTION;
UPDATE Inventario SET stock = 0 WHERE producto = 'X';
COMMIT TRANSACTION;

-- Transacción A vuelve a consultar
SELECT stock FROM Inventario WHERE producto = 'X'; -- Lee 0
COMMIT TRANSACTION;
```

**❗ Problema**: La transacción A no obtiene una vista estable de los datos.

### ⚠️ Inserción fantasma

Una transacción ejecuta la misma consulta dos veces, pero entre ambas otro proceso insertó nuevos registros que afectan el resultado.

```sql
-- Transacción A
BEGIN TRANSACTION;
SELECT COUNT(*) FROM Clientes WHERE ciudad = 'Córdoba'; -- Cuenta 100

-- Transacción B
INSERT INTO Clientes (nombre, ciudad) VALUES ('Juan', 'Córdoba');
COMMIT TRANSACTION;

-- Transacción A vuelve a consultar
SELECT COUNT(*) FROM Clientes WHERE ciudad = 'Córdoba'; -- Ahora cuenta 101
COMMIT TRANSACTION;
```

**❗ Problema**: El conjunto de resultados cambió durante la transacción sin que A lo notara.

---

## Niveles de aislamiento

| Nivel de aislamiento | Evita datos no confirmados | Evita datos inconsistentes | Evita inserción fantasma |
|---------------------|---------------------------|---------------------------|-------------------------|
| READ UNCOMMITTED    | ❌                        | ❌                        | ❌                      |
| READ COMMITTED      | ✅                        | ❌                        | ❌                      |
| REPEATABLE READ     | ✅                        | ✅                        | ❌                      |
| SNAPSHOT            | ✅                        | ✅                        | ✅                      |
| SERIALIZABLE        | ✅                        | ✅                        | ✅                      |

---

## Configuraciones avanzadas para transacciones

### 1. SET DEADLOCK_PRIORITY {NORMAL | HIGH | LOW}

Controla la prioridad de la transacción cuando ocurre un deadlock (interbloqueo). SQL Server usa esta prioridad para decidir qué transacción abortar si hay bloqueo mutuo.

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

### 2. SET LOCK_TIMEOUT período

Establece el tiempo (en milisegundos) que una transacción esperará para obtener un bloqueo antes de devolver un error. Evita que la transacción quede bloqueada indefinidamente esperando un recurso.

**Ejemplo:**

```sql
SET LOCK_TIMEOUT 5000; -- Espera máximo 5 segundos
BEGIN TRANSACTION;
-- operaciones que necesitan bloqueo
COMMIT TRANSACTION;
```

### 3. SET TRANSACTION ISOLATION LEVEL

Define el nivel de aislamiento para la transacción actual, que controla cómo y cuándo se ven los cambios hechos por otras transacciones concurrentes. Afecta la posibilidad de lecturas sucias, no repetibles y fantasmas.

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
| `BEGIN TRANSACTION` | Inicia una transacción |
| `SAVE TRANSACTION` / `SAVEPOINT` | Marca un punto para rollback parcial |
| `COMMIT TRANSACTION` | Confirma todos los cambios |
| `ROLLBACK TRANSACTION` | Deshace todos los cambios desde el inicio o último commit |
| `ROLLBACK TO SAVEPOINT` | Deshace cambios hasta un punto guardado específico |
| `SET DEADLOCK_PRIORITY` | Define prioridad para abortar transacciones en deadlocks |
| `SET LOCK_TIMEOUT` | Establece tiempo máximo de espera para bloqueos |
| `SET TRANSACTION ISOLATION LEVEL` | Configura nivel de aislamiento de la transacción |
