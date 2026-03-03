# Análisis de evasión de clientes – Telecom X

En este proyecto analizo el churn de clientes de una empresa de telecomunicaciones llamada Telecom X  
La idea es entender por qué algunos clientes dejan el servicio y qué perfiles son más propensos a cancelar

Usé Python y pandas para trabajar con los datos  
El objetivo es dejar un dataset limpio, explorar patrones básicos y sacar algunos insights que puedan ayudar al equipo del negocio


## Extracción de datos

Los datos vienen de una API en formato JSON publicada en GitHub  
Cargué la información directamente al notebook y la convertí a un DataFrame de pandas para poder trabajar mejor con las columnas y tipos

Se descargó un archivo con la información de cada cliente  
Incluye datos de cuenta, tipo de contrato, método de pago, facturación y el estado de churn, indicando si el cliente se fue o no


## Conociendo el conjunto de datos

Revisé la estructura usando `info()` y `head()`  
El dataset tiene miles de filas, cada fila representa un cliente

Hay columnas de varios tipos  
identificador del cliente  
datos de cuenta guardados en un diccionario (`account`)  
información de churn (`Churn`)  
otros bloques como `customer`, `phone`, `internet`

A partir de `account` extraje algunas variables que me interesaban más para el análisis de evasión  
tipo de contrato (`Contract`)  
si usa facturación electrónica (`PaperlessBilling`)  
método de pago (`PaymentMethod`)  
valores de cobro (`Charges`)


## Limpieza y tratamiento de datos

Antes de analizar, fue necesario ajustar varias cosas

La columna `account` venía como un diccionario en cada fila  
A partir de ahí armé un campo intermedio (`ficha_cuenta`) para poder acceder a las claves internas de forma más cómoda

De la parte de `Charges` creé tres columnas numéricas

- `pago_mensual` (Monthly)  
- `pago_total` (Total)  
- `pago_por_dia` (pago mensual dividido entre 30)

La variable `Churn` venía con valores en texto y con distintos formatos  
La normalicé y construí una columna binaria llamada `deja_de_pagar`, donde 1 significa que el cliente se fue y 0 que se quedó

También revisé valores nulos  
En los casos donde el target estaba vacío, esas filas no se usaron para entrenar el modelo

Al final quedó un dataset más simple, con las columnas principales

- `customerID`  
- `Contract`  
- `PaperlessBilling`  
- `PaymentMethod`  
- `pago_mensual`  
- `pago_total`  
- `pago_por_dia`  
- `deja_de_pagar`


## Análisis descriptivo

Con `describe()` obtuve estadísticas básicas de las variables numéricas de pago

El pago mensual promedio se sitúa alrededor de 65, con clientes que pagan menos y otros que pagan bastante más  
El pago total muestra mucha variación, lo que indica que hay clientes nuevos con poco tiempo y otros con varios meses o años acumulados

También miré la tasa global de evasión usando `value_counts()` y la media de `deja_de_pagar`  
La proporción de clientes que abandonan el servicio ronda cerca de una cuarta parte de la base total, lo que ya es un problema relevante para la empresa


## Distribución de evasión

Para ver la distribución de churn conté cuántos clientes se quedan y cuántos se van  
El gráfico de barras muestra que la mayoría de los clientes siguen activos, pero el grupo que cancela no es pequeño

Este desbalance es importante  
la clase de “no churn” tiene más ejemplos que la de “churn”  
Esto afecta tanto la lectura de los datos como el rendimiento de los modelos, que tienden a acertar más en la clase mayoritaria


## Evasión por variables categóricas

Analicé la tasa de evasión por tipo de contrato (`Contract`)

Los contratos de tipo mensual presentan una tasa de churn bastante más alta  
Los contratos de uno y dos años muestran valores de evasión mucho menores

También revisé la variable `PaymentMethod` para ver qué pasa por método de pago

Los clientes que pagan con `Electronic check` tienen una tasa de churn más elevada  
Los métodos automáticos como tarjeta o débito y el pago con `Mailed check` tienden a mostrar menos evasión

Con `PaperlessBilling` comparé quienes usan facturación electrónica frente a los que no

Los clientes con facturación electrónica tienen un porcentaje mayor de churn  
Esto se puede relacionar con otros hábitos, como uso más digital del servicio o combinación con planes de contrato mensual  
En cualquier caso es un grupo que vale la pena observar con más detalle


## Evasión y variables numéricas

Para las variables numéricas comparé los grupos de churn y no churn

Los clientes que se van suelen tener un `pago_mensual` más alto que los que se quedan  
Sin embargo, su `pago_total` acumulado es menor, indicando que se marchan antes de acumular muchos meses en la empresa

Los boxplots e histogramas permiten ver estas diferencias en la distribución  
En el grupo de churn se observa un desplazamiento hacia valores más altos de pago mensual  
Mientras que en el pago total el grupo de evasión se concentra en montos más bajos


## Modelo simple de churn

Además del análisis exploratorio, entrené un modelo sencillo de regresión logística

Usé solo variables de cuenta

- `Contract`  
- `PaperlessBilling`  
- `PaymentMethod`

Primero realicé one hot encoding sobre estas columnas para obtener variables binarias  
Luego separé los datos en entrenamiento y prueba con una proporción estándar y estratificación por el target

El modelo alcanzó una exactitud alrededor de tres cuartas partes  
Funciona mejor para identificar clientes que no se van  
y tiene más dificultad en la clase de evasión, lo cual es normal con el desbalance de clases

Los coeficientes se alinean con lo visto en el EDA  
ciertos tipos de contrato reducen la probabilidad de churn  
algunos métodos de pago y la facturación electrónica se relacionan con mayor riesgo de salida


## Conclusiones e insights

De todo el trabajo se pueden destacar varios puntos

- La tasa de evasión no es baja, un porcentaje importante de clientes acaba cancelando  
- Los contratos mensuales concentran la mayor proporción de churn, mientras que los contratos más largos retienen mejor  
- El método de pago `Electronic check` aparece como un foco de riesgo, con una tasa de salida más alta  
- Los clientes que se van pagan más al mes pero acumulan menos pago total, lo que sugiere una posible sensibilidad al precio o insatisfacción temprana

Estos hallazgos ofrecen una primera visión de qué segmentos merecen más atención en las estrategias de retención


## Recomendaciones

Con base en los resultados, algunas posibles acciones para el negocio son las siguientes

- Revisar y reforzar las ofertas para clientes con contrato mensual, promoviendo migraciones a planes de uno o dos años con beneficios claros  
- Analizar con mayor profundidad el segmento que usa `Electronic check` y proponer alternativas de pago más estables o campañas específicas para este grupo  
- Revisar los planes con pagos mensuales más altos, evaluando si es necesario ajustar precios, agregar valor percibido o trabajar mejor la comunicación del servicio  
- Aprovechar este dataset limpio y el modelo básico como punto de partida para construir modelos predictivos más completos, incluyendo más variables de uso y tiempo de permanencia

El proyecto deja preparada una base organizada y un análisis inicial que puede ayudar al equipo de Data Science y al área de negocio a diseñar acciones para reducir la evasión de clientes
