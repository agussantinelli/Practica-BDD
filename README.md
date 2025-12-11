<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
</head>
<body>

    <h1 align="center">Laboratorio SQL: Prácticas 9 a 12</h1>
    <hr>

    <!-- PRÁCTICA 9 -->
    <h2>🚀 Práctica 9: INSERT SELECT, UPDATE y DELETE con JOINS</h2>
    <p><b>BASE DE DATOS:</b> <code>AFATSE</code></p>

    <h3>🔹 Ejercicio 1</h3>
    <p>Crear una nueva lista de precios para todos los planes de capacitación, a partir del 01/06/2009 con un 20 por ciento más que su último valor. Eliminar las filas agregadas.</p>
    <pre><code>start transaction;

insert into `valores_plan`( `nom_plan`, `fecha_desde_plan`, `valor_plan`)
select val.`nom_plan`,'20090601', val.`valor_plan`*1.2
from valores_plan val
inner join
(
select vp.`nom_plan`, max(vp.`fecha_desde_plan`) ult_fecha
from valores_plan vp
group by vp.`nom_plan`
) fechas
on val.`nom_plan`=fechas.nom_plan
and val.`fecha_desde_plan`=fechas.ult_fecha;

commit;</code></pre>

    <h3>🔹 Ejercicio 2 (Opción A - Recomendada)</h3>
    <p>Crear una nueva lista de precios a partir del 01/08/2009. Regla: Cursos con valor menor a $90 aumentan 20%, el resto aumenta 12%.</p>
    <pre><code>start transaction;

insert into `valores_plan`( `nom_plan`, `fecha_desde_plan`, `valor_plan`)
select val.`nom_plan`,'20090801',
case
    when val.`valor_plan` < 90 then val.`valor_plan`*1.2
    else val.`valor_plan`*1.12
end
from valores_plan val
inner join
(
select vp.`nom_plan`, max(vp.`fecha_desde_plan`) ult_fecha
from valores_plan vp
group by vp.`nom_plan`
) fechas
on val.`nom_plan`=fechas.nom_plan
and val.`fecha_desde_plan`=fechas.ult_fecha;

commit;</code></pre>

    <h3>🔹 Ejercicio 2 (Opción B - Multiples Inserts)</h3>
    <p>Mismo ejercicio anterior pero resuelto con dos sentencias INSERT separadas (Importante respetar el orden lógico).</p>
    <pre><code>start transaction;

/* Primero los mayores o iguales a 90 */
insert into `valores_plan`( `nom_plan`, `fecha_desde_plan`, `valor_plan`)
select val.`nom_plan`,'20090801', val.`valor_plan`*1.12
from valores_plan val
inner join
(
select vp.`nom_plan`, max(vp.`fecha_desde_plan`) ult_fecha
from valores_plan vp
group by vp.`nom_plan`
) fechas
on val.`nom_plan`=fechas.nom_plan
and val.`fecha_desde_plan`=fechas.ult_fecha
where val.`valor_plan`>=90;

/* Luego los menores a 90 */
insert into `valores_plan`( `nom_plan`, `fecha_desde_plan`, `valor_plan`)
select val.`nom_plan`,'20090801', val.`valor_plan`*1.2
from valores_plan val
inner join
(
select vp.`nom_plan`, max(vp.`fecha_desde_plan`) ult_fecha
from valores_plan vp
group by vp.`nom_plan`
) fechas
on val.`nom_plan`=fechas.nom_plan
and val.`fecha_desde_plan`=fechas.ult_fecha
where val.`valor_plan`<90;

commit;</code></pre>

    <h3>🔹 Ejercicio 3</h3>
    <p>Crear un nuevo plan: "Marketing 1 Presen" con los mismos datos que "Marketing 1" pero modalidad presencial. Copiar temas, exámenes y materiales. Costo un 50% superior para los períodos de este año.</p>
    <pre><code>start transaction;

/* Cabecera del plan */
insert into plan_capacitacion
select 'Marketing 1 Presen', desc_plan,hs,'presencial'
from `plan_capacitacion`
where nom_plan= 'Marketing 1';

/* Temas */
insert into plan_temas
select 'Marketing 1 Presen', titulo,detalle
from plan_temas
where nom_plan= 'Marketing 1';

