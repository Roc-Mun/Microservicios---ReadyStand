# ReadyStand — Arquitectura de Microservicios

Plataforma web para eventos gastronómicos. El organizador crea el evento, registra stands y publica menús. Los clientes navegan los stands y, una vez iniciado el evento, pueden realizar pedidos de múltiples stands en una sola compra. El sistema divide internamente el pedido por stand, gestiona la preparación y notifica al cliente cuando cada parte está lista para retiro presencial.

---

## Microservicios y responsables

| Servicio | Encargado | Repositorio |
|---|---|---|
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

---

## Descripción de cada microservicio

**Usuario** — Gestiona identidad, registro, login y roles (cliente, organizador).

**Evento** — Administra el ciclo de vida del evento: borrador → publicado → iniciado → finalizado. Solo cuando está en `iniciado` se habilita la compra.

**Stand** — Gestiona los puestos participantes de cada evento. Puede activarse o desactivarse para bloquear recepción de subpedidos.

**Producto** — Catálogo de productos por stand. Incluye nombre, precio, descripción y estado de disponibilidad.

**Inventario** — Controla el stock por producto. Verifica disponibilidad antes de confirmar la compra y descuenta automáticamente al confirmar.

**Pedido** — Representa la compra global del cliente. Contiene internamente `pedido_detalle` (producto, cantidad, stand). No llama a SubPedido para evitar dependencia circular.

**SubPedido** — Divide el pedido por stand una vez confirmado el pago. Gestiona el retiro presencial (estados: pendiente → preparando → listo → retirado).

**Pago** — Registra y valida el cobro. Tras pago exitoso, dispara la generación de subpedidos.

**Preparacion** — Gestiona el estado operativo en cocina por subpedido (pendiente → preparando → listo) y dispara notificaciones al cambiar estado.

**Notificacion** — Envía avisos al cliente: pedido confirmado, subpedido en preparación, subpedido listo para retiro.

---

## Reglas de negocio principales

- Solo 1 pedido activo por usuario a la vez.
- Solo 1 subpedido por stand dentro de un mismo pedido.
- No se puede retirar un subpedido parcialmente; se entrega solo cuando todos sus productos están listos.
- La compra solo se habilita cuando el evento está en estado `iniciado`.
- El stock se descuenta automáticamente al confirmar el pedido. Si no hay stock suficiente, la operación se bloquea.
- El organizador puede definir máximo de productos por pedido y máximo de subpedidos por pedido.

---

## Flujo general

1. El organizador crea el evento, registra stands y define menús con stock.
2. El cliente navega eventos y stands (solo consulta, sin compra).
3. El organizador inicia el evento → se habilita la compra.
4. El cliente selecciona productos de uno o varios stands → se crea un pedido con sus detalles internos.
5. El cliente confirma el pedido → paga directamente al microservicio Pago.
6. Pago valida y, si es exitoso, llama a SubPedido para generar los subpedidos agrupados por stand.
7. Cada stand gestiona la preparación de su subpedido a través de Preparacion.
8. Preparacion notifica al cliente en cada cambio de estado relevante.
9. El cliente retira presencialmente en cada stand → el subpedido pasa a `retirado`.

---

## Dependencias entre servicios (llamadas Feign Client)
```text
┌──────────────┐       ┌──────────────┐
│   Evento     │──────▶│   Usuario    │
└──────────────┘       └──────────────┘

┌──────────────┐       ┌──────────────┐
│    Stand     │──────▶│    Evento    │
└──────────────┘       └──────────────┘

┌──────────────┐       ┌──────────────┐
│   Producto   │──────▶│    Stand     │
└──────────────┘       └──────────────┘

┌──────────────┐       ┌──────────────┐
│ Inventario   │──────▶│   Producto   │
└──────────────┘       └──────────────┘

┌──────────────┐
│   Pedido     │──────────────▶ Usuario
│              │──────────────▶ Evento
│              │──────────────▶ Producto
│              │──────────────▶ Inventario
│              │──────────────▶ Notificacion
└──────────────┘

┌──────────────┐
│    Pago      │──────────────▶ Pedido
│              │──────────────▶ SubPedido
└──────────────┘

┌──────────────┐
│ SubPedido    │──────────────▶ Pedido
│              │──────────────▶ Stand
└──────────────┘

┌──────────────┐
│ Preparacion  │──────────────▶ SubPedido
│              │──────────────▶ Notificacion
└──────────────┘

┌──────────────┐       ┌──────────────┐
│ Notificacion │──────▶│   Usuario    │
└──────────────┘       └──────────────┘
```

