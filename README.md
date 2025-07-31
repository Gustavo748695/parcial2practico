-- Cargar archivos con tipos correctos
estudiantes = LOAD 'estudiantes.csv' USING PigStorage(',')
    AS (registro:chararray, nombre:chararray, apellido:chararray, codigo_carrera:chararray, 
        carrera:chararray, semestre:int, fecha_ingreso:chararray);

materias = LOAD 'materias.csv' USING PigStorage(',')
    AS (codigo_materia:chararray, nombre_materia:chararray, creditos:int, 
        codigo_carrera:chararray, carrera:chararray, semestre_cursada:int, docente:chararray);

notas = LOAD 'notas.csv' USING PigStorage(',')
    AS (id_nota:chararray, registro_estudiante:chararray, codigo_materia:chararray, 
        nombre_materia:chararray, nota:int, fecha_evaluacion:chararray, tipo_evaluacion:chararray);

--------------------------------------------------
-- A) TOP 10 ESTUDIANTES CON MEJOR PROMEDIO
--------------------------------------------------

grupo_est = GROUP notas BY registro_estudiante;
promedios_est = FOREACH grupo_est GENERATE group AS registro, AVG(notas.nota) AS promedio;

-- Unir con datos de estudiante para mostrar nombre completo
join_est = JOIN promedios_est BY registro, estudiantes BY registro;
estudiantes_final = FOREACH join_est GENERATE CONCAT(estudiantes::nombre, ' ', estudiantes::apellido) AS nombre_completo, promedios_est::promedio AS promedio;

ordenado_est = ORDER estudiantes_final BY promedio DESC;
top10 = LIMIT ordenado_est 10;

--------------------------------------------------
-- B) TOP 3 CARRERAS CON MEJOR PROMEDIO GENERAL
--------------------------------------------------

-- Unir notas con materias
join_nota_materia = JOIN notas BY codigo_materia, materias BY codigo_materia;

-- Unir con estudiantes
join_todo = JOIN join_nota_materia BY notas::registro_estudiante, estudiantes BY registro;

-- Agrupar por carrera
grupo_carrera = GROUP join_todo BY estudiantes::codigo_carrera;
promedio_carrera = FOREACH grupo_carrera GENERATE group AS codigo_carrera, AVG(join_todo.notas::nota) AS promedio;

ordenado_carrera = ORDER promedio_carrera BY promedio DESC;
top3_carreras = LIMIT ordenado_carrera 3;

--------------------------------------------------
-- Mostrar resultados
--------------------------------------------------
DUMP top10;
DUMP top3_carreras;
