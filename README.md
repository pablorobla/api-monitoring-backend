# api-monitoring-backend

Se trata del backend de una aplicación de monitorización de endpoints HTTP.
El sistema realiza comprobaciones periodicas, almacena los resultados historicos y expone el estado actual de las url mediante un API REST. 

La funcionalidad principal es conocer si el endopint está disponible (**UP/DOWN**), consultar el historico de checks realizados y analizar los tiempos de respuesta.

# Idea del proyecto

Está pensado para aprender y demostrar:
- cómo diseñar un backend más allá de un CRUD
- cómo trabajar con tareas programadas (scheduler)
- cómo manejar llamadas HTTP externas y errores
- cómo exponer información útil mediante una API REST

# Arquitectura y flujo de la aplicación

El backend sigue una arquitectura en capas basada en Spring Boot, separando claramente responsabilidades:

- **Scheduler**: se encarga de ejecutar periódicamente las comprobaciones de los endpoints configurados.
- **Service**: contiene la lógica de negocio (monitorización, validación, transformación de datos).
- **Repository**: gestiona el acceso a datos mediante Spring Data JPA.
- **Controller**: expone la información mediante una API REST.
- **DTOs**: se utilizan para definir los modelos de respuesta de la API y evitar exponer directamente las entidades JPA.

El flujo principal de la aplicación es el siguiente:

1. El scheduler obtiene los endpoints activos desde la base de datos.
2. Para cada endpoint, se realiza una llamada HTTP externa.
3. Se mide el tiempo de respuesta y se captura el código de estado HTTP.
4. El resultado de la comprobación se almacena como un registro histórico.
5. La API REST permite consultar tanto el estado actual (UP / DOWN) como el histórico de checks.

## Endpoints principales

La API expone varios endpoints REST para consultar el estado y el histórico de los endpoints monitorizados.

### Endpoints de monitorización

| Método | URL | Descripción |
|------|-----|-------------|
| GET | `/api/endpoints` | Obtiene la lista de endpoints configurados |
| GET | `/api/endpoints/{id}/status` | Devuelve el estado actual del endpoint (UP / DOWN) |
| GET | `/api/endpoints/{id}/checks` | Devuelve el histórico (limitado a 20) de comprobaciones del endpoint |

### Detalle de los endpoints

- **Estado actual**
  - `GET /api/endpoints/{id}/status`
  - Devuelve el resultado de la última comprobación realizada sobre el endpoint.

- **Histórico de checks**
  - `GET /api/endpoints/{id}/checks`
  - Devuelve una lista de las ultimas 20 comprobaciones históricas, incluyendo:
    - fecha y hora del check
    - código de estado HTTP
    - tiempo de respuesta
    - resultado (success / failure)
      
## Decisiones técnicas

- **Uso de un scheduler**  
Se utiliza un scheduler para ejecutar las comprobaciones de forma periódica y automática, desacoplando la ejecución de los checks de las peticiones HTTP.

- **Separación por capas (Controller / Service / Repository)**  
La lógica de negocio se concentra en la capa de servicio, mientras que los controllers se limitan a exponer la API REST y los repositories a gestionar el acceso a       datos.

- **Uso de Spring Data JPA con queries derivadas**  
Se emplean métodos de repositorio basados en el nombre del método para generar consultas automáticamente, evitando SQL manual cuando no es necesario y manteniendo el    código más legible.

- **Uso de DTOs**  
Se utilizan DTOs para definir los modelos de respuesta de la API, evitando exponer directamente las entidades JPA y adaptando cada respuesta a su caso de uso concreto.

- **Gestión de errores en llamadas HTTP externas**  
Los fallos en las llamadas HTTP (timeouts, errores 4xx/5xx) se capturan y registran como resultados de monitorización, evitando que el scheduler se detenga ante errores externos.

- **Persistencia de resultados históricos**  
Cada comprobación se almacena como un registro independiente, permitiendo analizar el histórico de checks y el comportamiento de los endpoints a lo largo del tiempo.

