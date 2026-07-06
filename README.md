# ReadyStand | Arquitectura de Microservicios

---

Trabajo preparado para el examen del día **05/07/2026**, considerando la integración de todos los microservicios del sistema ReadyStand mediante **Spring Cloud Gateway**, **Eureka Server**, documentación **Swagger/OpenAPI** y pruebas unitarias.

---

## Descripción del dominio

ReadyStand es una plataforma web orientada a eventos gastronómicos. El sistema permite que un organizador cree eventos, registre stands participantes y publique productos o menús disponibles. Los clientes pueden navegar eventos y stands, y una vez iniciado el evento, realizar pedidos de productos pertenecientes a uno o varios stands dentro de una misma compra.

El sistema divide internamente el pedido por stand, gestiona el pago, controla inventario, administra la preparación y notifica al cliente cuando cada parte del pedido se encuentra lista para retiro presencial.

En esta versión del proyecto se integran los microservicios principales mediante API Gateway y Eureka Server, manteniendo bases de datos independientes por servicio, comunicación REST mediante Feign Client, documentación Swagger y pruebas unitarias por capas.

---

## Integrantes

| Integrante | Responsabilidad principal |
| ---------- | ------------------------- |
| Rocío Muñoz | Usuario-Service, Evento-Service, Stand-Service, Producto-Service |
| Misael Concha | Pedido-Service, SubPedido-Service, Preparacion-Service |
| Bastián | Pago-Service, Notificacion-Service, Inventario-Service, API Gateway, Eureka Server |

---

## Repositorio API Gateway y Eureka

Repositorio utilizado para API Gateway y Eureka Server:

