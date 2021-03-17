# sql-resources-2



CONSULTAS

SELECT ------ SELECCIONAR (COLUMNAS) (* = TODOS)
FROM ------ DE (COLUMNAS)
ORDER BY ------ ORDENAR POR
WHERE ------ DONDE
AS----- CAMBIAR EL NOMBRE
UNION ---- JUNTAR DOS SELECT           ** creo **           (MISMA CATEGORIA DISTINTAS TABLAS)
UNION ALL ----- IGUAL QUE UNION PERO MUESTRA DATOS REPETIDOS
GROUP BY ----- ORDENAR POR
HAVING ----- CONDICION DE FUNCIONES
EXTRACT ----- SELECCIONAR UN PERIODO DE TIEMPO

---------------------------------------------------------------

OPERADORES

AND, OR, NOT, <, >, =, BETWEEN (X AND Y) 

LIKE----PARA NOMBRES(LIKE %PALABRA%)

IN----- ESPECIFICAR VALORES (IN 3,4,5,6)

-------------------------------------------------

SUBFUNCIONES

MIN, MAX, COUNT---CONTAR, AVG----MEDIA, SUM---SUMA

---------------------------------------------------

JOINS
INNER JOIN ----- DONDE COINCIDEN LAS DOS TABLAS
LEFT JOIN ------ LA TABLA DE LA IZQ JUNTO AL INNER JOIN
RIGHT JOIN------ LA TABLE DE LA DCHA JUNTO AL INNER JOIN
FULL JOIN ----- LAS DOS TABLAS

----------

INSERT INTO CLIENTE (NIF, RAZON_SOCIAL, DIRECCION, POBLACION, PROVINCIA, TELEFONO, VENTAS) 
VALUES ('123456789F', '123T', 'CALLE LA CIRUELA', 'MALAGA', 'MALAGA', '1234789', '12');

UPDATE CLIENTE
SET POBLACION='SEVILLA'
WHERE POBLACION='MALAGA'

UPDATE CLIENTE
SET POBLACION='TENERIFE'
WHERE NIF='123456789F'

------

DELETE CLIENTE
WHERE NIF='123456789F'

-------

unidades * ( precio - precio * dto )

------------------

SELECT  *
FROM pedido
where fecha like '04/01%';

-------------

SELECT  *
FROM pedido
where EXTRACT(month FROM fecha)  = '01' and EXTRACT(day FROM fecha)  = '04'

---------------------------

SELECT * 
FROM cliente
WHERE LOWER(provincia) = 'sevilla'  AND ventas > 1000;

SELECT * 
FROM cliente
WHERE UPPER(provincia) = 'SEVILLA'  AND ventas > 1000;


---------

SELECT * FROM pedido
WHERE fecha BETWEEN TO_DATE('1/1/2018','DD/MM/YYYY')
            AND TO_DATE('31/01/2018', 'DD/MM/YYYY');


------------------------------

SELECT categoria, articulo.* 
FROM articulo
WHERE categoria IN ('CARC', 'BEBI', 'CONF') 
    OR precio_venta = 4;

------------------------

SELECT * FROM cliente
WHERE  telefono IS NULL;

------------------------------------

SELECT nif, razon_social, poblacion, provincia
FROM cliente
ORDER BY provincia ASC, poblacion DESC, razon_social DESC;


---------------------------------
SELECT SUM(existencias)
FROM articulo;

----------------

SELECT MAX(precio_venta), MIN(precio_venta)
FROM articulo;

----------------

SELECT COUNT(*)
FROM pedido;

----------------------------------

SELECT AVG(precio_venta) AS precio_medio
FROM articulo
WHERE categoria = 'FRUT';


---------------------

SELECT referencia, 
    sum(unidades * (precio - precio* dto)) AS importe
FROM lpedido
GROUP BY referencia;

-------------------------------------

SELECT referencia, SUM(unidades) as suma
FROM lpedido
GROUP BY referencia
HAVING SUM(unidades) > 50;


---------------------------------------

SELECT nif, razon_social, direccion
FROM cliente
WHERE nif IN ( SELECT cliente FROM pedido WHERE total_pedido > 0);

-------------------------

SELECT nfactura, nefecto, fecha_vto
FROM efecto
WHERE fecha_vto <> ( SELECT fecha
                     FROM factura
                     where efecto.nfactura = factura.nfactura);


------------------------

