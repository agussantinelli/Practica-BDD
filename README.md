
<body>

 <h1 align="center">🛢️ Laboratorio SQL: Prácticas 9 a 12</h1>
    <p align="center">Guía con código SQL profundamente tabulado para máxima legibilidad.</p>
    <hr>
    <br>

<h2>🚀 Práctica 9: INSERT SELECT, UPDATE y DELETE con JOINS</h2>
    <p><strong>BASE DE DATOS:</strong> <code>AFATSE</code></p>
    <br>

<h3>🔹 Ejercicio 1</h3>
    <p>Crear una nueva lista de precios para todos los planes de capacitación, a partir del 01/06/2009 con un 20 por ciento más que su último valor. Eliminar las filas agregadas.</p>
    
<pre><code>
    /* 1. Verificar tabla original */
    SELECT 
        * FROM 
        valores_plan
    ORDER BY 
        fecha_desde_plan;

    /* 2. Insertar nuevos precios */
    START TRANSACTION;

        INSERT INTO valores_plan(
            `nom_plan`, 
            `fecha_desde_plan`, 
            `valor_plan`
        )
        SELECT 
            val.`nom_plan`, 
            '20160601', 
            val.`valor_plan` * 1.2
        FROM 
            valores_plan val
        INNER JOIN (
            SELECT 
                vp.`nom_plan`, 
                MAX(vp.`fecha_desde_plan`) ult_fecha
            FROM 
                valores_plan vp
            GROUP BY 
                vp.`nom_plan`
        ) fechas
            ON val.`nom_plan` = fechas.`nom_plan`
            AND val.`fecha_desde_plan` = fechas.ult_fecha;

    COMMIT;

    /* 3. Limpieza */
    START TRANSACTION;
    
        DELETE FROM 
            valores_plan
        WHERE 
            fecha_desde_plan = '20160601';
            
    COMMIT;
</code></pre>
    
<br><hr><br>

<h3>🔹 Ejercicio 2</h3>
    <p>Crear una nueva lista de precios para todos los planes de capacitación, a partir del 01/08/2009, con la siguiente regla: Los cursos cuyo último valor sea menor a $90 aumentarlos en un 20% al resto aumentarlos un 12%.</p>
    
<pre><code>
    START TRANSACTION;

        INSERT INTO valores_plan(
            `nom_plan`, 
            `fecha_desde_plan`, 
            `valor_plan`
        )
        SELECT 
            val.`nom_plan`, 
            '20160801',
            CASE
                WHEN val.`valor_plan` < 90 THEN val.`valor_plan` * 1.2
                ELSE val.`valor_plan` * 1.12
            END
        FROM 
            valores_plan val
        INNER JOIN (
            SELECT 
                vp.`nom_plan`, 
                MAX(vp.`fecha_desde_plan`) ult_fecha
            FROM 
                valores_plan vp
            GROUP BY 
                vp.`nom_plan`
        ) fechas
            ON val.`nom_plan` = fechas.`nom_plan`
            AND val.`fecha_desde_plan` = fechas.ult_fecha;

    COMMIT;
    </code></pre>

<br><hr><br>

<h3>🔹 Ejercicio 3</h3>
    <p>Crear un nuevo plan: "Marketing 1 Presen". Con los mismos datos que el plan "Marketing 1" pero con modalidad presencial. Este plan tendrá los mismos temas, exámenes y materiales que Marketing 1 pero con un costo un 50% superior.</p>
    
