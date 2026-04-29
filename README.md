# Microservicios — ReadyStand

## Descripción

Proyecto basado en arquitectura de microservicios para la gestión de una plataforma tipo feria/eventos (usuarios, productos, stands, eventos, pedidos, etc.).

Cada microservicio es independiente y se despliega en su propia instancia.

---

## Microservicios y responsables

| Servicio                   | Encargado   |
| -------------------------- | ----------- |
| Usuarios                   | Rocío (R)   |
| Eventos                    | Rocío (R)   |
| Stands / Cocinas / Puestos | Rocío (R)   |
| Menú / Catálogo            | Rocío (R)   |
| Pedidos / Orden            | Misael (M)  |
| Sub Pedidos                | Misael (M)  |
| Pagos                      | Bastián (B) |
| Preparación                | Misael (M)  |
| Entrega / Retiro           | Misael (M)  |
| Notificaciones             | Bastián (B) |
| Inventario                 | Bastián (B) |

---

## Despliegue

Cada servicio se ejecuta de forma independiente en su respectiva EC2:

```
docker compose up -d
```

---

## Arquitectura

* Arquitectura distribuida
* Servicios desacoplados
* Comunicación vía API REST
* Base de datos por servicio

---

## Notas

* Cada repositorio contiene su propio backend y configuración
* No se comparten bases de datos entre servicios
* El repositorio maestro solo documenta e integra
