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

## Tabla Calendario
Cuando nos manejamos con fechas, en forma predefinida, Power BI crea una jerarquía de fechas (año, trimestre, mes, día) por tener activada la opción de inteligencia de tiempo.


## Tabla Tiempo
Una buena práctica si nos vamos a manejar con tiempos es tener una tabla tiempo bien completa, en este caso la generamos a partir de un código M ya establecido. De esta manera voy a tener cada hora, minuto y segundo por día.

![image](https://github.com/user-attachments/assets/1ee19ca9-7aa7-4933-96af-f1f41be8b2e0)












![image](https://github.com/user-attachments/assets/963dd0d4-57c9-49ee-999d-f89fb3d72564)


![image](https://github.com/user-attachments/assets/0f03ba45-aa03-426f-8f9e-7cbfa1cefd50)


![image](https://github.com/user-attachments/assets/abccf2a5-89f3-4298-80c2-78a6fa6efe1e)


![image](https://github.com/user-attachments/assets/98ea0f15-2e95-49b3-b960-3892acb5e7b1)


![image](https://github.com/user-attachments/assets/bec5f76e-2915-4fe7-836d-b2e3b0d96532)