[Api-Gateway-V2](https://github.com/Bastian0711/Api-Gateway-V2)

---

## Microservicios y responsables

| Servicio | Encargado | Repositorio |
| -------- | --------- | ----------- |
| Usuario-Service | Rocío | [usuarios-service](https://github.com/Roc-Mun/usuarios-service) |
| Evento-Service | Rocío | [eventos-service](https://github.com/Roc-Mun/eventos-service) |
| Stand-Service | Rocío | [stands-service](https://github.com/Roc-Mun/stands-service) |
| Producto-Service | Rocío | [productos-service](https://github.com/Roc-Mun/productos-service) |
| Pedido-Service | Misael | [entorno-desarrollo](https://github.com/MisaeLConcha/entorno-desarrollo) |
| SubPedido-Service | Misael | [entorno-subpedido](https://github.com/MisaeLConcha/entorno-subpedido) |
| Preparacion-Service | Misael | [entorno-preparacion](https://github.com/MisaeLConcha/entorno-preparacion) |
| Pago-Service | Bastián | [microservicio-pagos](https://github.com/Bastian0711/microservicio-pagos) |
| Notificacion-Service | Bastián | [microservicio-notificacion](https://github.com/Bastian0711/microservicio-notificacion) |
| Inventario-Service | Bastián | [microservicio-inventario](https://github.com/Bastian0711/microservicio-inventario) |
| API Gateway / Eureka | Bastián | [Api-Gateway-V2](https://github.com/Bastian0711/Api-Gateway-V2) |

---

## Listado general de microservicios

| Microservicio | Nombre registrado en Eureka | Función principal |
| ------------- | --------------------------- | ----------------- |
| Usuario | MS-USUARIO | Gestiona identidad, registro, login y roles de usuario. |
| Evento | MS-EVENTO | Administra eventos gastronómicos y su ciclo de vida. |
| Stand | MS-STAND | Gestiona stands asociados a eventos. |
| Producto | MS-PRODUCTO | Administra productos disponibles por stand. |
| Pedido / Orden | MS-ORDEN | Gestiona la compra global del cliente. |
| SubPedido | MS-SUBPEDIDO | Divide el pedido por stand y controla el retiro presencial. |
| Preparacion | MS-PREPARACION | Administra el estado de preparación de cada subpedido. |
| Pago | MS-PAGOS | Registra y valida el pago de pedidos. |
| Notificacion | MS-NOTIFICACION | Envía avisos al cliente sobre cambios de estado. |
| Inventario | MS-INVENTARIO | Controla stock y disponibilidad de productos. |
| API Gateway | MS-GATEWAY | Centraliza el acceso a los microservicios. |
| Eureka Server | MS-EUREKA | Registra y descubre servicios dentro de la arquitectura. |

---

## Descripción de cada microservicio

**Usuario-Service** — Gestiona identidad, registro, login y roles como cliente u organizador.

**Evento-Service** — Administra el ciclo de vida del evento: borrador, publicado, iniciado y finalizado. Solo cuando el evento está en estado `iniciado` se habilita el flujo de compra.

**Stand-Service** — Gestiona los puestos participantes de cada evento. Permite registrar stands, asociarlos a eventos, activarlos o desactivarlos.

**Producto-Service** — Administra el catálogo de productos por stand. Incluye nombre, precio, descripción y estado de disponibilidad.

**Inventario-Service** — Controla el stock por producto. Verifica disponibilidad antes de confirmar una compra y descuenta stock cuando corresponde.

**Pedido-Service / Orden-Service** — Representa la compra global del cliente. Contiene el detalle de productos seleccionados, cantidades y relación con otros servicios necesarios para validar la compra.

**SubPedido-Service** — Divide el pedido por stand una vez confirmado el pago. Gestiona el retiro presencial mediante estados como pendiente, preparando, listo y retirado.

**Pago-Service** — Registra y valida el cobro. Tras un pago exitoso, permite continuar el flujo del pedido.

**Preparacion-Service** — Gestiona el estado operativo de preparación de cada subpedido y permite actualizar su avance.

**Notificacion-Service** — Envía avisos al cliente sobre eventos relevantes, como pedido confirmado, preparación iniciada o subpedido listo para retiro.

**API Gateway** — Centraliza el acceso a los microservicios y enruta las solicitudes hacia cada servicio mediante rutas configuradas por path.

**Eureka Server** — Permite el registro y descubrimiento de microservicios dentro de la arquitectura distribuida.

---

## Reglas de negocio principales

* Solo se permite comprar cuando el evento está en estado `iniciado`.
* Un stand debe estar asociado a un evento existente.
* Un producto debe estar asociado a un stand existente y activo.
* El stock debe validarse antes de confirmar una compra.
* Si no existe stock suficiente, la operación debe bloquearse.
* Un pedido puede contener productos de distintos stands.
* Los subpedidos se generan agrupando los productos por stand.
* No se puede retirar un subpedido si aún no está listo.
* El sistema debe notificar al cliente ante cambios de estado relevantes.
* Cada microservicio mantiene independencia de datos y expone sus operaciones mediante API REST.
* La comunicación entre servicios se realiza mediante Feign Client y/o rutas registradas en Eureka.

---

## Flujo general del sistema

1. El organizador crea un evento.
2. Se registran stands asociados al evento.
3. Se agregan productos al catálogo de cada stand.
4. El cliente consulta eventos, stands y productos disponibles.
5. El organizador inicia el evento.
6. El cliente crea un pedido con productos de uno o varios stands.
7. El sistema valida disponibilidad e inventario.
8. El cliente realiza el pago.
9. Se generan subpedidos agrupados por stand.
10. Cada stand gestiona la preparación de sus subpedidos.
11. El cliente recibe notificaciones sobre el avance.
12. El cliente retira presencialmente los productos cuando están listos.

---

## Arquitectura general

La arquitectura del sistema está basada en microservicios independientes, cada uno con su propia responsabilidad de negocio y su propia base de datos. La comunicación se realiza mediante APIs REST, Feign Client, Eureka Server para descubrimiento de servicios y API Gateway como punto central de entrada.

Componentes principales:

* Microservicios Spring Boot.
* Bases de datos MySQL independientes por servicio.
* Comunicación REST mediante Feign Client.
* Descubrimiento de servicios con Eureka Server.
* Enrutamiento centralizado mediante Spring Cloud Gateway.
* Documentación Swagger/OpenAPI.
* Pruebas unitarias por capas.
* Despliegue distribuido en instancias AWS EC2.
* Contenedores Docker administrados mediante Docker Compose.

---

## Dependencias entre servicios mediante Feign Client

```text
Evento       → Usuario
Stand        → Evento
Producto     → Stand
Inventario   → Producto
Pedido       → Usuario
Pedido       → Evento
Pedido       → Producto
Pedido       → Inventario
Pedido       → Notificacion
Pago         → Pedido
Pago         → SubPedido
SubPedido    → Pedido
SubPedido    → Stand
Preparacion  → SubPedido
Preparacion  → Notificacion
Notificacion → Usuario
```

---

## ¿Por qué usamos Feign Client?

En este proyecto se utilizó OpenFeign para permitir la comunicación entre microservicios mediante llamadas HTTP REST de forma ordenada, desacoplada y fácil de mantener.

Cada microservicio posee su propia base de datos independiente, por lo que no puede acceder directamente a tablas de otros servicios. Por este motivo, cuando un servicio necesita validar o consultar información externa, debe hacerlo mediante APIs REST usando Feign Client.

El uso de Feign permitió:

* Mantener independencia entre microservicios.
* Evitar compartir bases de datos entre servicios.
* Reducir acoplamiento directo entre módulos.
* Simplificar llamadas HTTP mediante interfaces Java.
* Facilitar escalabilidad y mantenimiento.
* Representar dependencias reales del negocio sin generar acoplamiento excesivo.
* Mantener una arquitectura distribuida coherente con el enfoque de microservicios.

---

## Tabla de contratos entre microservicios

| # | Origen | Destino | Método | Endpoint |
| - | ------ | ------- | ------ | -------- |
| 1 | Evento | Usuario | GET | `/api/v3/usuarios/{id}` |
| 2 | Stand | Evento | GET | `/api/v3/eventos/{id}` |
| 3 | Producto | Stand | GET | `/api/v3/stands/{id}` |
| 4 | Inventario | Producto | GET | `/api/v3/productos/{id}` |
| 5 | Pedido / Orden | Usuario | GET | `/api/v3/usuarios/{id}` |
| 6 | Pedido / Orden | Evento | GET | `/api/v3/eventos/{id}` |
| 7 | Pedido / Orden | Producto | GET | `/api/v3/productos/{id}` |
| 8 | Pedido / Orden | Inventario | GET | `/api/v3/inventario/producto/{id}` |
| 9 | Pedido / Orden | Inventario | POST | `/api/v3/inventario/descontar` |
| 10 | Pedido / Orden | Notificacion | POST | `/api/v3/notificaciones` |
| 11 | Pago | Pedido / Orden | GET | `[POR CONFIRMAR]` |
| 12 | Pago | SubPedido | POST | `/api/v3/subpedidos/generar` |
| 13 | SubPedido | Pedido / Orden | GET | `[POR CONFIRMAR]` |
| 14 | SubPedido | Stand | GET | `/api/v3/stands/{id}` |
| 15 | Preparacion | SubPedido | GET | `/api/v3/subpedidos/{id}` |
| 16 | Preparacion | Notificacion | POST | `/api/v3/notificaciones` |
| 17 | Notificacion | Usuario | GET | `/api/v3/usuarios/{id}` |

---

## API Gateway

El API Gateway funciona como punto de entrada central para enrutar solicitudes hacia los microservicios registrados en Eureka.

Repositorio:

[Api-Gateway-V2](https://github.com/Bastian0711/Api-Gateway-V2)

URL base del Gateway:

```text
http://54.173.240.75:8080
```

### Rutas principales del Gateway

| Servicio | Ruta Gateway |
| -------- | ------------ |
| Usuario | `http://54.173.240.75:8080/api/v3/usuarios` |
| Evento | `http://54.173.240.75:8080/api/v3/eventos` |
| Stand | `http://54.173.240.75:8080/api/v3/stands` |
| Producto | `http://54.173.240.75:8080/api/v3/productos` |
| Pedido / Orden | `[POR CONFIRMAR: http://54.173.240.75:8080/api/v3/pedidos o /api/v3/ordenes]` |
| SubPedido | `http://54.173.240.75:8080/api/v3/subpedidos` |
| Preparacion | `[POR CONFIRMAR]` |
| Pago | `http://54.173.240.75:8080/api/v3/pagos` |
| Notificacion | `http://54.173.240.75:8080/api/v3/notificaciones` |
| Inventario | `http://54.173.240.75:8080/api/v3/inventario` |

---

## Configuración general del Gateway

El Gateway utiliza rutas con `lb://NOMBRE-SERVICIO`, lo que permite redireccionar las peticiones hacia los microservicios registrados en Eureka.

Ejemplo de rutas configuradas:

```yaml
spring:
  cloud:
    gateway:
      mvc:
        routes:
          - id: usuario-route
            uri: lb://MS-USUARIO
            predicates:
              - Path=/api/v3/usuarios/**

          - id: evento-route
            uri: lb://MS-EVENTO
            predicates:
              - Path=/api/v3/eventos/**

          - id: stand-route
            uri: lb://MS-STAND
            predicates:
              - Path=/api/v3/stands/**

          - id: producto-route
            uri: lb://MS-PRODUCTO
            predicates:
              - Path=/api/v3/productos/**
```

---

## Eureka Server

Eureka Server permite registrar y descubrir los microservicios disponibles dentro de la arquitectura.

URL del dashboard de Eureka:

```text
http://54.173.240.75:8761
```

Servicios registrados esperados:

```text
MS-GATEWAY
MS-USUARIO
MS-EVENTO
MS-STAND
MS-PRODUCTO
MS-ORDEN
MS-SUBPEDIDO
MS-PREPARACION
MS-PAGOS
MS-NOTIFICACION
MS-INVENTARIO
```

---

## Documentación Swagger

Los microservicios documentados incorporan Swagger mediante Springdoc OpenAPI.

La dependencia utilizada en los módulos Spring Boot es:

```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.6.0</version>
</dependency>
```

Swagger permite visualizar y probar los endpoints REST desde el navegador, mostrando métodos HTTP, rutas, modelos de datos y códigos de respuesta.

En los controladores se utilizaron anotaciones como:

* `@Tag`
* `@Operation`
* `@ApiResponse`
* `@ApiResponses`
* `@Schema`

---

## Enlaces Swagger

| Servicio | Swagger |
| -------- | ------- |
| Usuario | `http://34.231.120.127:8081/doc/swagger-ui.html` |
| Evento | `http://34.231.120.127:8082/doc/swagger-ui.html` |
| Stand | `http://34.231.120.127:8083/doc/swagger-ui.html` |
| Producto | `http://34.231.120.127:8084/doc/swagger-ui.html` |
| Pedido / Orden | `http://19.210.19.5:8085/doc/swagger-ui.html` |
| SubPedido | `http://19.210.19.5:8086/doc/swagger-ui.html` |
| Preparacion | `http://19.210.19.5:8088/doc/swagger-ui.html` |
| Pago | `http://34.197.162.232:8087/doc/swagger-ui/index.html` |
| Notificacion | `http://34.197.162.232:8089/doc/swagger-ui/index.html` |
| Inventario | `http://34.197.162.232:8090/doc/swagger-ui/index.html` |

---

## Testing

Para esta versión se incorporaron pruebas unitarias en los microservicios del sistema, considerando pruebas por capas y validación de reglas de negocio principales.

El procedimiento general aplicado fue:

* Pruebas de modelo con JUnit 5.
* Pruebas de servicio con Mockito, `@Mock` e `@InjectMocks`.
* Pruebas de controlador con MockMvc.
* Pruebas de repositorio con `@DataJpaTest` y base H2 en memoria.
* Validación de reglas de negocio principales.
* Confirmación de ejecución exitosa mediante `BUILD SUCCESS`.

---

## Ejecución de pruebas unitarias

La pauta solicita al menos **80% de cobertura** en pruebas unitarias. En la siguiente tabla se registra la ejecución de pruebas confirmada por microservicio. La cobertura porcentual debe respaldarse con el reporte correspondiente si el equipo utiliza una herramienta como JaCoCo.

| Microservicio | Estado de pruebas | Resultado confirmado |
| ------------- | ----------------- | -------------------- |
| Usuario | Implementadas | Tests run: 14, Failures: 0, Errors: 0, Skipped: 0 |
| Evento | Implementadas | Tests run: 16, Failures: 0, Errors: 0, Skipped: 0 |
| Stand | Implementadas | Tests run: 19, Failures: 0, Errors: 0, Skipped: 0 |
| Producto | Implementadas | Tests run: 21, Failures: 0, Errors: 0, Skipped: 0 |
| Pedido / Orden | Implementadas | Tests run: 13, Failures: 0, Errors: 0, Skipped: 0 |
| SubPedido | Implementadas | Tests run: 10 , Failures: 0, Errors: 0, Skipped: 0 |
| Preparacion | Implementadas | Tests run: 13, Failures: 0, Errors: 0, Skipped: 0 |
| Pago | `[POR COMPLETAR]` | `[POR COMPLETAR]` |
| Notificacion | `[POR COMPLETAR]` | `[POR COMPLETAR]` |
| Inventario | Implementadas | `[POR COMPLETAR]` |

---

## Capas consideradas en testing

| Capa | Herramienta |
| ---- | ----------- |
| Modelo | JUnit 5 |
| Servicio | JUnit 5 + Mockito |
| Controlador | MockMvc |
| Repositorio | DataJpaTest + H2 |

---

## Ejecución de pruebas

Dentro de cada microservicio:

```bash
./mvnw test
```

También puede ejecutarse con Maven instalado:

```bash
mvn test
```

En los microservicios ejecutados mediante Docker Compose, se puede ingresar al contenedor o utilizar el entorno configurado para ejecutar Maven.

Ejemplo:

```bash
docker compose run --rm workspace-java bash -c "cd demo && mvn test"
```

---

## Stack tecnológico

* Java 21
* Spring Boot 3
* Spring Data JPA
* Spring Validation
* OpenFeign
* Spring Cloud Gateway
* Eureka Server
* Springdoc OpenAPI / Swagger
* JUnit 5
* Mockito
* MockMvc
* H2 Database para pruebas
* MySQL como base de datos por servicio
* Docker
* Docker Compose
* AWS EC2
* GitHub

---

## Despliegue

Cada integrante levanta sus servicios en su propia instancia EC2 usando Docker Compose.

Comando general:

```bash
docker compose up -d
```

Para reconstruir y levantar un servicio:

```bash
docker compose up -d --build
```

Para revisar contenedores activos:

```bash
docker ps
```

Para revisar logs:

```bash
docker logs --tail=100 NOMBRE_CONTENEDOR
```

---

## IPs y puertos por servicio

| Servicio | IP EC2 | Puerto |
| -------- | ------ | ------ |
| API Gateway | 54.173.240.75 | 8080 |
| Eureka Server | 54.173.240.75 | 8761 |
| Usuario-Service | 34.231.120.127 | 8081 |
| Evento-Service | 34.231.120.127 | 8082 |
| Stand-Service | 34.231.120.127 | 8083 |
| Producto-Service | 34.231.120.127 | 8084 |
| Pedido / Orden-Service | 18.210.19.5 | 8085 |
| SubPedido-Service | 18.210.19.5 | 8086 |
| Preparacion-Service | 18.210.19.5 | 8088 |
| Pago-Service | 34.197.162.232 | 8087 |
| Notificacion-Service | 34.197.162.232 | 8089 |
| Inventario-Service | 34.197.162.232 | 8090 |

---

## Security Groups

Los Security Groups fueron configurados para permitir la comunicación HTTP/REST entre microservicios desplegados en distintas instancias EC2.

Configuración general:

* API Gateway expuesto en puerto `8080`.
* Eureka Server expuesto en puerto `8761`.
* Microservicios expuestos en sus puertos correspondientes `8081` a `8090`.
* Comunicación habilitada entre las IPs públicas de las instancias EC2 participantes.
* Puertos de base de datos no expuestos públicamente.
* Cada instancia debe exponer únicamente los puertos necesarios para sus servicios.

---

## Validación de servicios registrados en Eureka

Para validar que los servicios están registrados correctamente, se debe ingresar al dashboard de Eureka:

```text
http://54.173.240.75:8761
```

Los servicios deben aparecer con estado `UP`.

También se puede probar mediante navegador o terminal:

```bash
curl -i http://54.173.240.75:8761/eureka/apps
```

---

## Cómo probar la integración

1. Levantar todos los servicios con Docker Compose en cada EC2:

```bash
docker compose up -d
```

2. Verificar que los contenedores estén activos:

```bash
docker ps
```

3. Revisar Eureka Server:

```text
http://54.173.240.75:8761
```

4. Confirmar que los servicios aparezcan registrados como `UP`.

5. Probar rutas principales desde API Gateway:

```text
http://54.173.240.75:8080/api/v3/usuarios
http://54.173.240.75:8080/api/v3/eventos
http://54.173.240.75:8080/api/v3/stands
http://54.173.240.75:8080/api/v3/productos
http://54.173.240.75:8080/api/v3/pagos
http://54.173.240.75:8080/api/v3/notificaciones
http://54.173.240.75:8080/api/v3/inventario
```

6. Probar Swagger de los microservicios documentados.

7. Ejecutar flujo funcional principal:

```text
crear usuario → crear evento → registrar stand → agregar productos → validar inventario → crear pedido → procesar pago → generar subpedidos → preparar pedido → notificar cliente
```

8. Ejecutar pruebas unitarias en cada microservicio:

```bash
mvn test
```

---

## Pruebas rápidas por terminal

### Microservicios de ejemplo

```bash
curl -i http://34.231.120.127:8081/api/v3/usuarios
curl -i http://34.231.120.127:8082/api/v3/eventos
curl -i http://34.231.120.127:8083/api/v3/stands
curl -i http://34.231.120.127:8084/api/v3/productos
```

### Rutas por Gateway

```bash
curl -i http://54.173.240.75:8080/api/v3/usuarios
curl -i http://54.173.240.75:8080/api/v3/eventos
curl -i http://54.173.240.75:8080/api/v3/stands
curl -i http://54.173.240.75:8080/api/v3/productos
curl -i http://54.173.240.75:8080/api/v3/pagos
curl -i http://54.173.240.75:8080/api/v3/notificaciones
curl -i http://54.173.240.75:8080/api/v3/inventario
```

---

## Estado actual de integración

Los microservicios principales se encuentran integrados mediante Eureka Server y API Gateway. Los servicios de Usuario, Evento, Stand y Producto se registran en Eureka con los nombres `MS-USUARIO`, `MS-EVENTO`, `MS-STAND` y `MS-PRODUCTO`, permitiendo su acceso centralizado mediante rutas del Gateway.

Los demás microservicios del sistema se integran bajo el mismo enfoque, usando nombres de servicio registrados en Eureka y rutas `/api/v3/...` configuradas en el Gateway.

---

## Notas finales

* Las rutas marcadas como `[POR COMPLETAR]` o `[POR CONFIRMAR]` deben ser actualizadas cuando se confirme la ruta final del microservicio correspondiente.
* La cobertura porcentual debe completarse con el reporte real obtenido en cada módulo si se utiliza herramienta de cobertura.
* Los enlaces Swagger deben mantenerse actualizados según la IP pública y puerto activo de cada servicio.
* El repositorio debe permanecer público y accesible para el docente y los integrantes del equipo.
