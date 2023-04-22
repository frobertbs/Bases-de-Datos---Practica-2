## Bases-de-Datos---Practica-2

### Apartado 1 - Optimizacion de Consultas
IN
>
```sql
SELECT DISTINCT nombre_producto FROM productos WHERE productoID IN (SELECT productoID FROM detallePEDIDOS WHERE cantidad > 5)
```
EXISTS
>
```sql
SELECT DISTINCT nombre_producto FROM productos WHERE EXISTS(SELECT * FROM detallepedidos WHERE productos.productoID = detallepedidos.productoID AND cantidad > 5);
```

INNER JOIN
>
```sql
SELECT DISTINCT nombre_producto FROM productos INNER JOIN detallepedidos ON productos.productoID = detallepedidos.productoID WHERE cantidad > 5;
```
### Tiempos y Costes sin PKs y FKs
Sin las claves primarias y foráneas la consulta con el JOIN es la que menos tarda. En cuanto a costes, INNER JOIN y JOIN son las que más costes tienen

### Tiempos y Costes con PKs y sin FKs
Con las PKs definidas pero sin las FKs la consulta que menos tarda es la que tiene el EXISTS. Las consultas de IN y Exists se quedan en el mismo coste pero las consultas de INNER Join y Join bajan notablemente.

### Tiempos y Costes con PKs y FKs
Con las PKs y FKs definidas el tiempo de ejecución llega a subir notablemente pero los costes de ejecución bajan drásticamente.

### Índices
>
```sql
CREATE INDEX idx_cantidad ON detallepedidos(cantidad);
CREATE INDEX idx_productoID ON detallepedidos(productoID);
CREATE INDEX idx_nombre_producto ON productos(nombre_productos);
```

---

### Apartado 2 - Estudio de planes de consulta e índices

### Apartado 7 - Concluciones finales sobre la práctica
Los índices son estructuras de datos que se utilizan para mejorar y optimizar el tiempo de respuesta de las consultas de tablas grandes. Cuando se realiza una operación de UPDATE sobre una tabla que contiene índices el tiempo de respuesta de esta puede llegar a tardar porque además de actualizar los valores en la tabla también hay que actualizarlos en el índice.

La desnormalización es una técnica usada en las bases de datos relacionales para mejorar el rendimiento de las consultas y reducir la complejidad del modelo de datos. Permite una redundancia controlada de datos y la reducción de los JOINS para recuperar información. Mejora el rendimiento de las consultas pero puede afectar la integridad y consistencia de los datos en la BD. También puede aumentar el tamaño de la base de datos afectando las oepraciones de UPDATE.