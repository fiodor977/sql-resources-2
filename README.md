# sql-resources-2



CONSULTAS<br>

SELECT ------ SELECCIONAR (COLUMNAS) (* = TODOS) <br>
FROM ------ DE (COLUMNAS) <br>
ORDER BY ------ ORDENAR POR <br>
WHERE ------ DONDE <br>
AS----- CAMBIAR EL NOMBRE <br>
UNION ---- JUNTAR DOS SELECT           ** creo **           (MISMA CATEGORIA DISTINTAS TABLAS) <br>
UNION ALL ----- IGUAL QUE UNION PERO MUESTRA DATOS REPETIDOS <br>
GROUP BY ----- ORDENAR POR <br>
HAVING ----- CONDICION DE FUNCIONES <br>
EXTRACT ----- SELECCIONAR UN PERIODO DE TIEMPO <br>

---------------------------------------------------------------

OPERADORES<br>

AND, OR, NOT, <, >, =, BETWEEN (X AND Y) <br>

LIKE----PARA NOMBRES(LIKE %PALABRA%)<br>

IN----- ESPECIFICAR VALORES (IN 3,4,5,6)<br>

-------------------------------------------------

SUBFUNCIONES<br>

MIN, MAX, COUNT---CONTAR, AVG----MEDIA, SUM---SUMA<br>

---------------------------------------------------

JOINS<br>
INNER JOIN ----- DONDE COINCIDEN LAS DOS TABLAS<br>
LEFT JOIN ------ LA TABLA DE LA IZQ JUNTO AL INNER JOIN<br>
RIGHT JOIN------ LA TABLE DE LA DCHA JUNTO AL INNER JOIN<br>
FULL JOIN ----- LAS DOS TABLAS<br>

----------

INSERT INTO CLIENTE (NIF, RAZON_SOCIAL, DIRECCION, POBLACION, PROVINCIA, TELEFONO, VENTAS) <br>
VALUES ('123456789F', '123T', 'CALLE LA CIRUELA', 'MALAGA', 'MALAGA', '1234789', '12');<br>

UPDATE CLIENTE<br>
SET POBLACION='SEVILLA'<br>
WHERE POBLACION='MALAGA'<br>

UPDATE CLIENTE<br>
SET POBLACION='TENERIFE'<br>
WHERE NIF='123456789F'<br>

------

DELETE CLIENTE<br>
WHERE NIF='123456789F'<br>

-------

importe = unidades * ( precio - precio * dto )<br>

------------------

SELECT  *<br>
FROM pedido<br>
where fecha like '04/01%';<br>

-------------

SELECT  *<br>
FROM pedido<br>
where EXTRACT(month FROM fecha)  = '01' and EXTRACT(day FROM fecha)  = '04'<br>

---------------------------

SELECT * <br>
FROM cliente<br>
WHERE LOWER(provincia) = 'sevilla'  AND ventas > 1000;<br>

SELECT * <br>
FROM cliente<br>
WHERE UPPER(provincia) = 'SEVILLA'  AND ventas > 1000;<br>


---------

SELECT * FROM pedido<br>
WHERE fecha BETWEEN TO_DATE('1/1/2018','DD/MM/YYYY')<br>
            AND TO_DATE('31/01/2018', 'DD/MM/YYYY');


------------------------------

SELECT categoria, articulo.* <br>
FROM articulo<br>
WHERE categoria IN ('CARC', 'BEBI', 'CONF') <br>
    OR precio_venta = 4;<br>

------------------------

SELECT * FROM cliente<br>
WHERE  telefono IS NULL;<br>

------------------------------------

SELECT nif, razon_social, poblacion, provincia<br>
FROM cliente<br>
ORDER BY provincia ASC, poblacion DESC, razon_social DESC;<br>


---------------------------------
SELECT SUM(existencias)<br>
FROM articulo;<br>

----------------

SELECT MAX(precio_venta), MIN(precio_venta)<br>
FROM articulo;<br>

----------------

SELECT COUNT(*)<br>
FROM pedido;<br>

----------------------------------

SELECT AVG(precio_venta) AS precio_medio<br>
FROM articulo<br>
WHERE categoria = 'FRUT';<br>


---------------------

SELECT referencia, <br>
    sum(unidades * (precio - precio* dto)) AS importe<br>
FROM lpedido<br>
GROUP BY referencia;<br>

-------------------------------------

SELECT referencia, SUM(unidades) as suma<br>
FROM lpedido<br>
GROUP BY referencia<br>
HAVING SUM(unidades) > 50;<br>


---------------------------------------

SELECT nif, razon_social, direccion<br>
FROM cliente<br>
WHERE nif IN ( SELECT cliente FROM pedido WHERE total_pedido > 0);<br>

-------------------------

SELECT nfactura, nefecto, fecha_vto<br>
FROM efecto<br>
WHERE fecha_vto <> ( SELECT fecha<br>
                     FROM factura<br>
                     where efecto.nfactura = factura.nfactura);<br>