---

## Tabla de contratos

| # | Origen | Destino | Método | Endpoint |
|---|---|---|---|---|
| 1 | Evento | Usuario | GET | `/api/usuarios/{id}` |
| 2 | Stand | Evento | GET | `/api/eventos/{id}` |
| 3 | Producto | Stand | GET | `/api/stands/{id}` |
| 4 | Inventario | Producto | GET | `/api/productos/{id}` |
| 5 | Pedido | Usuario | GET | `/api/usuarios/{id}` |
| 6 | Pedido | Evento | GET | `/api/eventos/{id}` |
| 7 | Pedido | Producto | GET | `/api/productos/{id}` |
| 8 | Pedido | Inventario | GET | `/api/inventarios/producto/{id}` |
| 9 | Pedido | Inventario | POST | `/api/inventarios/descontar` |
| 10 | Pedido | Notificacion | POST | `/api/notificaciones` |
| 11 | Pago | Pedido | GET | `/api/pedidos/{id}` |
| 12 | Pago | SubPedido | POST | `/api/subpedidos/generar` |
| 13 | SubPedido | Pedido | GET | `/api/pedidos/{id}/detalles` |
| 14 | SubPedido | Stand | GET | `/api/stands/{id}` |
| 15 | Preparacion | SubPedido | GET | `/api/subpedidos/{id}` |
| 16 | Preparacion | Notificacion | POST | `/api/notificaciones` |
| 17 | Notificacion | Usuario | GET | `/api/usuarios/{id}` |

---

## Stack tecnológico

- Spring Boot 3 + Spring Data JPA + Spring Validation
- OpenFeign (comunicación entre servicios)
- MySQL — base de datos independiente por servicio
- Docker + Docker Compose
- Despliegue en AWS EC2 — Escenario B (servicios distribuidos en múltiples instancias)

---

## Despliegue

Cada integrante levanta sus servicios en su propia instancia EC2:

```bash
docker compose up -d
```

### IPs y puertos por servicio

| Servicio | IP (EC2) | Puerto |
|---|---|---|
| Usuario-Service | 34.231.120.127 | 8081 |
| Evento-Service | 34.231.120.127 | 8082 |
| Stand-Service | 34.231.120.127 | 8083 |
| Producto-Service | 34.231.120.127 | 8084 |
| Pedido-Service | 18.210.19.5 | 8085 |
| SubPedido-Service | 18.210.19.5| 8086 |
| Preparacion-Service | 18.210.19.5 | 8087 |
| Pago-Service | 100.52.12.161 | 8088 |
| Notificacion-Service | 100.52.12.161 | 8089 |
| Inventario-Service | 100.52.12.161 | 8090 |

### Security Groups

- Configurados: Configurados para permitir comunicación entre microservicios desplegados en distintas instancias EC2, habilitando acceso HTTP/REST a los puertos 8081–8090.
- Cada instancia debe exponer únicamente el puerto de su servicio.
- Los servicios se comunican entre sí mediante las IPs públicas de cada EC2.
- No se exponen puertos de MySQL al exterior.

---

## Cómo probar la integración

1. Levantar todos los servicios con `docker compose up -d` en cada EC2.
2. Importar la colección de Postman.
3. Ejecutar el flujo: crear usuario → crear evento → registrar stand → agregar productos → crear y confirmar pedido → procesar pago.
4. Verificar que se generan subpedidos, cambian estados y llegan notificaciones.
