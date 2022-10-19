# sql-coderhouse
repositorio para entrega de proyecto final


--creacion de schema--

CREATE SCHEMA constructora1;

--creacion de tablas --

USE constructora1;
CREATE TABLE clientes (
clientes_id int NOT NULL AUTO_INCREMENT,
fk_arquitectos_id INT NOT NULL,
fk_obras_id INT NOT NULL,
nombre char(50) NOT NULL,
direccion varchar(50) NOT NULL,
email varchar(70) NOT NULL,
telefono int(11) NOT NULL,
presup_dol decimal(8.2) NOT NULL
DEFAULT '0',
id_arquitectos int NOT NULL,
id_obras int NOT NULL,
PRIMARY KEY (clientes_id)
);


CREATE TABLE arquitectos (
arquitectos_id int NOT NULL AUTO_INCREMENT,
fk_clientes_id int NOT NULL,
fk_eq_trabajo_id INT NOT NULL,
fk_obras_id INT NOT NULL,
fk_proveedores_id INT NOT NULL,
PRIMARY KEY (arquitectos_id)
);


CREATE TABLE obras (
obras_id int NOT NULL AUTO_INCREMENT,
fk_clientes_id int NOT NULL,
fk_eq_trabajo_id int NOT NULL,
fk_arquitectos_id int NOT NULL,
tipo char(50) NOT NULL, 
ubicación varchar(50) NOT NULL,
PRIMARY KEY (obras_id)
);

CREATE TABLE eq_trabajo (
eq_trabajo_id int NOT NULL AUTO_INCREMENT,
fk_proveedores_id int NOT NULL,
nombre char(50) NOT NULL,	
cuit_cuil inT(11) NOT NULL,
rubro_jerarquia char(50) NOT NULL,
nro_poliza int(100) NULL,
telefono int(11),

PRIMARY KEY (eq_trabajo_id)
);


CREATE TABLE proveedores (
proveedores_id int NOT NULL AUTO_INCREMENT,
nombre VARCHAR(50) not null,
telefono int(11) NOT NULL,
direccion varchar(50) NOT NULL,
PRIMARY KEY (proveedores_id)
);



CREATE TABLE compras (
id_eq_trabajo int NOT NULL AUTO_INCREMENT,
id_proveedores int not null,
PRIMARY KEY (id_eq_trabajo)
);


CREATE TABLE `logs_clientes` (
  `log_clientes_id` int unsigned NOT NULL AUTO_INCREMENT,
  `fk_arquitectos_id` int NOT NULL,
  `fk_obras_id` int NOT NULL,
  `nombre` varchar(50) NOT NULL,
  `direccion` varchar(50) NOT NULL,
  `email` varchar(70) NOT NULL,
  `telefono` float NOT NULL,
  `presup_dol` decimal(8,0) NOT NULL,
  PRIMARY KEY (`log_clientes_id`)
) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=latin1