<pre><code>
    START TRANSACTION;

        -- 1. Copiar Cabecera del plan
        INSERT INTO plan_capacitacion
        SELECT 
            'Marketing 1 Presen', 
            desc_plan, 
            hs, 
            'presencial'
        FROM 
            `plan_capacitacion`
        WHERE 
            nom_plan = 'Marketing 1';

        -- 2. Copiar Temas
        INSERT INTO plan_temas
        SELECT 
            'Marketing 1 Presen', 
            titulo, 
            detalle
        FROM 
            plan_temas
        WHERE 
            nom_plan = 'Marketing 1';

        -- 3. Copiar Exámenes
        INSERT INTO `examenes`
        SELECT 
            'Marketing 1 Presen', 
            nro_examen
        FROM 
            `examenes`
        WHERE 
            nom_plan = 'Marketing 1';

        -- 4. Copiar Temas de Exámenes
        INSERT INTO `examenes_temas`
        SELECT 
            'Marketing 1 Presen', 
            titulo, 
            nro_examen
        FROM 
            `examenes_temas`
        WHERE 
            nom_plan = 'Marketing 1';

        -- 5. Generar Valores con 50% de aumento
        INSERT INTO `valores_plan`(
            `nom_plan`, 
            `fecha_desde_plan`, 
            `valor_plan`
        )
        SELECT 
            'Marketing 1 Presen', 
            fecha_desde_plan, 
            valor_plan * 1.5
        FROM 
            `valores_plan`
        WHERE 
            nom_plan = 'Marketing 1' 
            AND YEAR(fecha_desde_plan) = 2014;

    COMMIT;
   </code></pre>

 <br><hr><br>

 <h3>🔹 Ejercicio 4</h3>
    <p>Cambiar el supervisor de aquellos instructores que dictan "Reparac PC Avanzada" este año (2015) a 66-66666666-6 (Franz Kafka).</p>
    
 <pre><code>
    START TRANSACTION;

        UPDATE 
            instructores i
        INNER JOIN cursos_instructores ci 
            ON ci.`cuil` = i.`cuil`
        INNER JOIN cursos c 
            ON c.`nom_plan` = ci.`nom_plan`
        
        SET 
            i.`cuil_supervisor` = '66-66666666-6'
        
        WHERE 
            ci.`nom_plan` = 'Reparac PC Avanzada'
            AND YEAR(c.`fecha_ini`) = 2015;

    COMMIT;
    </code></pre>

 <br><hr><br>

 <h3>🔹 Ejercicio 5</h3>
    <p>Cambiar el horario de los cursos que dicta este año (2015) Franz Kafka (cuil 66-66666666-6) desde las 16 hs. Moverlos una hora más temprano.</p>
    
  <pre><code>
    START TRANSACTION;

        UPDATE 
            cursos_horarios ch
        INNER JOIN cursos_instructores ci 
            ON ch.`nom_plan` = ci.`nom_plan` 
            AND ch.`nro_curso` = ci.`nro_curso`
        INNER JOIN cursos c 
            ON ci.`nom_plan` = c.`nom_plan` 
            AND ci.`nro_curso` = c.`nro_curso`
        
        SET 
            ch.`hora_inicio` = ADDTIME(ch.`hora_inicio`, -010000),
            ch.`hora_fin` = ADDTIME(ch.`hora_fin`, -010000)
        
        WHERE 
            ci.`cuil` = '66-66666666-6'
            AND ch.`hora_inicio` = '160000'
            AND YEAR(c.`fecha_ini`) = 2015;

    COMMIT;
 </code></pre>

  <br><hr><br>

  <h3>🔹 Ejercicio 6</h3>
    <p>Eliminar los exámenes donde el promedio general de las evaluaciones sea menor a 5.5. Eliminar también los temas que sólo se evalúan en esos exámenes.</p>
    
 <pre><code>
    START TRANSACTION;

        /* 1. Identificar exámenes a eliminar en tabla temporal */
        DROP TEMPORARY TABLE IF EXISTS exa_elim;

        CREATE TEMPORARY TABLE exa_elim
        (
            SELECT 
                ev.`nom_plan`, 
                ev.`nro_examen`, 
                AVG(ev.`nota`) promedio
            FROM 
                evaluaciones ev
            GROUP BY 
                ev.`nom_plan`, 
                ev.`nro_examen`
            HAVING 
                promedio < 5.5
        );

        /* 2. Eliminar evaluaciones y temas asociados (DELETE multitabla) */
        DELETE 
            ev, et
        FROM 
            evaluaciones ev
        INNER JOIN exa_elim 
            ON ev.`nom_plan` = exa_elim.`nom_plan` 
            AND ev.`nro_examen` = exa_elim.`nro_examen`
        INNER JOIN `examenes_temas` et 
            ON et.`nom_plan` = exa_elim.`nom_plan` 
            AND et.`nro_examen` = exa_elim.`nro_examen`;

        /* 3. Eliminar el examen padre */
        DELETE 
            ex
        FROM 
            exa_elim
        INNER JOIN `examenes` ex 
            ON ex.`nom_plan` = exa_elim.`nom_plan` 
            AND ex.`nro_examen` = exa_elim.`nro_examen`;

    COMMIT;
    </code></pre>

  <br><hr><br>

 <h3>🔹 Ejercicio 7</h3>
    <p>Eliminar las inscripciones a los cursos de este año (2015) de los alumnos que adeuden cuotas impagas del año pasado (2014).</p>
    
 <pre><code>
    START TRANSACTION;

        /* Estrategia: Usar tabla temporal para aislar los deudores primero */
        DROP TEMPORARY TABLE IF EXISTS deudores;

        CREATE TEMPORARY TABLE deudores
        (
            SELECT DISTINCT 
                dni
            FROM 
                cuotas
            WHERE 
                fecha_pago IS NULL 
                AND anio = 2014
        );

        DELETE 
            insc
        FROM 
            inscripciones insc
        INNER JOIN deudores 
            ON insc.`dni` = deudores.`dni`
        WHERE 
            YEAR(insc.`fecha_inscripcion`) = 2015;

    COMMIT;
 </code></pre>

    <br><br><br>
    <hr>
    <hr>
    <br><br><br>

    <!-- PRÁCTICA 10 -->
    <h2>🔐 Práctica 10: DCL (Seguridad)</h2>
    <p><strong>BASE DE DATOS:</strong> <code>AGENCIA_PERSONAL</code></p>
    <br>

    <h3>🔹 Ejercicios Básicos: Usuario y Contraseña</h3>
    
    <pre><code>
    /* 1. Crear Usuario */
    CREATE USER 
        usuario@localhost 
    IDENTIFIED BY 
        'entre';

    /* 2. Cambiar Clave */
    SET PASSWORD FOR 
        'usuario'@'localhost' = PASSWORD('entrar');
    </code></pre>

    <br><hr><br>

    <h3>🔹 Gestión de Permisos</h3>
    
    <pre><code>
    /* 3. Permiso SELECT global */
    GRANT SELECT 
    ON 
        agencia_personal.* TO 
        usuario@localhost;

    /* 4. Permisos de Modificación en tabla específica */
    GRANT UPDATE ON agencia_personal.personas TO usuario@localhost;
    GRANT INSERT ON agencia_personal.personas TO usuario@localhost;
    GRANT DELETE ON agencia_personal.personas TO usuario@localhost;

    /* 5. Revocar permisos */
    REVOKE ALL PRIVILEGES 
    ON 
        agencia_personal.* FROM 
        usuario@localhost;
    </code></pre>

    <br><hr><br>

    <h3>🔹 Seguridad con Vistas</h3>
    
    <pre><code>
    GRANT UPDATE ON agencia_personal.vw_contratos TO usuario@localhost;
    GRANT INSERT ON agencia_personal.vw_contratos TO usuario@localhost;
    GRANT DELETE ON agencia_personal.vw_contratos TO usuario@localhost;
    </code></pre>

    <br><br><br>
    <hr>
    <hr>
    <br><br><br>

    <!-- PRÁCTICA 11 -->
    <h2>⚡ Práctica 11: TCL (Control de Transacciones)</h2>
    <p><strong>BASE DE DATOS:</strong> <code>AFATSE</code></p>
    <br>

    <h3>🔹 1. Autocommit Activado</h3>
    <pre><code>
    SET AUTOCOMMIT = 1;
    
    SELECT @@autocommit;
    </code></pre>

    <br><hr><br>

    <h3>🔹 2 y 3. Comportamiento sin Transacción Explícita</h3>
    <pre><code>
    /* INSERTAR */
    INSERT INTO alumnos (
        dni, nombre, apellido, tel, email, direccion
    )
    VALUES (
        16817618, 'Clotilde', 'Diez', 11111111, 'cloti@yahoo.com', 'Rioja 2030'
    );

    /* ELIMINAR */
    DELETE FROM 
        alumnos 
    WHERE 
        dni = 16817618;

    /* Resultado: Los cambios son visibles inmediatamente en otras conexiones. */
    </code></pre>

    <br><hr><br>

    <h3>🔹 4 y 5. START TRANSACTION con ROLLBACK / COMMIT</h3>
    <pre><code>
    /* CASO ROLLBACK (Deshacer) */
    START TRANSACTION;
        INSERT INTO alumnos ...; 
        -- En otra sesión NO se ve.
    ROLLBACK; 
    -- El alumno desaparece.

    /* CASO COMMIT (Confirmar) */
    START TRANSACTION;
        INSERT INTO alumnos ...;
        -- En otra sesión NO se ve.
    COMMIT;
    -- Ahora el alumno ES visible para todos.
    </code></pre>

    <br><hr><br>

    <h3>🔹 8. Modo Autocommit = 0</h3>
    <pre><code>
    SET AUTOCOMMIT = 0;

    INSERT INTO alumnos ...;
    -- No es visible fuera de esta sesión.

    COMMIT;
    -- Se confirma y se hace visible.
    </code></pre>

    <br><hr><br>

    <h3>🔹 9 y 10. Uso de SAVEPOINT</h3>
    <pre><code>
    START TRANSACTION;

        /* Paso 1: Insertar alumno */
        INSERT INTO alumnos (dni, ...) VALUES (16817618, ...);

        /* Paso 2: Marcar punto */
        SAVEPOINT alumno;

        /* Paso 3: Insertar inscripción */
        INSERT INTO inscripciones ...;

        /* Paso 4: Deshacer solo la inscripción (volver al punto 'alumno') */
        ROLLBACK TO alumno;

        /* Paso 5: Confirmar la transacción */
        COMMIT;

    /* Resultado: El alumno queda guardado, la inscripción NO. */
    </code></pre>

    <br><br><br>
    <hr>
    <hr>
    <br><br><br>

    <!-- PRÁCTICA 12 -->
    <h2>⚙️ Práctica 12: Stored Procedures y Functions</h2>
    <p><strong>BASE DE DATOS:</strong> <code>AFATSE</code></p>
    <br>

    <h3>🔹 Ejercicio 1: SP Básico con Tabla Temporal</h3>
    
    <pre><code>
    DELIMITER //

    CREATE PROCEDURE plan_lista_precios_actual()
    BEGIN
        DROP TEMPORARY TABLE IF EXISTS valor_actual;
        
        CREATE TEMPORARY TABLE valor_actual
        (
            SELECT 
                vp.`nom_plan`, 
                MAX(vp.`fecha_desde_plan`) ult_fecha
            FROM 
                `valores_plan` vp
            WHERE 
                vp.`fecha_desde_plan` <= CURRENT_DATE
            GROUP BY 
                vp.`nom_plan`
        );
        
        SELECT 
            pc.`nom_plan`, 
            pc.`modalidad`, 
            vp.`valor_plan` valor_actual
        FROM 
            `plan_capacitacion` pc
        INNER JOIN valor_actual va 
            ON pc.`nom_plan` = va.`nom_plan`
        INNER JOIN `valores_plan` vp 
            ON va.`nom_plan` = vp.`nom_plan`
            AND va.ult_fecha = vp.`fecha_desde_plan`;
        
        DROP TEMPORARY TABLE IF EXISTS valor_actual;
    END //

    DELIMITER ;
    </code></pre>

    <br><hr><br>

    <h3>🔹 Ejercicio 2 y 3: SP con Parámetros y Reutilización</h3>
    
    <pre><code>
    /* Crear SP con parámetro fecha */
    DELIMITER //

    CREATE PROCEDURE plan_lista_precios_a_fecha(IN fecha_hasta DATE)
    BEGIN
        DROP TEMPORARY TABLE IF EXISTS valor_a_fecha;
        
        CREATE TEMPORARY TABLE valor_a_fecha
        (
            SELECT 
                vp.`nom_plan`, 
                MAX(vp.`fecha_desde_plan`) ult_a_fecha
            FROM 
                `valores_plan` vp
            WHERE 
                vp.`fecha_desde_plan` <= fecha_hasta
            GROUP BY 
                vp.`nom_plan`
        );
        
        SELECT 
            pc.`nom_plan`, 
            pc.`modalidad`, 
            vp.`valor_plan` valor_fecha
        FROM 
            `plan_capacitacion` pc
        INNER JOIN valor_a_fecha va 
            ON pc.`nom_plan` = va.`nom_plan`
        INNER JOIN `valores_plan` vp 
            ON va.`nom_plan` = vp.`nom_plan`
            AND va.ult_a_fecha = vp.`fecha_desde_plan`;
        
        DROP TEMPORARY TABLE IF EXISTS valor_a_fecha;
    END //

    DELIMITER ;

    /* Reutilización: Llamar al SP parametrizado desde el SP sin parámetros */
    DROP PROCEDURE IF EXISTS plan_lista_precios_actual;

    DELIMITER //
    CREATE PROCEDURE plan_lista_precios_actual()
    BEGIN
        CALL plan_lista_precios_a_fecha(CURRENT_DATE);
    END //
    DELIMITER ;
    </code></pre>

    <br><hr><br>

    <h3>🔹 Ejercicios 4 y 5: Funciones</h3>
    
    <pre><code>
    SET GLOBAL log_bin_trust_function_creators = 1;

    DELIMITER //

    CREATE FUNCTION plan_valor(nom VARCHAR(20), fecha DATE)
    RETURNS FLOAT(9,3)
    BEGIN
        DECLARE valor FLOAT(9,3);
        
        SELECT 
            valor_plan INTO valor
        FROM 
            valores_plan
        WHERE 
            nom_plan = nom
            AND fecha_desde_plan = fecha;
        
        RETURN valor;
    END //

    DELIMITER ;

    /* Prueba */
    SELECT plan_valor('Marketing 3', '2015-01-02');
    </code></pre>

    <br><hr><br>

    <h3>🔹 Ejercicio 6: Parámetros de Salida (OUT)</h3>
    
    <pre><code>
    DELIMITER //

    CREATE PROCEDURE alumnos_pagos_deudas_a_fecha(
        IN fecha_limite DATE, 
        IN dni_alumno INTEGER(11), 
        OUT pagado FLOAT(9,3), 
        OUT cant_adeudado INTEGER(11)
    )
    BEGIN
        /* Calcular pagado */
        SELECT 
            @pagado := SUM(cuo.`importe_pagado`)
        FROM 
            cuotas cuo
        WHERE 
            cuo.`dni` = dni_alumno 
            AND cuo.`fecha_pago` IS NOT NULL
            AND cuo.`fecha_emision` <= fecha_limite;
        
        /* Calcular deuda */
        SELECT 
            @cant_adeudado := COUNT(*)
        FROM 
            cuotas cuo
        WHERE 
            cuo.`dni` = dni_alumno 
            AND cuo.`fecha_pago` IS NULL
            AND cuo.`fecha_emision` <= fecha_limite;
        
        /* Asignar a variables de salida */
        SET pagado := @pagado;
        SET cant_adeudado := @cant_adeudado;
    END //

    DELIMITER ;
    </code></pre>

    <br><hr><br>

    <h3>🔹 Ejercicio 8, 9 y 10: Transacciones en SPs</h3>
    
    <pre><code>
    DELIMITER //

    CREATE PROCEDURE alumno_inscrip_con_validacion_2(
        IN dni_alumno INTEGER(11), 
        IN plan CHAR(20), 
        IN curso INTEGER(11)
    )
    BEGIN
        DECLARE insc INTEGER(11);
        
        START TRANSACTION;
        
        /* 1. Verificar si ya está inscripto */
        SELECT 
            COUNT(dni) INTO @insc
        FROM 
            inscripciones
        WHERE 
            dni = dni_alumno 
            AND nom_plan = plan 
            AND nro_curso = curso;
        
        IF @insc = 0 THEN
            /* 2. Insertar inscripción */
            INSERT INTO inscripciones
            VALUES (plan, curso, dni_alumno, CURRENT_DATE);
            
            /* 3. Actualizar cupo */
            UPDATE 
                cursos 
            SET 
                cant_inscriptos = cant_inscriptos + 1
            WHERE 
                nom_plan = plan 
                AND nro_curso = curso;
            
            /* 4. Generar cuota */
            INSERT INTO cuotas
            VALUES (
                plan, curso, dni_alumno,
                YEAR(ADDDATE(CURRENT_DATE, INTERVAL 1 MONTH)),
                MONTH(ADDDATE(CURRENT_DATE, INTERVAL 1 MONTH)),
                CURRENT_DATE, NULL, NULL
            );
            
            /* 5. Validar Cupo final */
            IF (
                SELECT cant_inscriptos FROM cursos 
                WHERE nom_plan = plan AND nro_curso = curso
               ) 
               <= 
               (
                SELECT cupo FROM cursos 
                WHERE nom_plan = plan AND nro_curso = curso
               ) 
            THEN
                COMMIT;
            ELSE
                ROLLBACK; -- Excede cupo
            END IF;
        ELSE
            ROLLBACK; -- Ya inscripto
        END IF;
    END //

    DELIMITER ;
    </code></pre>

    <br><hr><br>

    <h3>🔹 Ejercicio 11: Modularización (Stock)</h3>
    
    <pre><code>
    /* 1. SP Genérico */
    DELIMITER //
    CREATE PROCEDURE `stock_movimiento`(
        IN cod_mat CHAR(6), 
        IN cant_movida INTEGER(11), 
        OUT stock INTEGER(11)
    )
    BEGIN
        DECLARE url VARCHAR(50);
        START TRANSACTION;
        
        SELECT 
            url_descarga INTO url 
        FROM 
            materiales 
        WHERE 
            cod_material = cod_mat;
        
        /* Solo actualizar si es físico (url is null) */
        IF url IS NULL THEN
            UPDATE 
                materiales 
            SET 
                cant_disponible = cant_disponible + cant_movida
            WHERE 
                cod_material = cod_mat;
        END IF;
        
        SELECT 
            cant_disponible INTO stock 
        FROM 
            materiales 
        WHERE 
            cod_material = cod_mat;
        
        /* Validar stock negativo */
        IF stock >= 0 THEN
            COMMIT;
        ELSE
            ROLLBACK;
            SELECT cant_disponible INTO stock FROM materiales WHERE cod_material = cod_mat;
        END IF;
    END //

    /* 2. Wrappers */
    CREATE PROCEDURE `stock_ingreso`(
        IN cod_mat CHAR(6), 
        IN cant_movida INTEGER(11), 
        OUT stock INTEGER(11)
    )
    BEGIN
        CALL stock_movimiento(cod_mat, cant_movida, stock);
    END //

    CREATE PROCEDURE `stock_egreso`(
        IN cod_mat CHAR(6), 
        IN cant_movida INTEGER(11), 
        OUT stock INTEGER(11)
    )
    BEGIN
        CALL stock_movimiento(cod_mat, (-1) * cant_movida, stock);
    END //
    DELIMITER ;
    </code></pre>

    <br><br><br>
    <hr>
    <hr>
    <br><br><br>

    <!-- TRIGGERS -->
    <h2>🔔 Triggers</h2>
    <p><strong>BASE DE DATOS:</strong> <code>AFATSE</code></p>
    <br>

    <h3>🔹 Ejercicio 1: Auditoría de Alumnos</h3>
    
    <pre><code>
    /* Tabla Histórica */
    CREATE TABLE alumnos_historico (
        dni INT(11) NOT NULL,
        fecha_hora_cambio DATETIME NOT NULL,
        /* ... otros campos ... */
        usuario_modificacion VARCHAR(50) DEFAULT NULL,
        PRIMARY KEY (`dni`, `fecha_hora_cambio`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8;

    /* Trigger Insert */
    DELIMITER //
    CREATE TRIGGER alumnos_after_ins_tr AFTER INSERT ON alumnos
    FOR EACH ROW
    BEGIN
        INSERT INTO alumnos_historico
        VALUES (
            NEW.dni, 
            CURRENT_TIMESTAMP, 
            NEW.nombre, 
            NEW.apellido, 
            NEW.tel, 
            NEW.email, 
            NEW.direccion, 
            CURRENT_USER
        );
    END //
    DELIMITER ;
    </code></pre>

    <br><hr><br>

    <h3>🔹 Ejercicio 3: Mantenimiento Automático de Contadores</h3>
    
    <pre><code>
    /* Al Inscribir */
    DELIMITER //
    CREATE TRIGGER `inscripciones_after_ins_tr` AFTER INSERT ON `inscripciones`
    FOR EACH ROW
    BEGIN
        UPDATE 
            cursos
        SET 
            cant_inscriptos = cant_inscriptos + 1
        WHERE 
            nom_plan = NEW.nom_plan 
            AND nro_curso = NEW.nro_curso;
    END //
    DELIMITER ;

    /* Al Borrar */
    DELIMITER //
    CREATE TRIGGER `inscripciones_after_del_tr` AFTER DELETE ON `inscripciones`
    FOR EACH ROW
    BEGIN
        UPDATE 
            cursos
        SET 
            cant_inscriptos = cant_inscriptos - 1
        WHERE 
            nom_plan = OLD.nom_plan 
            AND nro_curso = OLD.nro_curso;
    END //
    DELIMITER ;
    </code></pre>

    <br><hr><br>

    <h3>🔹 Ejercicio 4: Auditoría de Usuario (Before Insert)</h3>
    
    <pre><code>
    DELIMITER //
    CREATE TRIGGER valores_plan_before_ins_tr BEFORE INSERT ON valores_plan
    FOR EACH ROW
    BEGIN
        SET NEW.usuario_alta = CURRENT_USER;
    END //
    DELIMITER ;
    </code></pre>

</body>
</html>
