# AVI_Scrip1_ Eq06_Limpieza
Limpieza y validacion de datos SINAICA (calidad del aire) para la estacion Vallarta (VAL), Jalisco, marzo 2024: de 114,192 filas del año completo/13 estaciones a 744 filas validadas y estandarizadas.

**Limpieza de dataset SINAICA - Estacion Vallarta (VAL), Marzo 2024**

Limpieza y estandarizacion de datos de calidad del aire y meteorologia, tomados de la red SINAICA (Sistema Nacional de Informacion de la Calidad del Aire) para la estacion Vallarta (VAL), correspondientes al periodo del 1 al 31 de marzo de 2024.



*Contenido del repositorio*
1. `limpieza_s1_marzo24.ipynb` - notebook con el script de limpieza
2. `SINAICA_Marzo_2024_og.xlsx` - dataset original (entrada)
3. `SINAICA_VALmarzo2024_limpio.xlsx` - dataset limpio (salida)
4. `Resultados.md` - comparacion detallada entre ambos datasets



*Requisitos*
- Python 3
- pandas
- numpy
- openpyxl


*Instalacion:*

```
pip install pandas numpy openpyxl
```

Datos de entrada
Archivo: `SINAICA_Marzo_2024_og.xlsx`, hoja `Data`. 
Aunque el nombre del archivo dice "Marzo 2024", en realidad contiene el año completo 2024 para las 13 estaciones de la red de monitoreo de Jalisco (114,192 filas en total).

Columnas originales:

```
STATION, DATE, HOUR, DIA, MES, O3, NO, NO2, NOX, SO2, CO,
PM10, PM2.5, IT, ET, RH, WS, WD, PP, ATM, RS, UVI
```
La estacion objetivo es *VAL (Vallarta)*, la cual, segun la ficha tecnica de la red, solo tiene instalados sensores de contaminantes para O3, PM10 y PM2.5, y de meteorologia para ET, IT, RH, WS, WD, PP y UVI.


**Que hace el script, paso a paso**

1. *Carga de datos*: lee la hoja `Data` del archivo original con pandas y limpia espacios en los nombres de columnas.

2. *Filtrado de alcance*: convierte la columna `DATE` a formato de fecha y filtra unicamente los registros donde `STATION == "VAL"` y la fecha esta entre `2024-03-01 00:00:00` y `2024-03-31 23:00:00`. El numero esperado de registros es 31 dias x 24 horas = 744.

3. *Eliminacion de columnas de sensores inexistentes en VAL*: elimina `NO`, `NO2`, `NOX`, `SO2`, `CO`. Antes de eliminarlas, se verifica que su disponibilidad de datos sea 0% para VAL en marzo; si alguna columna tuviera datos, imprime un aviso en vez de eliminarla silenciosamente.

4. *Restriccion a las variables del alcance del analisis*: conserva solo las columnas identificadoras (`STATION`, `DATE`, `HOUR`, `DIA`, `MES`) mas las variables de interes (`O3`, `PM10`, `PM2.5`, `ET`, `IT`, `RH`, `WS`, `WD`, `PP`, `UVI`). Con esto se eliminan de forma implicita las columnas `ATM` y `RS`, que estan fuera del alcance declarado.

5. *Eliminacion de columnas sin datos reales*: detecta y elimina las columnas que, aunque en teoria deberian tener datos, vienen vacias al 100% en marzo 2024. En la corrida documentada, estas columnas fueron `RH`, `WD`, `PP` y `UVI`.

6. *Estandarizacion de tipos de dato*: convierte a numerico (con `errors="coerce"`) todas las columnas de medicion restantes, y castea `HOUR`, `DIA` y `MES` a entero. Esto corrige columnas como `PM2.5`, que llegaban como tipo `object` por mezclarse con datos no numericos de otras estaciones en el dataset original.

7. *Validacion de consistencia temporal*: construye la secuencia horaria completa esperada (744 horas, del 2024-03-01 00:00 al 2024-03-31 23:00) y la compara contra las fechas presentes para detectar horas faltantes como fila y fechas duplicadas.
El dataset se ordena cronologicamente al final de este paso.

8. *Validacion de rangos fisicos*: revisa que ninguna concentracion de `O3`, `PM10` o `PM2.5` sea negativa. Cualquier valor negativo se sustituye por `NaN`, ya que fisicamente no es posible y se considera un error de captura.

9. *Guardad*: exporta el resultado a `SINAICA_VALmarzo2024_limpio.xlsx`, hoja `VAL_marzo2024`.


**Resultado de la corrida documentada**

```
Registros esperados (31 dias x 24 h): 744
Registros obtenidos tras el filtro:   744
Horas faltantes como fila:            0
Filas con fecha duplicada:            0
Valores negativos corregidos a NaN:   0
Columnas eliminadas (sensor inexistente en VAL): NO, NO2, NOX, SO2, CO
Columnas eliminadas (fuera del alcance del PDF): ATM, RS
Columnas eliminadas (0% de datos reales en marzo): RH, WD, PP, UVI
Columnas finales en la base limpia: STATION, DATE, HOUR, DIA, MES, O3,
PM10, PM2.5, ET, IT, WS
```

Disponibilidad final por variable:

| Variable | % disponible | Dias con al menos 1 dato |
|----------|-------------|---------------------------|
| O3       | 42.6%       | 14                        |
| PM10     | 42.5%       | 14                        |
| PM2.5    | 36.3%       | 14                        |
| ET       | 42.6%       | 14                        |
| IT       | 42.6%       | 14                        |
| WS       | 33.5%       | 14                        |



**Notas importantes**

*La ausencia de las columnas `NO`, `NO2`, `NOX`, `SO2` y `CO` no es un dato faltante:* la estacion VAL no tiene fisicamente esos sensores instalados.
*La ausencia de datos en `RH`, `WD`, `PP` y `UVI` es distinta*: la ficha tecnica indica que VAL si deberia contar con esos sensores, pero en marzo 2024 no generaron ninguna lectura. Por esto se eliminan de la base final (0% de datos reales), aunque conceptualmente el equipo si existe.
*El bloque de validacion de valores negativos se deja en el codigo* aunque en esta corrida no se encontraron casos, puede ser reusable si el script se corre con otro mes o estacion.

