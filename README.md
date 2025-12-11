<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Laboratorio SQL Avanzado</title>
</head>
<body>

<h1 align="center">🛢️ Laboratorio Avanzado de SQL</h1>
<p align="center">
    <b>Guía Práctica Integral:</b> Transacciones, Stored Procedures, Funciones, Triggers y Control de Concurrencia (TCL).
</p>
<hr>

<!-- SECCIÓN 1: TRANSACCIONES BÁSICAS -->
<h2>📝 Parte 1: Transacciones y Modificación de Datos (DML)</h2>
<p>Ejercicios de actualización masiva, inserción lógica y limpieza de datos utilizando transacciones explícitas.</p>

<h3>🔹 Ejercicio 1: Actualización de Precios (+20%)</h3>
<p>Crear una nueva lista de precios para todos los planes a partir del 01/06/2009 con un 20% de aumento. Luego eliminar las filas agregadas.</p>
<pre><code>/* Verificar datos actuales */
select * from valores_plan order by fecha_desde_plan;

/* Insertar nuevos valores */
start transaction;
insert into valores_plan( `nom_plan`, `fecha_desde_plan`, `valor_plan`)
select val.nom_plan,'20160601', val.valor_plan*1.2
from valores_plan val
inner join
(
    select vp.nom_plan, max(vp.fecha_desde_plan) ult_fecha
    from valores_plan vp
    group by vp.nom_plan
) fechas
on val.nom_plan=fechas.nom_plan
and val.fecha_desde_plan=fechas.ult_fecha;
commit;

/* Eliminar lo agregado (Rollback lógico) */
start transaction;
Delete from valores_plan
where fecha_desde_plan ='20160601';
commit;</code></pre>

    <hr>

    <h3>🔹 Ejercicio 2: Aumento Condicional de Precios</h3>
    <p>Nueva lista a partir del 01/08/2009. Regla: Cursos < $90 aumentan 20%, el resto 12%.</p>
    <pre><code>start transaction;
insert into valores_plan( nom_plan, fecha_desde_plan, valor_plan)
select val.nom_plan,'20160801',
       case
           when val.valor_plan < 90 then val.valor_plan*1.2
           else val.valor_plan*1.12
       end
from valores_plan val
inner join
(
    select vp.nom_plan, max(vp.fecha_desde_plan) ult_fecha
    from valores_plan vp
    group by vp.nom_plan
) fechas
on val.nom_plan=fechas.nom_plan
and val.fecha_desde_plan=fechas.ult_fecha;
commit;</code></pre>

    <hr>

    <h3>🔹 Ejercicio 3: Duplicación de Plan (Marketing Presencial)</h3>
    <p>Crear "Marketing 1 Presen" basado en "Marketing 1", pero con costo +50% para el año 2014.</p>
    <pre><code>start transaction;

/* Copiar cabecera del plan */
insert into plan_capacitacion
select 'Marketing 1 Presen', desc_plan,hs,'presencial'
from `plan_capacitacion`
where nom_plan= 'Marketing 1';

/* Copiar temas */
insert into plan_temas
select 'Marketing 1 Presen', titulo,detalle
from plan_temas
where nom_plan= 'Marketing 1';

/* Copiar exámenes */
insert into `examenes`
select 'Marketing 1 Presen', nro_examen
from `examenes`
where nom_plan= 'Marketing 1';

/* Copiar temas de exámenes */
insert into `examenes_temas`
select 'Marketing 1 Presen',titulo,nro_examen
from `examenes_temas`
where nom_plan= 'Marketing 1';

/* Generar valores con aumento */
insert into `valores_plan`(`nom_plan`, `fecha_desde_plan`, `valor_plan`)
select 'Marketing 1 Presen',fecha_desde_plan,valor_plan*1.5
from `valores_plan`
where nom_plan= 'Marketing 1' and year(fecha_desde_plan)=2014;

/* #rollback; */
commit;</code></pre>

    <hr>

    <h3>🔹 Ejercicio 4: Reasignación de Supervisores</h3>
    <p>Cambiar el supervisor de instructores que dictan "Reparac PC Avanzada" en 2015 al instructor Franz Kafka (66-66666666-6).</p>
    <pre><code>/* Análisis previo de instructores y supervisores */
/* Select * from instructores; */

start transaction;
update  instructores i    
    inner join  cursos_instructores  ci
        on ci.cuil=i.cuil 
    inner join cursos c    
        on c.nom_plan=ci.nom_plan