-- Listar la referencia y descripción de los
-- artículos cuyas existencias son superiores
-- a la suma de las unidades pedidas de ese artículo.

SELECT REFERENCIA, DESCRIPCION
FROM ARTICULO
WHERE EXISTENCIAS > (SELECT SUM (UNIDADES) FROM LPEDIDO  WHERE LPEDIDO.REFERENCIA = ARTICULO.REFERENCIA);

------------------------------------

SELECT referencia, unidades * ( precio - precio * dto) as importe
FROM lalbaran
WHERE unidades * ( precio - precio * dto) < 
                    ( SELECT AVG(unidades * ( precio - precio * dto))
                      FROM lpedido);

--------------------------------------
SELECT nif, nombre
FROM representante
WHERE EXISTS (SELECT representante FROM ped_rep 
              WHERE representante = nif);

----------------------------------------------------

SELECT nfactura, fecha
FROM factura
WHERE NOT EXISTS ( SELECT * FROM efecto
                    WHERE efecto.nfactura = factura.nfactura);


-------------------------------------------------------

SELECT nif, razon_social
FROM cliente
WHERE ventas >= ALL ( SELECT total_albaran
                      FROM albaran
                      WHERE nif = cliente);

----------------------------------------------------

SELECT nalbaran, cliente, fecha
FROM albaran
WHERE total_albaran <= ANY ( SELECT total_pedido
                             FROM pedido
                             WHERE cliente = '30000001A'
                             AND fecha BETWEEN TO_DATE('01/01/18')
                                        AND TO_DATE('07/01/18')
                            );

---------------------------------------------------------------

SELECT npedido, cliente, fecha
FROM pedido
WHERE npedido IN (SELECT npedido FROM ped_rep
                  WHERE representante = (SELECT nif 
                                         FROM representante
                                         WHERE UPPER(nombre) = 'CARMEN RICO')
                 );


---------------------------------------------------------

SELECT cliente, AVG(total_pedido)
FROM pedido
WHERE EXTRACT(MONTH FROM fecha) = 1
GROUP BY cliente
HAVING AVG(total_pedido) > (SELECT AVG(total_pedido) FROM pedido );


----------------------------------------------------------------

SELECT articulo.referencia, descripcion, nalbaran, nlinea, lalbaran.referencia,
precio_venta, dto
FROM articulo
INNER JOIN lalbaran ON articulo.referencia = lalbaran.referencia;

SELECT referencia, descripcion, nalbaran, nlinea, referencia,
precio_venta, dto
FROM articulo
INNER JOIN lalbaran USING(referencia);


---------------------------------------

DISTINCT

----------------------------------------------------

-- Listar la referencia y descripción de los 
-- artículos que se han vendido con descuento.


SELECT DISTINCT ART.REFERENCIA, DESCRIPCION
FROM ARTICULO ART 
INNER JOIN LPEDIDO LPED ON ART.REFERENCIA = LPED.REFERENCIA 
WHERE LPED.DTO IS NOT NULL AND LPED.DTO <> 0;

SELECT DISTINCT REFERENCIA, DESCRIPCION
FROM ARTICULO ART 
INNER JOIN LPEDIDO LPED USING(REFERENCIA)
WHERE LPED.DTO IS NOT NULL AND LPED.DTO <> 0;
18 de marzo de 2021


--------------------------------------------------------

-- Listar nº de albarán, fecha, forma de pago, nº de línea, 
-- referencia, descripción de artículo, unidades
-- de los artículos vendidos con unidades superiores a 5
-- y fecha 4/1

SELECT albaran.nalbaran, fecha, forma_pago, nlinea, lalbaran.referencia
descripcion, unidades
FROM albaran
INNER JOIN lalbaran ON albaran.nalbaran = lalbaran.nalbaran
INNER JOIN articulo ON lalbaran.referencia = articulo.referencia
WHERE unidades > 5 AND fecha = TO_DATE('4/1/18');

SELECT nalbaran, fecha, forma_pago, nlinea, referencia
descripcion, unidades
FROM albaran
INNER JOIN lalbaran USING(nalbaran)
INNER JOIN articulo USING(referencia)
WHERE unidades > 5 AND fecha = TO_DATE('4/1/18');


--------------------------------------------------------------------------

SELECT npedido, cliente, fecha, total_pedido, forma_pago,
descripcion
FROM pedido
FULL JOIN forma_pago USING(forma_pago);
