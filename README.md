# MySQL2

## Actividad Uso de eventos con procedimientos

Haciendo uso de las siguientes tablas para la base de datos de `pizza` realice los siguientes ejercicios de `Events` centrados en el uso de **ON COMPLETION PRESERVE** y **ON COMPLETION NOT PRESERVE** :

## Tablas 
Se trabajara con la siguiente base de datos:

```
--BASE DE DATOS:

CREATE DATABASE pizza;
USE pizza;

-- CREAR TABLAS

CREATE TABLE IF NOT EXISTS resumen_ventas (
fecha       DATE      PRIMARY KEY,
total_pedidos INT,
total_ingresos DECIMAL(12,2),
creado_en DATETIME DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE ingrediente (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(30)
);  

CREATE TABLE IF NOT EXISTS alerta_stock (
  id              INT AUTO_INCREMENT PRIMARY KEY,
  ingrediente_id  INT UNSIGNED NOT NULL,
  stock_actual    INT NOT NULL,
  fecha_alerta    DATETIME NOT NULL,
  creado_en DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (ingrediente_id) REFERENCES ingrediente(id)
);

CREATE TABLE pedidos (
    id INT AUTO_INCREMENT PRIMARY KEY,
    fecha_pedido DATETIME NOT NULL,
    total DECIMAL(10,2) NOT NULL
);

```

-- Datos de prueba sugeridos

```
INSERT INTO ingrediente (nombre, stock) VALUES
('Queso mozzarella', 3),
('Pepperoni', 12),
('Jamón', 4),
('Champiñones', 7),
('Tomate', 2),
('Aceitunas', 9);

INSERT INTO pedidos (fecha_pedido, total) VALUES
(NOW() - INTERVAL 1 DAY, 150.00),
(NOW() - INTERVAL 1 DAY, 220.50),
(NOW() - INTERVAL 8 DAY, 180.75),
(NOW() - INTERVAL 5 DAY, 205.25),
(NOW() - INTERVAL 3 DAY, 99.99),
(NOW() - INTERVAL 7 DAY, 130.00);

```

## 📌 Instrucciones para la realizacion de los ejercicios

Desarrolla procedimientos almacenados (PROCEDURE) y eventos (EVENT) en MySQL para automatizar tareas comunes del negocio de la pizzería. Presta atención al uso de ON COMPLETION PRESERVE o NOT PRESERVE según el caso.

## EJERCICIOS:

## 1. 📅 Resumen Diario Único
Crear un procedimiento y evento que genere un resumen de ventas una sola vez, al finalizar el día anterior, y luego el evento se elimine automáticamente.
📌 Instrucciones
Nombre del procedimiento: generar_resumen_diario
Nombre del evento: ev_resumen_diario_unico
Requiere: ON COMPLETION NOT PRESERVE

## 2. 📆 Resumen Semanal Recurrente
Crear un procedimiento y evento que cada lunes a la 01:00 AM calcule el total de pedidos e ingresos de la semana pasada, y se siga ejecutando semanalmente.
📌 Instrucciones
Nombre del procedimiento: generar_resumen_semanal
Nombre del evento: ev_resumen_semanal
Requiere: ON COMPLETION PRESERVE

## 3. ⚠️ Alerta de Stock Bajo Única
Crear un procedimiento y evento que detecte ingredientes con stock < 5 y registre alertas una única vez (ej. durante el arranque del sistema), eliminando luego el evento.
📌 Instrucciones
Nombre del procedimiento: alerta_stock_bajo
Nombre del evento: ev_alerta_stock_bajo_unica
Requiere: ON COMPLETION NOT PRESERVE

## 4. 🔁 Monitoreo Continuo de Stock
Crear un procedimiento y evento que cada 30 minutos revise los ingredientes con stock < 10 y registre alertas, dejando el evento activo permanentemente.
📌 Instrucciones
Nombre del procedimiento: monitorear_stock_bajo
Nombre del evento: ev_monitor_stock_bajo
Requiere: ON COMPLETION PRESERVE

## 5. 🧹 Limpieza de Resúmenes Antiguos
Crear un procedimiento y evento que una sola vez elimine de resumen_ventas los registros con fecha anterior a hace 365 días, y luego el evento se elimine.
📌 Instrucciones
Nombre del procedimiento: purgar_resumen_antiguo
Nombre del evento: ev_purgar_resumen_antiguo
Requiere: ON COMPLETION NOT PRESERVE

## ✅ Consideraciones

- Asegúrate de tener el programador de eventos activo:
```
SHOW VARIABLES LIKE 'event_scheduler';
```

-Si no esta activo activalo:
```
SET GLOBAL event_scheduler = ON;
```

- Puedes verificar los eventos con:
```
SHOW EVENTS;
```

- Para consultar la definicion de un evento:
```
SHOW CREATE EVENT nombre_evento;
```

## REALIZADO POR: Leidy Johana Niño Villegas.
