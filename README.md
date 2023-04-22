### Bases-de-Datos---Practica-2

#### Apartado 1 - Optimizacion de Consultas
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
#### Tiempos y Costes sin PKs y FKs
Sin las claves primarias y foráneas la consulta con el JOIN es la que menos tarda. En cuanto a costes, INNER JOIN y JOIN son las que más costes tienen

#### Tiempos y Costes con PKs y sin FKs
Con las PKs definidas pero sin las FKs la consulta que menos tarda es la que tiene el EXISTS. Las consultas de IN y Exists se quedan en el mismo coste pero las consultas de INNER Join y Join bajan notablemente.

#### Tiempos y Costes con PKs y FKs
Con las PKs y FKs definidas el tiempo de ejecución llega a subir notablemente pero los costes de ejecución bajan drásticamente.

#### Índices
>
```sql
CREATE INDEX idx_cantidad ON detallepedidos(cantidad);
CREATE INDEX idx_productoID ON detallepedidos(productoID);
CREATE INDEX idx_nombre_producto ON productos(nombre_productos);
```



