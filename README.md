# Lab19_Cloud2

Laboratorio 19 · Modernización de una Aplicación Monolítica (patrón **Strangler Fig**).

Se construye un monolito de checkout y se extrae el chequeo de inventario a un microservicio independiente. Una fachada decide, mediante la variable `STRANGLE_INVENTORY`, si enrutar cada solicitud al monolito o al nuevo servicio.

## Estructura

| Servicio | Carpeta | Puerto | Qué hace |
|---|---|---|---|
| Monolito | `monolith/` | 4000 | Lógica original de inventario y pago (`/checkout`, `/checkout-payment-only`) |
| Microservicio de inventario | `inventory-service/` | 4001 | Solo revisa el stock (`/check-stock`) |
| Fachada | `facade/` | 3000 | Recibe los `/checkout` y decide a dónde enrutarlos |

## Instalación

Después de clonar, instalar las dependencias de cada servicio (`node_modules` no se sube al repo):

```bash
cd monolith && npm install && cd ..
cd inventory-service && npm install && cd ..
cd facade && npm install && cd ..
```

## Cómo levantar los servicios

Los 3 procesos se siguen usando en el Laboratorio 20. Cada uno corre en su propia terminal, desde la raíz del repo:

```bash
# Terminal 1 · Monolito (puerto 4000)
node monolith/app.js

# Terminal 2 · Microservicio de inventario (puerto 4001)
node inventory-service/app.js

# Terminal 3 · Fachada (puerto 3000), elegir un modo:
STRANGLE_INVENTORY=false node facade/app.js   # modo "antes": todo pasa por el monolito
STRANGLE_INVENTORY=true  node facade/app.js   # modo "modernizado": inventario por el microservicio
```

> En Windows (CMD) la variable se define antes con `set STRANGLE_INVENTORY=true` y luego se ejecuta `node facade/app.js`. En PowerShell: `$env:STRANGLE_INVENTORY="true"`.

Para detener cualquier servicio: `Ctrl + C` en su terminal.

## Prueba

```bash
curl -X POST http://localhost:3000/checkout \
  -H "Content-Type: application/json" \
  -d '{"orderId":"T1","productId":"sku-1","amount":20}'
```

| Modo | `source` esperado |
|---|---|
| `STRANGLE_INVENTORY=false` | `"monolito"` |
| `STRANGLE_INVENTORY=true` | `"monolito (solo pago)"`, o `"microservicio-inventario (via fachada)"` si no hay stock |

## Revertir la extracción

Si el microservicio de inventario falla, basta con cambiar `STRANGLE_INVENTORY` a `false` y reiniciar la fachada: como el monolito aún conserva su lógica de inventario, las solicitudes a `/checkout` vuelven a resolverse ahí sin que quien consume la API note diferencia. Es más rápido que revertir un despliegue completo porque no se modifica código en ningún servicio, solo la configuración de la fachada.