set cuil_supervisor ='66-66666666-6'
where ci.nom_plan='Reparac PC Avanzada'
and year(c.fecha_ini)=2015;</code></pre>

    <hr>

    <h3>🔹 Ejercicio 5: Modificación de Horarios</h3>
    <p>Adelantar 1 hora los cursos de Franz Kafka del año 2015 que inician a las 16 hs.</p>
    <pre><code>start transaction;
update  cursos_horarios ch 
inner join   cursos_instructores ci on ch.nom_plan=ci.nom_plan
                              and ch.nro_curso=ci.nro_curso
inner join      cursos c on ci.nom_plan=c.nom_plan
                        and ci.nro_curso=c.nro_curso
set ch.hora_inicio=ADDTIME(ch.hora_inicio,-010000), 
ch.hora_fin=ADDTIME(ch.hora_fin,-010000)
where ci.cuil='66-66666666-6' and ch.`hora_inicio`='160000' and 
year(c.fecha_ini)=2015;
commit;</code></pre>

    <hr>

    <h3>🔹 Ejercicio 6: Limpieza de Exámenes (Tablas Temporales/CTE)</h3>
    <p>Eliminar exámenes con promedio < 5.5 y los temas asociados. Se demuestra uso de Tablas Temporales y CTE.</p>
    <pre><code>start transaction;
drop temporary table if exists exa_elim;

/* Estrategia 1: Tabla Temporal */
create temporary table exa_elim
(
    select ev.nom_plan,ev.nro_examen, AVG(ev.nota) promedio
    from evaluaciones ev
    group by ev.nom_plan,ev.nro_examen
    having promedio < 5.5
);

/* Estrategia 2: CTE (Comentada como ejemplo) */
/* WITH exa AS 
(SELECT nom_plan,nro_examen, AVG(nota) promedio
from evaluaciones 
group by nom_plan, nro_examen
having promedio < 5.5
);
*/

/* Eliminación Multi-tabla (Delete Join) */
delete ev,et
from evaluaciones ev
inner join exa_elim
    on ev.`nom_plan`=exa_elim.nom_plan and ev.`nro_examen`=exa_elim.nro_examen
inner join `examenes_temas` et 
    on et.`nom_plan`=exa_elim.nom_plan and et.`nro_examen`=exa_elim.nro_examen;

/* Eliminar exámenes padres */
delete ex
from exa_elim
inner join examenes ex on ex.nom_plan=exa_elim.nom_plan              
and ex.nro_examen=exa_elim.nro_examen;

commit;</code></pre>

    <hr>

    <h3>🔹 Ejercicio 7: Depuración de Morosos</h3>
    <p>Eliminar inscripciones de 2015 para alumnos que adeuden cuotas de 2014.</p>
    <pre><code>start transaction;

/* Opción A: Usando tabla temporal */
drop temporary table if exists deudores;
create temporary table deudores
(
    select distinct dni 
    from cuotas 
    where fecha_pago is null and anio= 2014
);

delete insc
from inscripciones insc inner join deudores 
on insc.dni=deudores.dni
where year(insc.fecha_inscripcion)=2015;

/* Opción B: Subquery directa con fechas dinámicas */
/*
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
*/
commit;</code></pre>

    <br><br>

    <!-- SECCIÓN 2: PROGRAMACIÓN -->
    <h2>⚙️ Parte 2: Stored Procedures y Funciones (Práctica 10)</h2>
    <p>Gestión de lógica de negocio encapsulada en la base de datos.</p>

    <h3>🔹 1. SP: Lista de Precios Actual</h3>
    <pre><code>delimiter //   
CREATE PROCEDURE plan_lista_precios_actual() 
begin 
 drop temporary table if exists valor_actual; 
 create temporary table valor_actual 
 ( 
     select vp.nom_plan, max(vp.fecha_desde_plan) ult_fecha 
     from valores_plan vp 
     where vp.fecha_desde_plan <= CURRENT_DATE 
     group by vp.nom_plan 
 ); 
 
 select pc.nom_plan, pc.modalidad, vp.valor_plan  valor_actual 
 from plan_capacitacion pc 
 inner join valor_actual va on  pc.nom_plan=va.nom_plan 
 inner join valores_plan vp on  va.nom_plan=vp.nom_plan 
                            and  va.ult_fecha=vp.fecha_desde_plan; 
 
 drop temporary table if exists valor_actual; 
end // 
delimiter ;

