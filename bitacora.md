# Bitácora de Análisis - Café Origen v0.1

## 1. Diccionario de datos (Minirreto 1)

| Propiedad | Tipo de dato | Significado |
| :--- | :--- | :--- |
| `id` | `number` | Identificador único consecutivo de la orden de venta (del 1 al 12). |
| `fecha` | `string` | Fecha en la que se registró la venta en formato ISO (`YYYY-MM-DD`). |
| `cliente` | `string` | Código identificador del cliente que realizó la compra (ej. `"C01"`). |
| `producto` | `string` | Nombre del artículo o variedad de café vendida (ej. `"Origen Huila 500 g"`). |
| `ciudad` | `string` | Ciudad de destino o sucursal donde se realizó la venta (ej. `"Medellín"`). |
| `cantidad` | `number` | Número de unidades compradas del producto en esa transacción. |
| `precio` | `number` | Precio unitario del producto en pesos colombianos (COP). |

---

## 2. Casos borde detectados en los datos (Minirreto 1)

### Hallazgo 1:
- **Descripción**: En el registro con `id: 5`, la propiedad `ciudad` está escrita completamente en minúsculas (`"medellín"`), mientras que en otros registros (`id: 1, 4, 11`) aparece con mayúscula inicial (`"Medellín"`).
- **¿Cómo se debe tratar en el análisis?**: Se debe normalizar el texto aplicando `.trim().toLowerCase()` tanto al comparar como al agrupar por ciudad, evitando que `"Medellín"` y `"medellín"` se cuenten como ciudades distintas sin modificar el objeto original.

### Hallazgo 2:
- **Descripción**: El registro con `id: 8` (`"Origen Huila 500 g"` en Cali) tiene una `cantidad: 0` (orden anulada o sin unidades efectivas).
- **¿Cómo se debe tratar en el análisis?**: Se debe filtrar o ignorar todo registro cuya `cantidad <= 0` al calcular totales, unidades, búsqueda de extremos, agrupaciones y especialmente en el cálculo del promedio por venta (dividiendo solo entre las 11 ventas válidas y no entre los 12 registros totales).

---

## 3. Patrones con ciclos manuales (Minirreto 2)

- **`totalVendido(ventas)`**:
  - *Patrón usado (contador, acumulador, filtro, extremo)*: **Acumulador con filtro manual** (suma `valorVenta(venta)` únicamente cuando `venta.cantidad > 0`).
- **`totalUnidades(ventas)`**:
  - *Patrón usado*: **Acumulador con filtro manual** (suma `venta.cantidad` en cada iteración donde `venta.cantidad > 0`).
- **`ventaMayorValor(ventas)`**:
  - *Patrón usado*: **Búsqueda de extremo (máximo)** inicializando el candidato en `null` y comparando `valorVenta(venta) > valorVenta(mayor)` sobre las ventas válidas.
- **`ventasDeCiudad(ventas, ciudad)`**:
  - *Patrón usado*: **Filtro manual con normalización de texto** (construye un nuevo arreglo agregando con `.push()` las ventas con `cantidad > 0` cuya ciudad normalizada con `.trim().toLowerCase()` coincide con la ciudad buscada).
- **`promedioPorVenta(ventas)`**:
  - *Patrón usado*: **Acumulador + Contador combinados** (acumula el dinero de las ventas con `cantidad > 0`, cuenta cuántas ventas válidas hay sumando `+1`, y divide el acumulado entre el contador cuidando la división por cero).

---

## 4. Del ciclo al método: Preguntas de transformación y agrupación (Minirreto 3)

- **¿Qué filtro? (`.filter`)**: Se filtran los elementos del arreglo `ventas` conservando únicamente aquellos que cumplen la condición `venta.cantidad > 0` para descartar órdenes en cero.
- **¿Qué transformo? (`.map`)**: Cada objeto completo de venta válida se transforma 1 a 1 en un nuevo objeto resumido `{ id, producto, ciudad, total }`, normalizando `ciudad` en minúsculas y calculando `total` con `valorVenta(venta)`.
- **¿Qué reduzco / acumulo? (`.reduce`)**: Se reduce el arreglo de ventas a un único objeto acumulador (`{}`) que consolida las sumas de dinero total por ciudad, unidades totales por producto o recaudo total por producto.
- **¿Cómo agrupo en un diccionario?**: Dentro de `.reduce()` se inicia con un objeto vacío `{}` y por cada venta válida se toma la categoría como clave dinámica (`ciudadNormalizada` o `producto`), acumulando con la expresión `diccionario[clave] = (diccionario[clave] || 0) + valorASumar`.
- **¿Cómo ordeno sin modificar el arreglo original? (`.sort`)**: Creando un nuevo arreglo antes de invocar `.sort()` —por ejemplo, mediante copia superficial `[...arreglo].sort(...)` o aplicando `.sort()` sobre el nuevo arreglo de pares devuelto por `Object.entries(diccionario)`— seguido de `.slice(0, k)` y `.map()`.

---

## 5. Verificación del reporte y pureza de funciones (Minirreto 4)

- **¿Cómo comprobaste que el arreglo `ventas` original no fue modificado?**: Verificando al final de `src/reporte.js` que `ventas.length === 12` sigue siendo `true`, que el primer elemento conserva `ventas[0].id === 1`, y que registros como `id: 5` mantienen `ciudad: "medellín"` y `id: 8` mantiene `cantidad: 0` intactos.
- **Comando para ejecutar el reporte**: `node src/reporte.js`
.