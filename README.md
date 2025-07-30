-- Cargar archivos
estudiantes = LOAD 'estudiantes.csv' USING PigStorage(',') 
    AS (id_est:int, nombre:chararray, id_carrera:int);

materias = LOAD 'materias.csv' USING PigStorage(',') 
    AS (id_mat:int, nombre_mat:chararray, id_carrera:int);

notas = LOAD 'notas.csv' USING PigStorage(',') 
    AS (id_est:int, id_mat:int, nota:float);

-- A) TOP 10 ESTUDIANTES SEGÚN PROMEDIO
grupo_estudiantes = GROUP notas BY id_est;
promedios_est = FOREACH grupo_estudiantes GENERATE group AS id_est, AVG(notas.nota) AS promedio;

-- Unir con nombres de estudiantes
join_est = JOIN promedios_est BY id_est, estudiantes BY id_est;
estudiantes_final = FOREACH join_est GENERATE estudiantes::nombre AS nombre, promedios_est::promedio AS promedio;
ordenado_est = ORDER estudiantes_final BY promedio DESC;
top10 = LIMIT ordenado_est 10;

-- B) TOP 3 CARRERAS CON MAYOR PROMEDIO
-- Unir notas → materias → estudiantes (para saber a qué carrera pertenece cada estudiante)
join_nota_materia = JOIN notas BY id_mat, materias BY id_mat;
join_todo = JOIN join_nota_materia BY notas::id_est, estudiantes BY id_est;

-- Agrupar por carrera
grupo_carrera = GROUP join_todo BY estudiantes::id_carrera;
promedio_carrera = FOREACH grupo_carrera GENERATE group AS id_carrera, AVG(join_todo.notas::nota) AS promedio;

-- Ordenar y sacar top 3
ordenado_carrera = ORDER promedio_carrera BY promedio DESC;
top3_carreras = LIMIT ordenado_carrera 3;

-- Mostrar resultados
DUMP top10;
DUMP top3_carreras;