/* Invocación */
call plan_lista_precios_actual();</code></pre>

    <hr>

    <h3>🔹 2. SP: Lista de Precios a Fecha</h3>
    <pre><code>delimiter // 
CREATE PROCEDURE plan_lista_precios_a_fecha(IN fecha_hasta DATE) 
    NOT DETERMINISTIC 
    SQL SECURITY DEFINER 
BEGIN 
    drop temporary table if exists valor_a_fecha; 
    create temporary table valor_a_fecha 
    ( 
     select vp.nom_plan, max(vp.fecha_desde_plan) ult_a_fecha 
     from valores_plan vp 
     where vp.fecha_desde_plan<=fecha_hasta 
     group by vp.nom_plan 
    ); 
     
    select pc.nom_plan, pc.modalidad, vp.valor_plan valor_fecha 
    from plan_capacitacion pc 
    inner join valor_a_fecha va on  pc.nom_plan=va.nom_plan 
    inner join valores_plan vp on  va.nom_plan =vp.nom_plan 
                               and va.ult_a_fecha=vp.fecha_desde_plan; 
     
    drop temporary table if exists valor_a_fecha; 
END // 
delimiter ;

/* Invocación */
call plan_lista_precios_a_fecha('2014-02-01');</code></pre>

    <hr>

    <h3>🔹 3. Reutilización de SPs</h3>
    <p>Modificar el SP 1 para invocar internamente al SP 2.</p>
    <pre><code>DROP PROCEDURE plan_lista_precios_actual; 
delimiter // 
CREATE PROCEDURE plan_lista_precios_actual() 
    NOT DETERMINISTIC 
    SQL SECURITY DEFINER 
BEGIN 
    call plan_lista_precios_a_fecha(CURRENT_DATE); 
END // 
delimiter ;</code></pre>

    <hr>

    <h3>🔹 4. Función: Obtener Valor de Plan</h3>
    <pre><code>SET GLOBAL log_bin_trust_function_creators = 1; 
delimiter // 
CREATE FUNCTION plan_valor (nom varchar(20), fecha DATE) 
RETURNS float(9,3) 
    NOT DETERMINISTIC 
    SQL SECURITY DEFINER 
BEGIN 
    declare valor  float(9,3); 
    select valor_plan into valor 
    from valores_plan 
    where nom_plan = nom 
    and fecha_desde_plan = fecha; 
    return valor; 
end // 
delimiter ; 

/* Prueba */
select  plan_valor("Marketing 3",'2015-01-02');</code></pre>

    <hr>

    <h3>🔹 5. Integración SP + Función</h3>
    <p>Modificar el SP del punto 2 para usar la función creada en el punto 4.</p>
    <pre><code>DROP PROCEDURE plan_lista_precios_a_fecha; 
delimiter // 
CREATE PROCEDURE plan_lista_precios_a_fecha(in fecha date) 
    NOT DETERMINISTIC 
    SQL SECURITY DEFINER 
BEGIN 
    /* Nota: Esto simplifica la lógica mostrando solo un ejemplo */
    select  plan_valor("Marketing 3",fecha); 
END // 
delimiter ;</code></pre>

    <hr>

    <h3>🔹 6. SP con Parámetros de Salida (OUT)</h3>
    <p>Calcula pagos realizados y deudas de un alumno a una fecha.</p>
    <pre><code>delimiter // 
CREATE PROCEDURE alumnos_pagos_deudas_a_fecha 
(IN fecha_limite DATE, IN dni_alumno INTEGER(11),  
  OUT pagado FLOAT(9,3), OUT cant_adeudado INTEGER(11)) 
 BEGIN 
    select @pagado:=sum(cuo.importe_pagado) 
    from cuotas cuo 
    where cuo.dni=dni_alumno and cuo.fecha_pago is not null 
          and cuo.fecha_emision<=fecha_limite; 
     
    select @cant_adeudado:=count(*) 
    from cuotas cuo 
    where cuo.dni=dni_alumno and cuo.fecha_pago is null 
          and cuo.fecha_emision<=fecha_limite; 
     
    set pagado:=@pagado; 
    set cant_adeudado:=@cant_adeudado; 
END // 
delimiter ; 

/* Invocación y lectura de variables */
call alumnos_pagos_deudas_a_fecha (CURRENT_DATE,24242424,@pagado,@cant_adeudado); 
select @pagado,  @cant_adeudado;</code></pre>

    <hr>

    <h3>🔹 7. Función: Cálculo de Deudas</h3>
    <pre><code>DELIMITER // 