CREATE TABLE `logs_eq_trabajo` (
  `logs_eq_trabajo_id` int unsigned NOT NULL AUTO_INCREMENT,
  `fecha_hora` date NOT NULL,
  `tipo` varchar(45) NOT NULL,
  `detalle` varchar(145) NOT NULL,
  PRIMARY KEY (`logs_eq_trabajo_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci


insertar datos-- desde workbench


--creacion de vistas--

--VISTA DE NOMBRE DE CLIENTES QUE SE HA ENTREGADO OBRA AL DIA DE LA FECHA--

Create view obras_entregadas AS 
SELECT c.nombre
FROM clientes AS c, obras AS o
Where o.fecha_final_contrato < curdate();

--VISTA DE UBICACIÓN DE OBRA CON MAYOR PRESUPUESTO DE OBRA--

CREATE VIEW ubicacion_mayor_prespuesto AS
SELECT o.ubicación, MAX(c.presup_dol)
FROM obras as o, clientes as c
INNER JOIN obras ON c.fk_obras_id = obras_id
HAVING MAX(c.presup_dol) > 60000
ORDER BY c.presup_dol DESC;

--VISTA de cliente que realiza el pago de obra por porcentaje de entrega y que lo realiza mediante transferencia--

CREATE VIEW filtro_forma_modo_pago AS
SELECT c.nombre, p.forma_pago, p.modo_pago
FROM clientes as c, pagos as p
WHERE p.forma_pago = "porcentaje de entrega" and p.modo_pago = "transferencia";

--Vista de datos de las personas que trabajan en la obra ubicada en Agua de Oro--

CREATE VIEW datos_eq_agua_de_oro as
SELECT eq.nombre, eq.telefono, o.ubicación
FROM eq_trabajo as eq, obras as o
where ubicación LIKE '% Agua de Oro';

--vista de datos de obras iniciadas éntre los años 2021 y 2022. --

CREATE VIEW nombre_ubicacion_cliente_ultimo_año as
SELECT distinct o.fecha_inicio_contrato, o.ubicación, c.nombre
  FROM obras as o, clientes as c
  INNER JOIN obras ON c.clientes_id = fk_clientes_id
WHERE o.fecha_inicio_contrato BETWEEN '20210101' AND '20221231';

creacion de funciones

--dar tipo de obra segun su nombre-- 

delimiter $$
create function dar_tipo_obra_por_cliente (p_nombre varchar(50))
returns char(50)
deterministic
BEGIN
DECLARE v_tipo char(50);
 SELECT o.tipo, c.nombre	
 into v_tipo
 From obras as o, clientes as c
 where nombre = p_nombre;

 RETURN v_tipo;
end $$


--eq de trabajo activo por obra--

delimiter $$
CREATE FUNCTION eq_trabajo_activo_por_obra (p_obras_id int)
returns int
deterministic
BEGIN
DECLARE v_eq_trabajo_id int;
SELECT eq.eq_trabajo_id, o.obras_id
into v_eq_trabajo_id
From eq_trabajo as eq, obras as o
Where  obras_id = p_obras_id;

return v_eq_trabajo_id;
end $$

creacion de stored procedures

--SP actualizar presupuesto de cliente

CREATE PROCEDURE actualiza_presup(n_presup_dol INT, obras_id INT)
UPDATE clientes SET presup_dol=n_presup_dol WHERE obras=obras_id

--ASC ORDENAR EQ TRABAJO 

delimiter // 
create procedure eq_rubro_jerarquia (IN rubro_jerarquia varchar(50))
BEGIN
select nombre, rubero_jerarquia, eq_trabajo_id
From eq_trabajo
WHERE eq_rubro_jerarquia LIKE rubro_jerarquia;
END//


creacion de triggers

-- trigger eq_trabajo--

CREATE TABLE `logs_eq_trabajo` (
  `logs_eq_trabajo_id` int unsigned NOT NULL AUTO_INCREMENT,
  `fecha_hora` date NOT NULL,
  `tipo` varchar(45) NOT NULL,
  `detalle` varchar(145) NOT NULL,
  PRIMARY KEY (`logs_eq_trabajo_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci

--after--

DELIMITER $$
CREATE TRIGGER trigger_eq_trabajo
AFTER INSERT ON eq_trabajo
FOR EACH ROW
BEGIN 
   INSERT INTO eq_trabajo(nombre, cuit_cuil, rubro_jerarquia, nro_poliza, telefono)
   VALUES (NEW.nombre, NEW.cuit_cuil, NEW.rubro_jerarquia, NEW.nro_poliza, NEW.telefono, CURDATE());
END; $$

--before--

delimiter //
 create trigger before_eq_trabajo_insert
   before insert
   on eq_trabajo
   for each row
 begin
   update nombre, cuit_cuil, rubro_jerarquia, nro_poliza, telefono set eq_trabajo_id=new.eq_trabajo_id
     where new.eq_trabajo_id=eq_trabajo_id; 
 end //
 delimiter ;

--trigger after clientes--

CREATE TABLE `logs_clientes` (
  `log_clientes_id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `fk_arquitectos_id` INT NOT NULL,
  `fk_obras_id` INT NOT NULL,
  `nombre` varchar(50) NOT NULL,
  `direccion` varchar(50) NOT NULL,
  `email` varchar(70) NOT NULL,
  `telefono` float NOT NULL,
  `presup_dol` decimal(8,0) NOT NULL,
  PRIMARY KEY (`log_clientes_id`)
) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=latin1;


DELIMITER $$
CREATE TRIGGER trigger_cliente
AFTER INSERT ON clientes
FOR EACH ROW
BEGIN 
   INSERT INTO clientes(nombre, direccion, email, telefono, presup_dol)
   VALUES (NEW.nombre, NEW.direccion, NEW.email, NEW.telefono, NEW.presup_dol, CURDATE());
END; $$

delimiter //
 create trigger before_clientes_insert
   before insert
   on clientes
   for each row
 begin
   update nombre, direccion, email, telefono,presup_dol set clientes_id=new.clientes_id
     where new.clientes_id=clientes_id; 
 end //
 delimiter ;
 
 
implementacion de sentencias

--Este usuario ha sido creado y deberia poseer habilitado solo la lectura de la base--

Create user 'a_jorge'@'127.0.0.1' identified by 'Admin1234';


--Se le otorga mediante la sentencia GRANT el permiso de lectura de nuestra base "constructoraa"--

grant select on constructoraa.* to 'a_jorge'@'127.0.0.1';


--Visualizamos los permisos que posee el usuario.


Show grants for 'a_jorge'@'127.0.0.1'; 



--Creamos al usuario g_jose éste usuario tiene la capacidad de lectura, modificación y de agregar datos a nuestra base--


CREATE USER g_jose@127.0.0.1 identified by 'Admin12345';



--le otorgamos los permisos de lectura, agregar y modificar datos de la base.


grant select, insert, update on constructoraa.* to g_jose@127.0.0.1;



--visualizamos los permisos del usuario


show grants for g_jose@127.0.0.1; 
