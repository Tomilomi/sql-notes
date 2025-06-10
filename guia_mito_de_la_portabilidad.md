# ⚠️ Portabilidad en SQL

La existencia de estándares de SQL generó **aseveraciones exageradas** acerca de la portabilidad de SQL, de que podía ser utilizado en cualquier sistema de manejo de base de datos basado en SQL.  
Las diferencias existentes entre los dialectos SQL de cada vendedor son **suficientemente significativas** para que una aplicación tenga que ser cambiada al tener que emigrar de una base de datos a otra.

---

## 🔍 Las diferencias entre los diferentes dialectos de SQL incluyen:

- **Códigos de error.**  
  Cada implementación comercial tiene sus propios códigos de error.

- **Tipos de datos.**  
  Los tipos de datos difieren entre un fabricante y otro, por ejemplo, las cadenas de caracteres de longitud variable, los datos monetarios y los formatos de fecha y hora.

- **Tablas del sistema.**  
  Estas tablas proporcionan información de la estructura de la propia base de datos. Y cada vendedor tiene su propia estructura.

- **Interfaz de programa.**  
  Cada producto utiliza su propia técnica de interfaz con el programa de aplicación.

- **Diferencias semánticas.**  
  Debido a que el estándar SQL especifica ciertos detalles como "definidos por el fabricante", es posible encontrar una misma consulta de dos implementaciones de SQL diferentes que arrojen resultados distintos. Estas diferencias ocurren en el manejo de valores NULL y la eliminación de filas duplicadas.

- **Secuencias de ordenación.**  
  Los resultados de una consulta serán diferentes si se ejecutan en un computador personal (con caracteres ASCII) o en un maxicomputador (con caracteres EBCDIC).
