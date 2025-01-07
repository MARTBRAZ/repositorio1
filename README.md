# Reporte de Tickets


El presente trabajo consiste en un **Reporte de Tickets**, producto de los los registros que se ingresaron por medio de un sistema.

Se parte de una tabla única en Excel, una sábana de datos, y se procede a analizar los datos de los que se dispone:

- Nro de Ticket
- Prioridad
- Ubicación (Provincia, Estado)
- Categoría y Subcategoría
- Fecha de creación
- Fecha en que se tomó
- Fecha en que se completó
- Quien lo va a solucionar
- Por donde ingresó el ticket (Llamada, Correo)
- Supervisor

Se carga la tabla en Power Bi a partir del Libro de Excel y se empiezan a hacer las transformaciones necesarias.

## Modelo Relacional

Como primer paso hay que crear el Modelo Relacional, desagregando la tabla en varias tablas relacionadas a traves de campos llave.

Para ello, estando en Power Query, se hacen varios duplicados de la tabla, y se dejan los campos correspondientes a cada tabla nueva.

### Tablas generadas (campos de cada tabla)
- Supervisores (Supervisor)
- Categoría Tickets (Category, Subcategory)
- Equipos (Skill Team)
- Estados (State, Latitude, Longitude, Provincia)
- Prioridades (Priority)
- Vía (Reached Via)
- Analistas (Assignee)
- Calificaciones (Customer rating)

Por cada tabla se remueven los duplicados.

### Tabla Calendario
Cuando nos manejamos con fechas, en forma predefinida, Power BI crea un calendario automático, es decir genera una jerarquía de fechas (año, trimestre, mes, día) a partir de las fechas del modelo. Esto pasa por tener activada la opción de inteligencia de tiempo.

Estas tablas que se generan en forma automática no son la mejor opción para el modelo, ya que suelen provocar mas problemas que soluciones.

>[!tip]
>**Buena práctica: desactivar el calendario automático y generar una tabla calendario con todas las columnas que necesite el modelo.**

En este caso generamos el calendario a partir de código DAX en Power BI, desde Nueva Tabla y la marcamos como **Tabla de Fechas**.