/* Exámenes */
insert into `examenes`
select 'Marketing 1 Presen', nro_examen
from `examenes`
where nom_plan= 'Marketing 1';

/* Temas de Exámenes */
insert into `examenes_temas`
select 'Marketing 1 Presen',titulo,nro_examen
from `examenes_temas`
where nom_plan= 'Marketing 1';

/* Valores */
insert into `valores_plan`(`nom_plan`, `fecha_desde_plan`, `valor_plan`)
select 'Marketing 1 Presen',fecha_desde_plan,valor_plan*1.5
from `valores_plan`
where nom_plan= 'Marketing 1' and year(fecha_desde_plan)=year(CURRENT_DATE);

commit;</code></pre>

    <h3>🔹 Ejercicio 4</h3>
    <p>Cambiar el supervisor de los instructores que dictan "Reparac PC Avanzada" a Franz Kafka (66-66666666-6).</p>
    <pre><code>start transaction;

update `instructores` i 
inner join `cursos_instructores` ci on ci.`cuil`=i.`cuil`
set `cuil_supervisor` ='66-66666666-6'
where ci.`nom_plan`='Reparac PC Avanzada';

commit;</code></pre>

    <h3>🔹 Ejercicio 5</h3>
    <p>Adelantar 1 hora el horario de los cursos que dicta Franz Kafka este año desde las 16 hs.</p>
    <pre><code>start transaction;

update `cursos_horarios` ch 
inner join `cursos_instructores` ci on ch.`nom_plan`=ci.`nom_plan` and ch.`nro_curso`=ci.`nro_curso` 
inner join `cursos` c on ci.`nom_plan`=c.`nom_plan` and ci.`nro_curso`=c.`nro_curso`

set ch.`hora_inicio`=ADDTIME(ch.`hora_inicio`,-010000),
    ch.`hora_fin`=ADDTIME(ch.`hora_fin`,-010000)

where ci.`cuil`='66-66666666-6' 
and ch.`hora_inicio`='160000'
and year(c.`fecha_ini`)=year(CURRENT_DATE);

commit;</code></pre>

    <h3>🔹 Ejercicio 6</h3>
    <p>Eliminar exámenes con promedio general menor a 5.5 y sus temas asociados.</p>
    <pre><code>start transaction;

drop temporary table if exists exa_elim;

create temporary table exa_elim
(
select ev.`nom_plan`,ev.`nro_examen`, AVG(ev.nota) promedio
from `evaluaciones` ev
group by ev.`nom_plan`,ev.`nro_examen`
having promedio < 5.5
);

/* Eliminar evaluaciones y temas asociados (Delete Multitabla) */
delete ev,et
from evaluaciones ev
inner join exa_elim
on ev.`nom_plan`=exa_elim.nom_plan and ev.`nro_examen`=exa_elim.nro_examen
inner join `examenes_temas` et on et.`nom_plan`=exa_elim.nom_plan
and et.`nro_examen`=exa_elim.nro_examen;

/* Eliminar el examen padre */
delete ex
from exa_elim
inner join `examenes` ex on ex.`nom_plan`=exa_elim.`nom_plan`
and ex.`nro_examen`=exa_elim.`nro_examen`;

commit;</code></pre>

    <h3>🔹 Ejercicio 7</h3>
    <p>Eliminar inscripciones de cursos de este año para alumnos con deuda del año anterior.</p>
    <pre><code>start transaction;

delete insc
from inscripciones insc inner join
(
select distinct dni
from cuotas
where fecha_pago is null
and anio=year(CURRENT_DATE)-1
) deudores
on insc.`dni`=deudores.dni
where year(insc.`fecha_inscripcion`)=year(CURRENT_DATE);

