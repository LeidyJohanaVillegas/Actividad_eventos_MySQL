## ACTIVIDAD EVENTOS.


## 📌 Siga las instrucciones: 
- Tener MySQL.
- Crear la base de datos y seleccionarla para trabajar en ella.
- Crear las tablas.
- Crear los procedimientos y eventos que se evidencian en las respuestas de los ejercicios propuestos.


### BASE DE DATOS

```
CREATE DATABASE pizza;
USE pizza;
```

-- TABLAS 
--resumen_ventas
```
CREATE TABLE IF NOT EXISTS resumen_ventas (
fecha       DATE      PRIMARY KEY,
total_pedidos INT,
total_ingresos DECIMAL(12,2),
creado_en DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

--ingrediente
```
CREATE TABLE ingrediente (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(30),
    stock INT 
);  
```

--alerta_stock
```
CREATE TABLE IF NOT EXISTS alerta_stock (
  id              INT AUTO_INCREMENT PRIMARY KEY,
  ingrediente_id  INT UNSIGNED NOT NULL,
  stock_actual    INT NOT NULL,
  fecha_alerta    DATETIME NOT NULL,
  creado_en DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (ingrediente_id) REFERENCES ingrediente(id)
);
```

--pedidos
```
CREATE TABLE pedidos (
    id INT AUTO_INCREMENT PRIMARY KEY,
    fecha_pedido DATETIME NOT NULL,
    total DECIMAL(10,2) NOT NULL
);
```


## DATOS DE PRUBA:
```
INSERT INTO ingrediente (nombre, stock) VALUES ('Queso mozzarella', 3), ('Pepperoni', 12), ('Jamón', 4), ('Champiñones', 7), ('Tomate', 2), ('Aceitunas', 9);

INSERT INTO pedidos (fecha_pedido, total) VALUES (NOW() - INTERVAL 1 DAY, 150.00), (NOW() - INTERVAL 1 DAY, 220.50), (NOW() - INTERVAL 8 DAY, 180.75), (NOW() - INTERVAL 5 DAY, 205.25), (NOW() - INTERVAL 3 DAY, 99.99), (NOW() - INTERVAL 7 DAY, 130.00);
```


1. 📆 Resumen Diario Único : crear un evento que genere un resumen de ventas **una sola vez** al finalizar el día de ayer y luego se elimine automáticamente llamado `ev_resumen_diario_unico`.

PROCEDIMIENTO:
```
DELIMITER //
CREATE PROCEDURE generar_resumen_diario()
BEGIN
    INSERT INTO resumen_ventas (fecha, total_pedidos, total_ingresos)
    SELECT CURDATE() - INTERVAL 1 DAY, COUNT(*), SUM(total)
    FROM pedidos
    WHERE DATE(fecha_pedido) = CURDATE() - INTERVAL 1 DAY;
END //
DELIMITER ;
```
EVENTO:
```
CREATE EVENT ev_resumen_diario_unico
ON SCHEDULE AT CURRENT_DATE
ON COMPLETION NOT PRESERVE
DO CALL generar_resumen_diario();
```

2. 📆 Resumen Semanal Recurrente: cada lunes a las 01:00 AM, generar el total de pedidos e ingresos de la semana pasada, **manteniendo** el evento para que siga ejecutándose cada semana llamado `ev_resumen_semanal`.

PROCEDIMIENTO:
```
DELIMITER //
CREATE PROCEDURE generar_resumen_semanal()
BEGIN
    INSERT INTO resumen_ventas (fecha, total_pedidos, total_ingresos)
    SELECT CURDATE() - INTERVAL 7 DAY, COUNT(*), SUM(total)
    FROM pedidos
    WHERE fecha_pedido BETWEEN CURDATE() - INTERVAL 7 DAY AND CURDATE() - INTERVAL 1 DAY;
END //
DELIMITER ;
```
EVENTO:
```
CREATE EVENT ev_resumen_semanal
ON SCHEDULE EVERY 1 WEEK
STARTS (CURRENT_DATE + INTERVAL 1 - WEEKDAY(CURRENT_DATE) DAY + INTERVAL 1 HOUR)
ON COMPLETION PRESERVE
DO CALL generar_resumen_semanal();
```

3. ⚠️ Alerta de Stock Bajo Única: en un futuro arranque del sistema (requerimiento del sistema), generar una única pasada de alertas (`alerta_stock`) de ingredientes con stock < 5, y luego autodestruir el evento.

PROCEDIMIENTO:
```
DELIMITER //
CREATE PROCEDURE alerta_stock_bajo()
BEGIN
    INSERT INTO alerta_stock (ingrediente_id, stock_actual, fecha_alerta)
    SELECT id, stock, NOW()
    FROM ingrediente
    WHERE stock < 5;
