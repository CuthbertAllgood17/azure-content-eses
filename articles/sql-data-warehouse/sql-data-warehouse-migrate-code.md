<properties
   pageTitle="Migración del código SQL a Almacenamiento de datos SQL | Microsoft Azure"
   description="Sugerencias para migrar el código SQL a Almacenamiento de datos SQL de Azure a fin de desarrollar soluciones."
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="lodipalm"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="data-services"
   ms.date="06/03/2016"
   ms.author="lodipalm;barbkess;sonyama;jrj"/>

# Migración del código SQL a Almacenamiento de datos SQL

Al migrar el código desde otra base de datos a Almacenamiento de datos SQL, es muy probable que tenga que realizar cambios en el código base. Algunas características de Almacenamiento de datos SQL pueden mejorar significativamente el rendimiento, ya que están diseñadas para trabajar directamente en un modo distribuido. Sin embargo, para mantener el rendimiento y la escala, también hay algunas características que no están disponibles.

## Limitaciones comunes de T-SQL

En la lista siguiente se resumen las características más comunes que no se admiten en Almacenamiento de datos SQL de Azure. Los vínculos siguientes le llevan a soluciones alternativas para la característica no admitida:

- [ANSI JOINS en UPDATE][]
- [ANSI JOINS en DELETE][]
- [instrucción MERGE][]
- combinaciones entre bases de datos
- [cursores][]
- [SELECT..INTO][]
- [INSERT..EXEC][]
- cláusula OUTPUT
- funciones insertadas definidas por el usuario
- funciones de múltiples instrucciones
- [expresiones de tabla comunes](#Common-table-expressions)
- [expresiones de tabla común recursivas (CTE)](#Recursive-common-table-expressions-(CTE)
- procedimientos y funciones CLR
- función $partition
- variables de tabla
- parámetros de valor de tabla
- transacciones distribuidas
- trabajo con COMMIT/ROLLBACK
- SAVE TRANSACTION
- contextos de ejecución (EXECUTE AS)
- [cláusula GROUP BY con opciones ROLLUP/CUBE/GROUPING SETS][]
- [anidación de niveles más allá de 8][]
- [actualización a través de vistas][]
- [uso de SELECT para la asignación de variables][]
- [No escriba ningún tipo de datos MAX en cadenas de SQL dinámico][]

Afortunadamente, la mayoría de estas limitaciones se puede solucionar. Los artículos de desarrollo correspondientes antes mencionados incluyen explicaciones.

## Características admitidas de las CTE

Las expresiones de tabla común (CTE) se admiten parcialmente en Almacenamiento de datos SQL. Actualmente se admiten las siguientes características de CTE:

- Se puede especificar una CTE en una instrucción SELECT.
- Se puede especificar una CTE en una instrucción CREATE VIEW.
- Se puede especificar una CTE en una instrucción CREATE TABLE AS SELECT (CTAS) .
- Se puede especificar una CTE en una instrucción CREATE REMOTE TABLE AS SELECT (CRTAS).
- Se puede especificar una CTE en una instrucción CREATE EXTERNAL TABLE AS SELECT (CETAS).
- Se puede hacer referencia a una tabla remota desde una CTE.
- Se puede hacer referencia a una tabla externa desde una CTE.
- Se pueden incluir varias definiciones de consulta CTE en una CTE.

## Limitaciones de las CTE

Estas son algunas de las limitaciones de las expresiones de tabla comunes en Almacenamiento de datos SQL:

- Una CTE debe ir seguida de una sola instrucción SELECT. No se admiten las instrucciones INSERT, UPDATE, DELETE y MERGE.
- No se admite una expresión de tabla común que incluya referencias a sí misma (una expresión de tabla común recursiva). Consulte la sección siguiente.
- No se permite especificar más de una cláusula WITH en una CTE. Por ejemplo, si una definición de consulta CTE contiene una subconsulta, esta no puede contener una cláusula WITH anidada que defina otra CTE.
- No se puede usar una cláusula ORDER BY en las definiciones de consulta CTE, excepto cuando se especifica una cláusula TOP.
- Cuando se usa una CTE en una instrucción que forma parte de un lote, la instrucción antes de ella debe ir seguida por un punto y coma.
- Cuando se usa en instrucciones preparadas por sp\_prepare, las CTE comportarán del mismo modo que otras instrucciones SELECT en PDW. Sin embargo, si las CTE se usan como parte de las CETAS preparadas por sp\_prepare, el comportamiento puede diferir de SQL Server y de otras instrucciones PDW debido a la manera en que se implementa el enlace para sp\_prepare. Si SELECT que hace referencia a la CTE está usando una columna incorrecta que no existe en la CTE, sp\_prepare pasará sin detectar el error, pero el error se generará durante sp\_execute en su lugar.

## CTE recursivas

Las CTE recursivas no se admiten en Almacenamiento de datos SQL. La migración de CTE recursivas puede ser bastante completa y el mejor proceso es desglosarla en varios pasos. Normalmente puede usar un bucle y rellenar una tabla temporal conforme se recorren en iteración las consultas provisionales recursivas. Cuando se rellene la tabla temporal, puede devolver los datos como un conjunto único de resultados. Se ha usado un enfoque similar para resolver `GROUP BY WITH CUBE` en el artículo [cláusula agrupar por con opciones de acumulación/cubo/agrupación][].

## Funciones del sistema

También hay algunas funciones del sistema que no son compatibles. Algunas de las principales que normalmente se usan en almacenamiento de datos son:

- NEWSEQUENTIALID()
- @@NESTLEVEL()
- @@IDENTITY()
- @@ROWCOUNT()
- ROWCOUNT\_BIG
- ERROR\_LINE()

De nuevo, muchos de estos problemas se pueden solucionar.

Por ejemplo, el código siguiente es una solución alternativa para recuperar información de @@ROWCOUNT:

```sql
SELECT  SUM(row_count) AS row_count
FROM    sys.dm_pdw_sql_requests
WHERE   row_count <> -1
AND     request_id IN
                    (   SELECT TOP 1    request_id
                        FROM            sys.dm_pdw_exec_requests
                        WHERE           session_id = SESSION_ID()
                        AND             resource_class IS NOT NULL
                        ORDER BY end_time DESC
                    )
;
```

## Pasos siguientes
Para ver una lista completa de todas las instrucciones de T-SQL admitidas, consulte [Temas de Transact-SQL][].

<!--Image references-->

<!--Article references-->
[ANSI JOINS en UPDATE]: ./sql-data-warehouse-develop-ctas.md
[ANSI JOINS en DELETE]: ./sql-data-warehouse-develop-ctas.md
[instrucción MERGE]: ./sql-data-warehouse-develop-ctas.md
[INSERT..EXEC]: ./sql-data-warehouse-develop-temporary-tables.md
[Temas de Transact-SQL]: ./sql-data-warehouse-reference-tsql-statements.md

[cursores]: ./sql-data-warehouse-develop-loops.md
[SELECT..INTO]: ./sql-data-warehouse-develop-ctas.md
[cláusula GROUP BY con opciones ROLLUP/CUBE/GROUPING SETS]: ./sql-data-warehouse-develop-group-by-options.md
[cláusula agrupar por con opciones de acumulación/cubo/agrupación]: ./sql-data-warehouse-develop-group-by-options.md
[anidación de niveles más allá de 8]: ./sql-data-warehouse-develop-transactions.md
[actualización a través de vistas]: ./sql-data-warehouse-develop-views.md
[uso de SELECT para la asignación de variables]: ./sql-data-warehouse-develop-variable-assignment.md
[No escriba ningún tipo de datos MAX en cadenas de SQL dinámico]: ./sql-data-warehouse-develop-dynamic-sql.md

<!--MSDN references-->

<!--Other Web references-->

<!---HONumber=AcomDC_0608_2016-->