commit;</code></pre>

    <br><hr>

    <!-- PRÁCTICA 10 -->
    <h2>🔐 Práctica 10: DCL (Seguridad)</h2>
    <p><b>BASE DE DATOS:</b> <code>AGENCIA_PERSONAL</code></p>
    <p><i>Nota: Verificar cambios en <code>information_schema.table_privileges</code> y <code>user_privileges</code>.</i></p>

    <h3>🔹 Ejercicio 1</h3>
    <p>Crear el usuario 'usuario' con contraseña 'entre'.</p>
    <pre><code>CREATE USER usuario@localhost identified by 'entre';</code></pre>

    <h3>🔹 Ejercicio 2</h3>
    <p>Cambiar la contraseña de usuario a 'entrar'.</p>
    <pre><code>SET PASSWORD FOR 'usuario'@'localhost' = PASSWORD('entrar');</code></pre>

    <h3>🔹 Ejercicio 3</h3>
    <p>Darle permisos a usuario para realizar SELECT de todas las tablas.</p>
    <pre><code>GRANT SELECT ON agencia_personal.* to usuario@localhost;</code></pre>

    <h3>🔹 Ejercicio 4</h3>
    <p>Darle permisos (INSERT, UPDATE, DELETE) sobre la tabla PERSONAS.</p>
    <pre><code>GRANT UPDATE ON agencia_personal.personas to usuario@localhost;
GRANT INSERT ON agencia_personal.personas to usuario@localhost;
GRANT DELETE ON agencia_personal.personas to usuario@localhost;

/* O todo junto */
GRANT all PRIVILEGES ON agencia_personal.personas to usuario@localhost;</code></pre>

    <h3>🔹 Ejercicio 5</h3>
    <p>Quitarle todos los permisos al usuario.</p>
    <pre><code>REVOKE SELECT ON agencia_personal.* FROM usuario@localhost;
REVOKE INSERT ON agencia_personal.* FROM usuario@localhost;
REVOKE DELETE ON agencia_personal.* FROM usuario@localhost;
REVOKE UPDATE ON agencia_personal.* FROM usuario@localhost;

/* O todo junto */
REVOKE all PRIVILEGES ON agencia_personal.* FROM usuario@localhost;</code></pre>

    <h3>🔹 Ejercicio 6</h3>
    <p>Dar permisos CRUD sobre la vista `vw_contratos` (estrategia para ocultar columnas sensibles).</p>
    <pre><code>GRANT UPDATE ON agencia_personal.vw_contratos to usuario@localhost;
GRANT INSERT ON agencia_personal.vw_contratos to usuario@localhost;
GRANT DELETE ON agencia_personal.vw_contratos to usuario@localhost;</code></pre>

    <br><hr>

    <!-- PRÁCTICA 11 -->
    <h2>⚡ Práctica 11: TCL (Control de Transacciones)</h2>
    <p><b>BASE DE DATOS:</b> <code>AFATSE</code></p>

    <h3>🔹 Ejercicio 1: Autocommit Activado</h3>
    <p>Verificar el estado del autocommit.</p>
    <pre><code>SET AUTOCOMMIT=1; 
/* SELECT @@autocommit; */</code></pre>

    <h3>🔹 Ejercicios 2 y 3: Sin Transacción Explícita</h3>
    <p>Crear y eliminar datos. Al estar Autocommit=1, los cambios se ven inmediatamente en la conexión secundaria.</p>
    <pre><code>/* Insertar */
INSERT INTO alumnos ...;
/* Verificar en sesión secundaria: SE VE. */

/* Eliminar */
DELETE FROM inscripciones ...;
/* Verificar en sesión secundaria: YA NO SE VE. */</code></pre>

    <h3>🔹 Ejercicio 4: Uso de ROLLBACK</h3>
    <p>Iniciar transacción explícita, insertar datos y deshacer cambios.</p>
    <pre><code>START TRANSACTION;
/* Insertar datos... */
/* Verificar en secundaria: NO SE VEN (Aislamiento por defecto). */

ROLLBACK;
/* Los datos desaparecen en la sesión principal. */</code></pre>

    <h3>🔹 Ejercicio 5: Uso de COMMIT</h3>
    <p>Iniciar transacción explícita, insertar datos y confirmar cambios.</p>
    <pre><code>START TRANSACTION;
/* Insertar datos... */

COMMIT;
/* Los datos ahora son visibles en ambas sesiones. */</code></pre>

    <h3>🔹 Ejercicio 8: Modo Autocommit Apagado</h3>
    <p>Al desactivar autocommit, toda sentencia inicia una transacción implícita que requiere commit manual.</p>
    <pre><code>SET AUTOCOMMIT=0;