CREATE FUNCTION alumnos_deudas_a_fecha(dni_alumno INTEGER(11), fecha_limite DATE) 
    RETURNS integer(11) 
    NOT DETERMINISTIC 
    SQL SECURITY DEFINER 
BEGIN 
    declare cant_adeudado integer(11); 
    select count(*) into cant_adeudado 
    from cuotas cuo 
    where cuo.dni=dni_alumno and cuo.`fecha_pago` is null 
          and cuo.fecha_emision<=fecha_limite; 
    return cant_adeudado; 
END // 
DELIMITER ; 

SELECT alumnos_deudas_a_fecha (24242424, current_date);</code></pre>

    <hr>

    <h3>🔹 8. SP: Inscripción de Alumnos</h3>
    <p>Inscribe alumno, actualiza contador de curso y genera primera cuota.</p>
    <pre><code>delimiter // 
CREATE PROCEDURE alumno_inscripcion (IN dni_alumno INTEGER(11), IN plan CHAR(20), IN curso INTEGER(11)) 
    NOT DETERMINISTIC 
    SQL SECURITY DEFINER 
BEGIN 
    start transaction; 
    insert into inscripciones 
    values (plan, curso,dni_alumno,CURRENT_DATE); 
      
    /* Modifico la cantidad de inscriptos al curso */
    update cursos set cant_inscriptos= cant_inscriptos +1 
    where nom_plan = plan  and nro_curso =curso; 

    insert into cuotas 
    values (plan,curso,dni_alumno,    
            year(adddate(CURRENT_DATE,interval 1 month)), 
            month(adddate(CURRENT_DATE,interval 1 month)),CURRENT_DATE, null,null); 
    commit; 
END // 
delimiter ;</code></pre>

    <hr>

    <h3>🔹 9. SP: Inscripción con Validación</h3>
    <p>Verifica si el alumno ya está inscripto antes de procesar.</p>
    <pre><code>delimiter // 
CREATE PROCEDURE alumno_inscrip_con_validacion_2 (IN dni_alumno INTEGER(11), IN plan CHAR(20), IN curso INTEGER(11)) 
    NOT DETERMINISTIC 
    SQL SECURITY DEFINER 
BEGIN 
    declare insc INTEGER(11); 
    start transaction; 
    select count(dni) into @insc 
    from inscripciones 
    where dni=dni_alumno and nom_plan =plan and nro_curso=curso ; 

    if @insc = 0 then 
        insert into inscripciones 
        values (plan, curso,dni_alumno,CURRENT_DATE); 
         
        update cursos set cant_inscriptos= cant_inscriptos +1 
        where nom_plan = plan  and nro_curso =curso; 
    
        insert into cuotas 
        values (plan,curso,dni_alumno, year(adddate(CURRENT_DATE,interval 1 month)), 
                month(adddate(CURRENT_DATE,interval 1 month)),CURRENT_DATE, null,null);          
        commit; 
    else  
        rollback; 
    end if; 
END // 
delimiter ;</code></pre>

    <hr>

    <h3>🔹 10. SP: Validación de Cupos (Rollback Condicional)</h3>
    <pre><code>delimiter // 
CREATE PROCEDURE alumno_inscrip_con_validacion_3 (IN dni_alumno INTEGER(11), IN plan CHAR(20), IN curso INTEGER(11)) 
    NOT DETERMINISTIC 
    SQL SECURITY DEFINER 
BEGIN 
    declare insc INTEGER(11); 
    start transaction; 
    /* ... (Lógica de inserción previa igual al ejercicio 9) ... */
    
    /* Nueva Validación de Cupo */
    If cant_inscriptos <= cupo then 
         commit; 
    else  
         rollback; 
    end if;    
END // 
delimiter ;</code></pre>

    <hr>

    <h3>🔹 11. Gestión de Stock (Modularización)</h3>
    <p>Procedimientos `stock_ingreso` y `stock_egreso` reutilizando `stock_movimiento`.</p>
    <pre><code>delimiter // 
CREATE PROCEDURE `stock_movimiento`(IN cod_mat CHAR(6), IN cant_movida INTEGER(11), OUT stock INTEGER(11)) 
    NOT DETERMINISTIC 
    SQL SECURITY DEFINER 
