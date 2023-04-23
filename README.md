## Bases-de-Datos---Practica-2

### Apartado 2 - Optimizacion de Consultas
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

### Apartado 3 - Estudio de planes de consulta e índices
Consulta
>
```sql
SELECT p.nombre_producto, COUNT(*) AS num_pedidos
FROM productos p
INNER JOIN subcategoria s
    ON p.subcategoriaID = s.subCategoriaID
INNER JOIN categoria c
    ON s.categoriaID = c.categoriaID
INNER JOIN detallepedidos d
    ON p.ProductoID = d.ProductoID
WHERE c.nombre_categoria = 'Componente'
    AND s.nombre_subcategoria LIKE '%Cuadro%'
    AND p.color = 'Blue'
GROUP BY p.nombre_producto
HAVING COUNT(*) > (
    SELECT COUNT(*)
    FROM subcategoria sc
    INNER JOIN productos pd
        ON sc.subCategoriaID = pd.subcategoriaID
    INNER JOIN detallepedidos dp
        ON pd.ProductoID = dp.ProductoID
    WHERE sc.nombre_subcategoria IN ('Dirección', 'Horquilla')
    );
```
En el apartado 3 el coste de la consulta anterior es de 681.69.
A continuación creamos los índices necesarios para mejorar la consulta
>
```sql
CREATE UNIQUE INDEX idx_categoria_nombrecategoria ON categoriaproducto(nombre_categoria)
CREATE UNIQUE INDEX idx_subcategoria_nombresubcategoria ON subcategoriaproducto(nombre_subcategoria)
CREATE INDEX productos_color ON productos(color);
```
Con el índice de la columna color el coste de ejecución baja.

---

### Apartado 4 - Estudio de una consulta
Describir en lenguaje natural la consulta
> Devuelve el el primer nombre y apellidos de las mujeres nacidas entre los años 1980 y 1986. Además estas mujeres deben tener los ingresos anuales mayores o iguales al percentil 0.9 de los ingresos anuales a las mujeres nacidas entre los años 1970 y 1980.
Los resultados se agrupan por el id del cliente y se filtran aquellos clientes cuyo total de compras en el años 2021 es >= que la media de compras en el año 2021 para cada cliente.

_Creación de índices para la consulta_
```sql
-- fecha_nacimiento
create index idx_fecha_nacimiento ON clientes(fecha_nacimiento);
-- fecha_venta
create index idx_fecha_venta ON pedido(fecha_venta);
Sin indices - 2972
Indice fecha nacimiento - 3397
indice fecha venta - 4098
solo indice fecha_venta 1.63
```

---
### Apartado 5 - Estudio de índices en actualizaciones
>
```sql
UPDATE detallepedidos
SET descuento_unitario = descuento_unitario+1
WHERE PedidoID IN (
    SELECT PedidoID
    FROM pedidos
    WHERE YEAR(fecha_venta) = 2023
)
AND ProductoID IN (
    SELECT ProductoID
    FROM productos
    WHERE subcategoriaID IN (
        SELECT subCategoriaID
        FROM subcategoriaproducto
        WHERE nombre_subcategoria like '%Bicicleta%'
    )
    
)AND descuento_unitario <> 0;
```
El coste de la operación de UPDATE es de 0.516 sec.
Una vez creado el índice el coste de la operación de UPDATE es de 0.156 sec

---
### Apartado 6 - Desnormalización
La desnormalización es una técnica empleada en las bases de datos relacionales que tiene como fin mejorar el rendimiento de las consultas y reducir la complejidad del modelo relacional. Permite una redundancia controlada de los datos y una reducción de JOINS de la consulta con el fin de recuperar la información.

_Crear una consulta que devuelva, para cada pedido el nombre del cliente, su pais, fecha del pedido y el total del pedido_
>
```sql
SELECT c.primer_nombre, c.apellidos, c.pais, p.fecha_pedido SUM((d.precio_unitario - d.descuento_unitario)*d.cantidad) AS  total_pedido FROM pedidos p
INNER JOIN clientes c ON p.clienteID = c.clienteID
INNER JOIN detallepedidos d ON d.pedidoID = p.pedidoID
GROUP BY pedidoID;
```
_Script para la técnica de desnormalización_
> 
```sql
CREATE TABLE resumen_pedidos(
    pedidoID INT PRIMARY KEY,
    cliente_nombre VARCHAR(255),
    cliente_pais VARCHAR(255),
    pedidoFecha DATE,
    total_pedido DECIMAL(10,2)
);
```
_Script para actualizar los datos implicados en la desnormalización_
>
```sql
INSERT INTO resumen_pedidos(pedidoID, cliente_nombre, cliente_pais, pedidoFecha, total_pedido);
SELECT c.primer_nombre, c.apellidos, c.pais, p.fecha_pedido, SUM((d.precio_unitario - d.descuento_unitario)*d.cantidad) AS total_pedido FROM pedidos p
INNER JOIN clientes c ON c.clienteID = p.clienteID
INNER JOIN detallepedidos d ON d.pedidoID = p.pedidoID
GROUP BY pedidoID
ON DUPLICATE KEY UPDATE
    cliente_nombre = VALUES(cliente_nombre)
    cliente_pais = VALUES(cliente_pais)
    pedidoFecha = VALUES(pedidoFecha)
    total_pedido = VALUES(total_pedido)
```

### Apartado 7 - Concluciones finales sobre la práctica
Los índices son estructuras de datos que se utilizan para mejorar y optimizar el tiempo de respuesta de las consultas de tablas grandes. Cuando se realiza una operación de UPDATE sobre una tabla que contiene índices el tiempo de respuesta de esta puede llegar a tardar porque además de actualizar los valores en la tabla también hay que actualizarlos en el índice.

La desnormalización es una técnica usada en las bases de datos relacionales para mejorar el rendimiento de las consultas y reducir la complejidad del modelo de datos. Permite una redundancia controlada de datos y la reducción de los JOINS para recuperar información. Mejora el rendimiento de las consultas pero puede afectar la integridad y consistencia de los datos en la BD. También puede aumentar el tamaño de la base de datos afectando las oepraciones de UPDATE.