INSERT INTO alumnos ...;
/* No visible en secundaria aún */

COMMIT;
/* Ahora visible */</code></pre>

    <h3>🔹 Ejercicios 9 y 10: SAVEPOINT</h3>
    <p>Crear puntos de guardado intermedios para deshacer solo una parte de la transacción.</p>
    <pre><code>START TRANSACTION;
INSERT INTO alumnos ...; /* Acción 1 */

SAVEPOINT alumno;

INSERT INTO inscripciones ...; /* Acción 2 */

/* Deshacer solo Acción 2 */
ROLLBACK TO alumno;

/* Confirmar Acción 1 */
COMMIT;</code></pre>

    <br><hr>

    <!-- PRÁCTICA 12 -->
    <h2>⚙️ Práctica 12: Stored Procedures y Functions</h2>
    <p><b>BASE DE DATOS:</b> <code>AFATSE</code></p>

    <h3>🔹 Ejercicio 1</h3>
    <p>SP: <code>plan_lista_precios_actual</code>. Devuelve planes con su valor actual usando tabla temporal.</p>
    <pre><code>CREATE PROCEDURE `plan_lista_precios_actual`()
BEGIN
drop temporary table if exists valor_actual;

create temporary table valor_actual
(
select vp.`nom_plan`, max(vp.`fecha_desde_plan`) ult_fecha
from `valores_plan` vp
group by vp.`nom_plan`
);

select pc.`nom_plan`, pc.`modalidad`, vp.`valor_plan` valor_actual
from `plan_capacitacion` pc
inner join valor_actual va
on pc.`nom_plan`=va.nom_plan
inner join `valores_plan` vp
on va.`nom_plan`=vp.`nom_plan`
and va.ult_fecha=vp.`fecha_desde_plan`;

drop temporary table if exists valor_actual;
END;</code></pre>

    <h3>🔹 Ejercicio 2</h3>
    <p>SP: <code>plan_lista_precios_a_fecha</code>. Recibe una fecha parámetro.</p>
    <pre><code>CREATE PROCEDURE `plan_lista_precios_a_fecha`(IN fecha_hasta DATE)
BEGIN
drop temporary table if exists valor_actual;

create temporary table valor_actual
(
select vp.`nom_plan`, max(vp.`fecha_desde_plan`) ult_fecha
from `valores_plan` vp
where vp.`fecha_desde_plan`<=fecha_hasta
group by vp.`nom_plan`
);

select pc.`nom_plan`, pc.`modalidad`, vp.`valor_plan` valor_a_fecha
from `plan_capacitacion` pc
inner join valor_actual va
on pc.`nom_plan`=va.nom_plan
inner join `valores_plan` vp
on va.`nom_plan`=vp.`nom_plan`
and va.ult_fecha=vp.`fecha_desde_plan`;

drop temporary table if exists valor_actual;
END;</code></pre>

    <h3>🔹 Ejercicio 3</h3>
    <p>Reutilización: Modificar el SP 1 para que llame al SP 2 con la fecha actual.</p>
    <pre><code>DROP PROCEDURE `plan_lista_precios_actual`;

CREATE PROCEDURE `plan_lista_precios_actual`()
BEGIN
    call plan_lista_precios_a_fecha(CURRENT_DATE);
END;</code></pre>

    <h3>🔹 Ejercicio 6</h3>
    <p>SP con Parámetros OUT: <code>alumnos_pagos_deudas_a_fecha</code>.</p>
    <pre><code>CREATE PROCEDURE `alumnos_pagos_deudas_a_fecha`(IN fecha_limite DATE, IN dni_alumno
INTEGER(11), OUT pagado FLOAT(9,3), OUT cant_adeudado INTEGER(11))
BEGIN

select @pagado:=sum(cuo.`importe_pagado`)
from cuotas cuo
where cuo.dni=dni_alumno and cuo.`fecha_pago` is not null
and cuo.`fecha_emision`<=fecha_limite;

select @cant_adeudado:=count(*)
from cuotas cuo
where cuo.dni=dni_alumno and cuo.`fecha_pago` is null
and cuo.`fecha_emision`<=fecha_limite;