END //
DELIMITER ;
```
EVENTO:
```
CREATE EVENT ev_alerta_stock_bajo_unica
ON SCHEDULE AT CURRENT_TIMESTAMP + INTERVAL 1 MINUTE
ON COMPLETION NOT PRESERVE
DO CALL alerta_stock_bajo();
```

4. 🔁 Monitoreo Continuo de Stock: cada 30 minutos, revisar ingredientes con stock < 10 e insertar alertas en `alerta_stock`, **dejando** el evento activo para siempre llamado `ev_monitor_stock_bajo`.

PROCEDIMIENTO:
```
DELIMITER //
CREATE PROCEDURE monitorear_stock_bajo()
BEGIN
    INSERT INTO alerta_stock (ingrediente_id, stock_actual, fecha_alerta)
    SELECT id, stock, NOW()
    FROM ingrediente
    WHERE stock < 10;
END //
DELIMITER ;
```
EVENTO:
```
CREATE EVENT ev_monitor_stock_bajo
ON SCHEDULE EVERY 30 MINUTE
STARTS CURRENT_TIMESTAMP
ON COMPLETION PRESERVE
DO CALL monitorear_stock_bajo();
```

5. 🧹 Limpieza de Resúmenes Antiguos: una sola vez, eliminar de `resumen_ventas` los registros con fecha anterior a hace 365 días y luego borrar el evento llamado `ev_purgar_resumen_antiguo`.

PROCEDIMIENTO:
```
DELIMITER //
CREATE PROCEDURE purgar_resumen_antiguo()
BEGIN
    DELETE FROM resumen_ventas
    WHERE fecha < CURDATE() - INTERVAL 365 DAY;
END //
DELIMITER ;
```
EVENTO:
```
CREATE EVENT ev_purgar_resumen_antiguo
ON SCHEDULE AT CURRENT_TIMESTAMP + INTERVAL 1 MINUTE
ON COMPLETION NOT PRESERVE
DO CALL purgar_resumen_antiguo();
```
ANEXOS:

SI DESEAS REVISAR LOS DATOS INGRESADOS INSERTA LOS SIGUIENTES CODIGOS:
```
SELECT fecha, total_pedidos, total_ingresos, creado_en FROM resumen_ventas;
```
```
+------------+---------------+----------------+---------------------+
| fecha      | total_pedidos | total_ingresos | creado_en           |
+------------+---------------+----------------+---------------------+
| 2025-07-08 |             2 |         370.50 | 2025-07-09 18:56:40 |
+------------+---------------+----------------+---------------------+
```
```
SELECT id, ingrediente_id, stock_actual, fecha_alerta, creado_en FROM alerta_stock;
```
```
+----+----------------+--------------+---------------------+---------------------+
| id | ingrediente_id | stock_actual | fecha_alerta        | creado_en           |
+----+----------------+--------------+---------------------+---------------------+
|  1 |              1 |            3 | 2025-07-09 18:56:51 | 2025-07-09 18:56:51 |
|  2 |              3 |            4 | 2025-07-09 18:56:51 | 2025-07-09 18:56:51 |
|  3 |              5 |            2 | 2025-07-09 18:56:51 | 2025-07-09 18:56:51 |
+----+----------------+--------------+---------------------+---------------------+
```
SI DESEAS VER LOS INGREDIENTES EN LAS ALERTAS:
```
SELECT a.id, i.nombre AS ingrediente, a.stock_actual, a.fecha_alerta, a.creado_en FROM alerta_stock a JOIN ingrediente i ON a.ingrediente_id = i.id;
```
```
+----+------------------+--------------+---------------------+---------------------+
| id | ingrediente      | stock_actual | fecha_alerta        | creado_en           |
+----+------------------+--------------+---------------------+---------------------+
|  1 | Queso mozzarella |            3 | 2025-07-09 18:56:51 | 2025-07-09 18:56:51 |
|  2 | Jamón            |            4 | 2025-07-09 18:56:51 | 2025-07-09 18:56:51 |
|  3 | Tomate           |            2 | 2025-07-09 18:56:51 | 2025-07-09 18:56:51 |
+----+------------------+--------------+---------------------+---------------------+
```