BEGIN 
     declare url varchar(50); 
     start transaction; 
     
     /* Verificar si es físico o digital */
     select url_descarga into url from materiales where cod_material=cod_mat; 
      
     if url is null then 
        update materiales set cant_disponible=cant_disponible+cant_movida 
        where cod_material=cod_mat; 
     end if; 
      
     select cant_disponible into stock from materiales where cod_material=cod_mat; 

     /* Validación de stock negativo */
     if stock>=0 then 
        commit; 
     else 
        rollback; 
     end if; 

     select cant_disponible into stock from materiales where cod_material=cod_mat; 
END // 

CREATE PROCEDURE `stock_ingreso`(IN cod_mat CHAR(6), IN cant_movida INTEGER(11), OUT stock INTEGER(11)) 
BEGIN 
     call stock_movimiento(cod_mat,cant_movida,stock); 
END // 

CREATE PROCEDURE `stock_egreso`(IN cod_mat CHAR(6), IN cant_movida INTEGER(11), OUT stock INTEGER(11)) 
BEGIN 
     call stock_movimiento(cod_mat,(-1)*cant_movida,stock); 
END// 
delimiter ;</code></pre>

    <hr>

    <h3>🔹 12. SP: Anulación de Inscripción</h3>
    <p>Elimina inscripción solo si el alumno no ha pagado cuotas. Limpia cuotas impagas.</p>
    <pre><code>delimiter // 
CREATE PROCEDURE alumno_anula_inscripcion (IN plan CHAR(20), IN curso INTEGER(11), IN alumno INTEGER(11)) 
    NOT DETERMINISTIC 
    SQL SECURITY DEFINER 
BEGIN 
     declare cuotas_pagas integer(11); 
      
     select count(*) into cuotas_pagas 
     from cuotas 
     where nom_plan=plan and nro_curso=curso and dni=alumno 
           and fecha_pago is not null; 

     if cuotas_pagas<=0 then 
        start transaction; 
        delete from cuotas 
        where nom_plan=plan and nro_curso=curso 
              and dni=alumno and fecha_pago is null; 

        update cursos set cant_inscriptos= cant_inscriptos -1 
        where nom_plan ="Marketing 1" and nro_curso=1; 

        delete from inscripciones 
        where nom_plan=plan and nro_curso=curso and dni=alumno; 

        commit; 
     end if; 
END// 
delimiter ;</code></pre>

    <br><br>

    <!-- SECCIÓN 3: TRIGGERS -->
    <h2>⚡ Parte 3: Triggers (Práctica 11)</h2>
    <p>Automatización de auditoría y lógica reactiva.</p>

    <h3>🔹 1. Histórico de Alumnos</h3>
    <p>Registrar cambios (Insert/Update) en tabla `alumnos_historico`.</p>
    <pre><code>/* Trigger INSERT */
delimiter // 
CREATE TRIGGER  alumnos_after_ins_tr  AFTER INSERT ON alumnos 
  FOR EACH ROW 
BEGIN 
     insert into alumnos_historico 
     values (new.dni,CURRENT_TIMESTAMP,new.nombre,new.apellido, 
             new.tel,new.email,new.direccion,CURRENT_USER); 
END// 
delimiter ; 

/* Trigger UPDATE */
delimiter // 
CREATE TRIGGER  alumnos_Aftet_upd_tr AFTER  UPDATE ON alumnos 
  FOR EACH ROW 
BEGIN 
     insert into alumnos_historico 
     values (new.dni,CURRENT_TIMESTAMP,new.nombre,new.apellido, 
             new.tel,new.email,new.direccion,CURRENT_USER) ; 
END// 
delimiter ;</code></pre>

    <hr>

    <h3>🔹 2. Auditoría de Stock</h3>
    <p>Registrar movimientos en `stock_movimientos` al insertar o actualizar materiales.</p>
    <pre><code>/* Trigger INSERT */
delimiter // 
CREATE TRIGGER `materiales_after_ins_tr` AFTER INSERT ON `materiales` 
  FOR EACH ROW 
BEGIN 
     if new.cant_disponible is not null then 
        insert into stock_movimientos( cod_material, cantidad_movida, cantidad_restante, usuario_movimiento) 
        values (new.cod_material,new.cant_disponible,new.cant_disponible,CURRENT_USER); 
     end if; 
END // 
delimiter ; 

/* Trigger UPDATE (Calculando diferencial) */
DELIMITER // 
CREATE TRIGGER `materiales_before2_upd_tr` BEFORE UPDATE ON `materiales` 
  FOR EACH ROW 