set pagado:=@pagado;
set cant_adeudado:=@cant_adeudado;

END;</code></pre>

    <h3>🔹 Ejercicio 7</h3>
    <p>Función: <code>alumnos_deudas_a_fecha</code>. Retorna cantidad de cuotas adeudadas.</p>
    <pre><code>CREATE FUNCTION `alumnos_deudas_a_fecha`(dni_alumno INTEGER(11), fecha_limite DATE)
RETURNS float(9,3)
BEGIN
declare cant_adeudado integer(11);

select count(*) into cant_adeudado
from cuotas cuo
where cuo.dni=dni_alumno and cuo.`fecha_pago` is null
and cuo.`fecha_emision`<=fecha_limite;

return cant_adeudado;
END;</code></pre>

    <h3>🔹 Ejercicio 8</h3>
    <p>SP Transaccional: <code>alumno_inscripcion</code>. Inscribe y genera cuota.</p>
    <pre><code>CREATE PROCEDURE `alumno_inscripcion`(IN dni_alumno INTEGER(11), IN plan CHAR(20), IN curso
INTEGER(11))
BEGIN
    start transaction;

    insert into inscripciones
    values (plan, curso,dni_alumno,CURRENT_DATE);

    insert into cuotas
    values (plan,curso,dni_alumno, year(adddate(CURRENT_DATE,interval 1 month)),
    month(adddate(CURRENT_DATE,interval 1 month)),CURRENT_DATE, null,null);
    
    commit;
END;</code></pre>

    <h3>🔹 Ejercicio 11</h3>
    <p>Modularización y Lógica de Negocio: <code>stock_movimiento</code>, <code>stock_ingreso</code>, <code>stock_egreso</code>.</p>
    <pre><code>/* SP Genérico de Movimiento */
CREATE PROCEDURE `stock_movimiento`(IN cod_mat CHAR(6), IN cant_movida INTEGER(11), OUT
stock INTEGER(11))
BEGIN
    declare url varchar(50);
    start transaction;
    
    /* Validar tipo de material */
    select url_descarga into url
    from materiales
    where cod_material=cod_mat;
    
    /* Solo actualizar stock si no es descargable (url is null) */
    if url is null then
        update materiales set cant_disponible=cant_disponible+cant_movida
        where cod_material=cod_mat;
    end if;
    
    /* Obtener stock resultante */
    select cant_disponible into stock
    from materiales
    where cod_material=cod_mat;
    
    /* Validar stock negativo */
    if stock >= 0 then
        commit;
    else
        rollback;
        /* Refrescar valor de stock tras rollback */
        select cant_disponible into stock
        from materiales
        where cod_material=cod_mat;
    end if;
END;

/* Wrappers */
CREATE PROCEDURE `stock_ingreso`(IN cod_mat CHAR(6), IN cant_movida INTEGER(11), OUT stock
INTEGER(11))
BEGIN
    call stock_movimiento(cod_mat,cant_movida,stock);
END;

CREATE PROCEDURE `stock_egreso`(IN cod_mat CHAR(6), IN cant_movida INTEGER(11), OUT stock
INTEGER(11))
BEGIN
    call stock_movimiento(cod_mat,(-1)*cant_movida,stock);
END;</code></pre>

    <h3>🔹 Ejercicio 12</h3>
    <p>SP: <code>alumno_anula_inscripcion</code>. Validar pagos antes de anular.</p>
    <pre><code>CREATE PROCEDURE `alumno_anula_inscripcion`(IN plan CHAR(20), IN curso INTEGER(11), IN
alumno INTEGER(11))
BEGIN
    declare cuotas_pagas integer(11);
    
    select count(*) into cuotas_pagas
    from cuotas
    where nom_plan=plan and nro_curso=curso and dni=alumno
    and fecha_pago is not null;
    
    if cuotas_pagas <= 0 then
        start transaction;
        delete from cuotas where nom_plan=plan and nro_curso=curso
        and dni=alumno and fecha_pago is null;
        
        delete from inscripciones where nom_plan=plan and nro_curso=curso and dni=alumno;
        commit;
    end if;
END;</code></pre>

</body>
</html>
