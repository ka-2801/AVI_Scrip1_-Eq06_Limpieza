**Resultados: Diferencias entre el dataset original y el dataset limpio**

Este documento describe que genero el proceso de limpieza y compara los dos archivos de datos del repositorio.


*Dataset original: `SINAICA_Marzo_2024_og.xlsx`*
Hoja `Data`: 114,192 filas.

1. Contiene el año completo 2024 (no solo marzo), para las 13 estaciones de la red de monitoreo de calidad del aire de Jalisco.
2. 22 columnas: `STATION`, `DATE`, `HOUR`, `DIA`, `MES`, `O3`, `NO`, `NO2`, `NOX`, `SO2`, `CO`, `PM10`, `PM2.5`, `IT`, `ET`, `RH`, `WS`, `WD`, `PP`, `ATM`, `RS`, `UVI`.
3. Incluye una segunda hoja, `Param`, con la ficha tecnica de cada estacion (nombre, abreviatura, parametros de contaminantes y de meteorologia que mide, coordenadas, direccion, año de instalacion) y un catalogo de parametros con su unidad de medicion.
4. Al venir de todas las estaciones, varias columnas de medicion (por ejemplo `PM2.5`) se cargan con tipo de dato mixto (`object`) en lugar de numerico, porque combinan valores numericos de unas estaciones con datos vacios o mal capturados de otras.
5. No esta filtrado por estacion, periodo, ni alcance del analisis: incluye estaciones y contaminantes que no son relevantes para este trabajo.


*Dataset limpio: `SINAICA_VALmarzo2024_limpio.xlsx`*
Hoja `VAL_marzo2024`: 744 filas (31 dias x 24 horas de marzo 2024).

1. Una sola estacion: VAL (Vallarta).
2. 11 columnas: `STATION`, `DATE`, `HOUR`, `DIA`, `MES`, `O3`, `PM10`, `PM2.5`, `ET`, `IT`, `WS`.
3. Tipos de dato estandarizados: columnas de medicion en numerico, `HOUR`/`DIA`/`MES` en entero.
4. Secuencia horaria completa y validada, sin huecos ni fechas duplicadas.
5. Sin valores negativos en las concentraciones de contaminantes.


*Qué se elimino*

| Columnas eliminadas | Motivo |
|---|---|
| `NO`, `NO2`, `NOX`, `SO2`, `CO` | VAL no tiene esos sensores instalados fisicamente. Se verifico en el codigo que su disponibilidad es 0% en VAL para marzo 2024 |
| `ATM`, `RS` | Quedan fuera del alcance declarado del analisis (presion atmosferica y radiacion solar) |
| `RH`, `WD`, `PP`, `UVI` | Segun la ficha tecnica, VAL si deberia registrar estas 4 variables (humedad relativa, direccion del viento, precipitacion e indice UV). Sin embargo, en los datos reales de marzo 2024 vienen vacias al 100% (0 de 744 horas). |


*Correcciones*

Tipos de dato: columnas de medicion pasadas de `object`/mixto a numerico; `HOUR`, `DIA`, `MES` pasados a entero.
Valores negativos: se busco en `O3`, `PM10` y `PM2.5`. En esta corrida no se encontraron valores negativos (0 corregidos), pero el proceso queda documentado y disponible para reutilizarse con otros meses o estaciones donde si pudieran aparecer.

*Que se valido y no requirio correccion*

Registros esperados vs. obtenidos: 744 esperados, 744 obtenidos.
Horas faltantes como fila: 0.
Fechas duplicadas: 0.


*Disponibilidad de datos en el dataset limpio*

De las 744 horas posibles en marzo 2024, la disponibilidad real de cada variable que si quedo en la base limpia es:

| Variable | % disponible | Dias con al menos 1 dato |
|----------|--------------|---------------------------|
| O3       | 42.6%        | 14 de 31                  |
| PM10     | 42.5%        | 14 de 31                  |
| PM2.5    | 36.3%        | 14 de 31                  |
| ET       | 42.6%        | 14 de 31                  |
| IT       | 42.6%        | 14 de 31                  |
| WS       | 33.5%        | 14 de 31                  |

Aunque estas 6 variables si tienen datos reales (a diferencia de las 4 que se eliminaron debido a que estar vacias al 100%), ninguna llega ni a la mitad de las horas del mes: como maximo cubren 14 de los 31 dias de marzo con al menos una lectura.


**Resumen**
El dataset original es una base cruda, sin filtrar, de todo 2024 para las 13 estaciones de la red, con columnas de tipos mixtos y sin ninguna validacion de calidad.
En cambio, el dataset limpio es el subconjunto exacto necesario para el analisis de la estacion VAL en el mes de marzo de 2024: filtrado por estacion y periodo, restringido a las variables dentro del alcance declarado, sin columnas sin informacion real, con tipos de dato estandarizados, y con validaciones explicitas de consistencia temporal y de rangos fisicos posibles.