BEGIN 
   if new.cant_disponible is not null then 
     set @cant_movida=new.cant_disponible-old.cant_disponible; 
     if @cant_movida!=0 then 
          insert into stock_movimientos( cod_material, cantidad_movida, cantidad_restante, usuario_movimiento) 
          values (new.cod_material,@cant_movida, new.cant_disponible,CURRENT_USER); 
     end if; 
   end if; 
END// 
DELIMITER ;</code></pre>

    <hr>

    <h3>🔹 3. Mantenimiento Automático de 'cant_inscriptos'</h3>
    <p>Actualizar contador en tabla `cursos` al insertar o borrar en `inscripciones`.</p>
    <pre><code>/* Al Inscribir */
DELIMITER // 
CREATE TRIGGER `inscripciones_after_ins_tr` AFTER INSERT ON `inscripciones` 
  FOR EACH ROW 
BEGIN 
     update cursos 
     set cant_inscriptos=cant_inscriptos+1 
     where nom_plan=new.nom_plan and nro_curso=new.nro_curso; 
END// 
DELIMITER ; 

/* Al Borrar Inscripción */
DELIMITER // 
CREATE TRIGGER `inscripciones_after_del_tr` AFTER DELETE ON `inscripciones` 
  FOR EACH ROW 
BEGIN 
     update cursos 
     set cant_inscriptos=cant_inscriptos-1 
     where nom_plan=old.nom_plan and nro_curso=old.nro_curso; 
END// 
DELIMITER ;</code></pre>

    <hr>

    <h3>🔹 4. Auditoría de Usuario (Before Insert)</h3>
    <p>Asignar automáticamente el `usuario_alta` antes de insertar un precio.</p>
    <pre><code>DELIMITER // 
CREATE TRIGGER valores_plan_before_ins_tr BEFORE INSERT ON valores_plan 
FOR EACH ROW 
BEGIN 
    set new.usuario_alta=CURRENT_USER; 
END// 
DELIMITER ;</code></pre>

    <br><br>

    <!-- SECCIÓN 4: TCL & ISOLATION -->
    <h2>🔐 Parte 4: TCL y Niveles de Aislamiento (Práctica 12)</h2>
    <p>Commit, Rollback, Savepoints y visibilidad de datos entre sesiones.</p>

    <h3>🔹 1-7. Comportamiento Básico (Commit/Rollback)</h3>
    <pre><code>set autocommit=1; /* Default */

/* Caso A: Rollback */
start transaction; 
insert into alumnos ... ;
insert into inscripciones ... ;
/* En otra sesión NO se ven los datos aun */
ROLLBACK; 
/* Los datos desaparecen */

/* Caso B: Commit */
start transaction;
/* ... inserts ... */
COMMIT;
/* Los datos son permanentes y visibles en todas las sesiones */</code></pre>

    <hr>

    <h3>🔹 8. Modo Autocommit = 0</h3>
    <p>Al desactivar el autocommit, todo se comporta como una transacción implícita hasta que se hace commit explícito.</p>
    <pre><code>set autocommit = 0;
/* Inserts... */
/* Solo visibles en sesión actual */
COMMIT; 
/* Ahora visibles globalmente */</code></pre>

    <hr>

    <h3>🔹 9-10. Uso de SAVEPOINT</h3>
    <p>Permite deshacer cambios parcialmente dentro de una transacción.</p>
    <pre><code>START TRANSACTION; 
insert into alumnos ... ; /* (1) */

SAVEPOINT ALUMNO; 

insert into inscripciones ... ; /* (2) */

ROLLBACK TO ALUMNO; 
/* Se deshace (2) pero se mantiene (1) pendiente */

COMMIT; 
/* Se confirma solo (1) */</code></pre>

    <hr>

    <h3>🔹 11-12. Niveles de Aislamiento</h3>
    <p>Configuración de visibilidad de datos sucios o fantasmas.</p>

    <pre><code>/* Ver nivel actual */
Select @@transaction_isolation;

/* 1. READ UNCOMMITTED (Lectura Sucia permitida) */
SET SESSION TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
/* Permite ver datos de transacciones NO commiteadas de otras sesiones */

/* 2. READ COMMITTED (Evita Lectura Sucia) */
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;
/* Solo ve datos confirmados con COMMIT */

/* 3. REPEATABLE READ (Default MySQL) */
/* Garantiza consistencia en lecturas consecutivas dentro de la misma tx */</code></pre>

    <hr>
    <p align="center"><i>Documentación generada automáticamente para repositorio académico.</i></p>

</body>
</html>