------------------------

-- Listar la referencia y descripción de los artículos cuyas existencias son superiores a la suma de las unidades pedidas de ese artículo.<br>

SELECT REFERENCIA, DESCRIPCION<br>
FROM ARTICULO<br>
WHERE EXISTENCIAS > (SELECT SUM (UNIDADES) FROM LPEDIDO  WHERE LPEDIDO.REFERENCIA = ARTICULO.REFERENCIA);<br>

------------------------------------

SELECT referencia, unidades * ( precio - precio * dto) as importe<br>
FROM lalbaran<br>
WHERE unidades * ( precio - precio * dto) < 
                    ( SELECT AVG(unidades * ( precio - precio * dto))<br>
                      FROM lpedido);

--------------------------------------
SELECT nif, nombre<br>
FROM representante<br>
WHERE EXISTS (SELECT representante FROM ped_rep <br>
              WHERE representante = nif);<br>

----------------------------------------------------

SELECT nfactura, fecha<br>
FROM factura<br>
WHERE NOT EXISTS ( SELECT * FROM efecto<br>
                    WHERE efecto.nfactura = factura.nfactura);<br>


-------------------------------------------------------

SELECT nif, razon_social<br>
FROM cliente<br>
WHERE ventas >= ALL ( SELECT total_albaran<br>
                      FROM albaran<br>
                      WHERE nif = cliente);<br>

----------------------------------------------------

SELECT nalbaran, cliente, fecha<br>
FROM albaran<br>
WHERE total_albaran <= ANY ( SELECT total_pedido<br>
                             FROM pedido<br>
                             WHERE cliente = '30000001A'<br>
                             AND fecha BETWEEN TO_DATE('01/01/18')<br>
                                        AND TO_DATE('07/01/18')<br>
                            );<br>

---------------------------------------------------------------

SELECT npedido, cliente, fecha<br>
FROM pedido<br>
WHERE npedido IN (SELECT npedido FROM ped_rep<br>
                  WHERE representante = (SELECT nif <br>
                                         FROM representante<br>
                                         WHERE UPPER(nombre) = 'CARMEN RICO')<br>
                 );<br>


---------------------------------------------------------

SELECT cliente, AVG(total_pedido)<br>
FROM pedido<br>
WHERE EXTRACT(MONTH FROM fecha) = 1<br>
GROUP BY cliente<br>
HAVING AVG(total_pedido) > (SELECT AVG(total_pedido) FROM pedido );<br>


----------------------------------------------------------------

SELECT articulo.referencia, descripcion, nalbaran, nlinea, lalbaran.referencia, precio_venta, dto<br>
FROM articulo<br>
INNER JOIN lalbaran ON articulo.referencia = lalbaran.referencia;<br>

SELECT referencia, descripcion, nalbaran, nlinea, referencia, precio_venta, dto <br>
FROM articulo <br>
INNER JOIN lalbaran USING(referencia); <br>


---------------------------------------

DISTINCT

----------------------------------------------------

-- Listar la referencia y descripción de los artículos que se han vendido con descuento.


SELECT DISTINCT ART.REFERENCIA, DESCRIPCION<br>
FROM ARTICULO ART <br>
INNER JOIN LPEDIDO LPED ON ART.REFERENCIA = LPED.REFERENCIA  <br>
WHERE LPED.DTO IS NOT NULL AND LPED.DTO <> 0; <br>

SELECT DISTINCT REFERENCIA, DESCRIPCION <br>
FROM ARTICULO ART  <br>
INNER JOIN LPEDIDO LPED USING(REFERENCIA) <br>
WHERE LPED.DTO IS NOT NULL AND LPED.DTO <> 0; <br>



--------------------------------------------------------

-- Listar nº de albarán, fecha, forma de pago, nº de línea, referencia, descripción de artículo, unidades de los artículos vendidos con unidades superiores a 5 y fecha 4/1 <br>

SELECT albaran.nalbaran, fecha, forma_pago, nlinea, lalbaran.referencia, descripcion, unidades <br>
FROM albaran<br>
INNER JOIN lalbaran ON albaran.nalbaran = lalbaran.nalbaran<br>
INNER JOIN articulo ON lalbaran.referencia = articulo.referencia<br>
WHERE unidades > 5 AND fecha = TO_DATE('4/1/18');<br>

SELECT nalbaran, fecha, forma_pago, nlinea, referencia descripcion, unidades <br>
FROM albaran <br>
INNER JOIN lalbaran USING(nalbaran) <br>
INNER JOIN articulo USING(referencia) <br>
WHERE unidades > 5 AND fecha = TO_DATE('4/1/18'); <br>


--------------------------------------------------------------------------

SELECT npedido, cliente, fecha, total_pedido, forma_pago, descripcion <br>
FROM pedido<br>
FULL JOIN forma_pago USING(forma_pago);<br>