![image](https://github.com/user-attachments/assets/78bc5101-0cba-478b-b157-7fb1b513d7aa)

### Tabla Tiempo

>[!tip]
>**Buena práctica: generamos una tabla de tiempos bien completa.**


Generamos la tabla de tiempos a partir de un código M ya establecido. De esta manera voy a tener cada hora, minuto y segundo por día.

![image](https://github.com/user-attachments/assets/1ee19ca9-7aa7-4933-96af-f1f41be8b2e0)

El **Modelo Relacional** resultante, una vez conectadas todas la tablas es el siguiente:

![image](https://github.com/user-attachments/assets/cc20d2ab-7e64-49e0-8646-31610e045af5)


## Creación de Medidas

>[!tip]
>**Buena práctica: generamos una tabla de medidas para guardar todas las medidas en ella**


Medida que calcula cantidad de tickets --> utilizo **COUNTROWS**

```JS
00.Q Tickets (Creados) = COUNTROWS('00_Tickets')
```
En el caso de las tablas ***Calendario*** y ***Tiempo*** vemos que hay 3 relaciones por cada una, esto es porque el modelo maneja 3 tipos de fechas: 
- fecha de creación del ticket
- fecha en la que se tomó el ticket
- fecha en la que se completó el ticket.
  
Pero se observa que sólo una de ellas está activa, la de creación, las restantes están con líneas punteadas (inactivas)

![image](https://github.com/user-attachments/assets/f0c35350-8f60-4d7a-bb86-48ffdab6c4de)

Puedo crear mas de una relación con un campo, pero sólo 1 de las relaciones va a ser la activa.

Para calcular la cantidad de Tickets Tomados, debo activar una relación inactiva --> lo hago con **USERELATIONSHIP**, y activo la relación de tickets tomados con la tabla calendario.

```JS
01.Q Tickets (Tomados) =
    CALCULATE(COUNTROWS('00_Tickets'),
    USERELATIONSHIP('00_Tickets'[PICKED_DATE], '10_Calendar DAX'[Date]))
```
Y hago lo mismo para los Tickets Cerrados

```JS
02.Q Tickets (Cerrados) =
    CALCULATE(COUNTROWS('00_Tickets'),
    USERELATIONSHIP('00_Tickets'[COMPLETED_DATE], '10_Calendar DAX'[Date]))
```

Debo seguir el mismo procedimiento para vincular la **Tabla de Tiempos** con los 3 tipos de tickets, ya que una única relación va a ser la activa (tickets creados).

### Slicer Tipo de Ticket 

Nuestro reporte va a tener un slicer para poder seleccionar entre **Tickets Creados**, **Tickets Tomados** y **Tickets Cerrados**.

Para ello creo una tabla **"Tipo de Ticket"**

![image](https://github.com/user-attachments/assets/d53f2939-c47e-4ca5-93ec-bb9921043515)

Se debe crear medida para que tome los datos de acuerdo a la selección.

Primero creo la Variable **S** para que almacene el tipo de dato seleccionado con **SELECTEDVALUE**.

Luego con **SWITCH** indico que medida tomar de acuerdo a la variable seleccionada.

```js
05. Q Tickets Seleccionados (Fechas) =

Var S = SELECTEDVALUE('11_Tipo de Ticket'[ID Tipo],1)

RETURN
SWITCH(TRUE(),
    S = 1, [00.Q Tickets (Creados)],
    S = 2, [01.Q Tickets (Tomados)],
    S = 3, [02.Q Tickets (Cerrados)]
)
```

Hago lo mismo con Tiempo:

```js
06. Q Tickets Seleccionados (Tiempo) =
 
Var S = SELECTEDVALUE('11_Tipo de Ticket'[ID Tipo],1)
  
RETURN
SWITCH(TRUE(),
    S = 1, [00.Q Tickets (Creados)],
    S = 2, [03.Q Tickets (Tomados Tiempo)],
    S = 3, [04.Q Tickets (Cerrados Tiempo)]
)
```
___
## Lienzo Supervisores

En este lienzo se muestran las estadísticas de los 3 supervisores, por tickets creados, en progreso y cerrados.

Para el reporte se necesitan las medidas que explicitamente me muestren los datos de cada supervisor.

**Medida para Aurora:**

Utilizo un CALCULATE sobre la medida de Tickets seleccionados previamente creada.

```js
08.Aurora Tickets =
    CALCULATE('11_Tipo de Ticket'[05. Q Tickets Seleccionados (Fechas)],
    '00_Tickets'[SUPERVISOR] = "Aurora Krammer")
```
Y hago lo mismo para Heather y para Jack.

Los visuales por supervisor quedan de la siguiente manera, mostrándose:
* las calificaciones de los tickets de cada uno (Excellent, Good, Satisfactory, Unsatisfactory)
* la prioridad (High, Medium, Low)
* Cantidad de tickets por día de la semana

![image](https://github.com/user-attachments/assets/6eb21455-89a3-494e-adbe-f61b814e9527)

De esta manera, gracias al slicer por estado del ticket y la medida por supervisor, puedo ver el desempeño de cada uno de manera muy visual

![image](https://github.com/user-attachments/assets/725e4185-9995-436f-b8e4-891ba1a83ce9)

___
## Lienzo Fechas

En este lienzo se muestra:
* la cantidad de tickets por semana del mes (Semana 1, Semana 2, Semana 3, Semana 4) --> utilizo el campo Semana Texto de la Tabla Calendario y la medida **Q Tickets Seleccionados (Fechas)**
* la cantidad de tickets por rango horario (12AM a 6AM, 6AM a 12PM, 12PM a 6PM, 6PM a 12AM) --> utilizo el campo Hourly Quartile de la Tabla Tiempo y la medida **Q Tickets Seleccionados (Fechas)**

El lienzo resultante es el siguiente:

![image](https://github.com/user-attachments/assets/21d015c8-0a9f-44a2-b627-6ba55fed76d9)

___
## Lienzo Empleados

En este lienzo puedo ver los empleados asignados a cada supervisor en una matriz, pudiendo seleccionar al supervisor por medio de un slicer.

También puedo visualizar, de acuerdo a la selección (creados, en progreso o cerrados):
* cantidad de tickets por día
* cantidad de tickets según su clasificación en Excellent, Good y Satisfactory
* cantidad de tickets según su prioridad(low, medium, high)
* cantidad de ticket por via (portal, call, mail)

La medida seleccionada para los distintos visuales sera la de **Q Tickets Seleccionados (Fechas)**

![image](https://github.com/user-attachments/assets/189a4e01-b056-491f-acaf-3af2f609f5e2)





___
## Lienzo Localización

En este reporte se responde a la pregunta **¿DONDE?**




___
## Lienzo Empresa





















