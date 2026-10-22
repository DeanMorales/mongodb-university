# El Framework de Agregación en MongoDB

**Última actualización:** 22 de octubre de 2026

Este documento cubre el framework de agregación de MongoDB, una de las herramientas más poderosas para procesar y transformar datos. Se exploran las etapas principales del pipeline de agregación (`$set`, `$group`, `$unwind`, `$out` y `$lookup`) con definiciones detalladas, ejemplos prácticos y casos de uso reales.

## Índice

- [El Framework de Agregación](#el-framework-de-agregación)
- [Casos de Uso Reales](#casos-de-uso-reales)
  - [Ejemplo: Snake Game en MongoDB](#ejemplo-snake-game-en-mongodb)
  - [Ejemplo: Personal Budgeting Helper](#ejemplo-personal-budgeting-helper)
- [Etapas del Pipeline de Agregación](#etapas-del-pipeline-de-agregación)
  - [Etapa $set](#etapa-set)
  - [Etapa $group](#etapa-group)
  - [Etapa $unwind](#etapa-unwind)
  - [Etapa $out](#etapa-out)
  - [Etapa $lookup](#etapa-lookup)
- [Consideraciones de Rendimiento](#consideraciones-de-rendimiento)
- [Referencias y Fuentes](#referencias-y-fuentes)

## El Framework de Agregación

El **framework de agregación** de MongoDB es un mecanismo para procesar documentos y devolver resultados calculados. Funciona mediante la construcción de un **pipeline**, es decir, una secuencia de etapas donde cada etapa transforma los documentos que recibe y pasa el resultado a la siguiente etapa. Este modelo es análogo a una línea de ensamblaje: los datos entran en un extremo y salen transformados por el otro.

A diferencia de una consulta simple con `find()`, el pipeline de agregación permite realizar operaciones complejas como agrupaciones, proyecciones, cálculos matemáticos, uniones entre colecciones y escritura de resultados en una nueva colección. Actualmente MongoDB ofrece más de **60 etapas** de pipeline y más de **170 operadores**, lo que lo convierte en una herramienta extremadamente versátil para análisis de datos, generación de reportes y transformación de documentos directamente en la base de datos, sin necesidad de mover datos a una capa de aplicación.

```javascript
// Estructura básica de un pipeline de agregación
// Cada elemento del array es una etapa que transforma los documentos
db.nombreColeccion.aggregate([
  { $etapa1: { /* parámetros */ } },  // Primera transformación
  { $etapa2: { /* parámetros */ } },  // Segunda transformación
  { $etapa3: { /* parámetros */ } }   // Resultado final
])
```

## Casos de Uso Reales

### Ejemplo: Snake Game en MongoDB

Uno de los ejemplos más ilustrativos del poder del framework de agregación es una implementación del clásico juego Snake que se ejecuta íntegramente dentro de MongoDB, usando el pipeline de agregación para gestionar el estado del juego, calcular posiciones y resolver la lógica de movimiento. Este caso demuestra que el motor de agregación es suficientemente expresivo como para implementar lógica computacional compleja.

### Ejemplo: Personal Budgeting Helper

Otro caso de uso práctico es una herramienta de gestión de presupuesto personal, donde se utiliza un pipeline de agregación para categorizar transacciones, calcular totales por categoría y generar reportes de gastos. Este ejemplo ilustra cómo la agregación sirve directamente para resolver problemas de negocio en el mundo real.

**Ejemplo de reporte de actividad de usuarios:**

Un caso típico en aplicaciones web es analizar la frecuencia de visitas de usuarios a una página en un período de tiempo determinado. El pipeline permite agrupar eventos por usuario y contar cuántas veces recargaron la página dentro de una ventana de una hora, clasificándolos en rangos:

- 3 veces por hora
- 5 veces por hora
- 10 o más veces por hora

Este tipo de análisis es útil para detectar patrones de uso, identificar usuarios activos o detectar comportamientos anómalos.

## Etapas del Pipeline de Agregación

### Etapa $set

La etapa **`$set`** permite agregar nuevos campos o sobrescribir campos existentes en los documentos que pasan por el pipeline. Es importante entender que `$set` opera **únicamente dentro del pipeline**: no modifica los documentos almacenados en disco. Los cambios que realiza son temporales y solo existen durante la ejecución del pipeline, a menos que se combine con una etapa `$out` o `$merge` para persistir los resultados.

`$set` es un alias más legible de la etapa `$addFields` y es especialmente útil para calcular campos derivados, como un precio con descuento, una edad calculada a partir de una fecha de nacimiento, o la concatenación de campos de texto.

```javascript
// $set agrega o modifica campos temporalmente dentro del pipeline
// No persiste cambios en los documentos originales de la colección
db.productos.aggregate([
  {
    $set: {
      // Calcula el precio final aplicando un 10% de descuento
      precioConDescuento: { $multiply: ["$precio", 0.90] },
      // Agrega una etiqueta de disponibilidad basada en el stock
      disponibilidad: {
        $cond: { if: { $gt: ["$stock", 0] }, then: "En stock", else: "Agotado" }
      }
    }
  }
])
```

### Etapa $group

La etapa **`$group`** agrupa documentos por una clave especificada y permite calcular valores acumulados para cada grupo, como sumas, promedios, conteos o valores máximos y mínimos. En términos de bases de datos relacionales, `$group` es conceptualmente equivalente a la cláusula `GROUP BY` de SQL.

El campo `_id` dentro de `$group` define la clave de agrupación. Si se establece en `null`, todos los documentos se agrupan en uno solo, lo que es útil para calcular totales globales. Cada campo adicional en la etapa define una acumulación utilizando operadores como `$sum`, `$avg`, `$max`, `$min`, `$push` o `$addToSet`.

```javascript
// $group agrupa documentos por una clave y calcula acumulaciones
db.ventas.aggregate([
  {
    $group: {
      // Agrupa por la categoría del producto
      _id: "$categoria",
      // Suma el total de ventas por categoría
      totalVentas: { $sum: "$monto" },
      // Cuenta cuántas transacciones hubo por categoría
      cantidadTransacciones: { $sum: 1 },
      // Calcula el promedio de venta por categoría
      promedioVenta: { $avg: "$monto" }
    }
  }
])
```

### Etapa $unwind

La etapa **`$unwind`** descompone un campo de tipo array dentro de un documento, generando un documento separado por cada elemento del array. Es decir, si un documento tiene un campo con un array de 5 elementos, `$unwind` produce 5 documentos independientes, uno por cada elemento, manteniendo el resto de los campos del documento original.

Esta etapa es fundamental cuando se necesita aplicar filtros, agrupaciones u otras transformaciones sobre los elementos individuales de un array, operaciones que no serían posibles directamente sobre el array completo. Sin embargo, es importante considerar que `$unwind` puede generar un gran volumen de documentos intermedios si los arrays son extensos, lo que incrementa el consumo de memoria y procesamiento del cluster. Por eso, se recomienda usarla solo cuando es necesaria y, de ser posible, combinarla con etapas de filtrado previas (`$match`) para reducir el conjunto de documentos antes de expandirlos.

```javascript
// Ejemplo: colección de órdenes donde cada orden tiene múltiples items
// Documento original:
// { _id: 1, cliente: "Ana", items: ["libro", "lapiz", "cuaderno"] }

db.ordenes.aggregate([
  {
    // $unwind expande el array "items":
    // Genera un documento por cada elemento del array
    $unwind: "$items"
  }
  // Resultado:
  // { _id: 1, cliente: "Ana", items: "libro" }
  // { _id: 1, cliente: "Ana", items: "lapiz" }
  // { _id: 1, cliente: "Ana", items: "cuaderno" }
])

// Ejemplo práctico: calcular el total de unidades vendidas por producto
db.ordenes.aggregate([
  { $unwind: "$items" },              // Expande el array de items
  { $group: {                         // Agrupa por producto
      _id: "$items.producto",
      totalVendido: { $sum: "$items.cantidad" }
    }
  }
])
```

### Etapa $out

La etapa **`$out`** toma todos los documentos resultantes del pipeline y los escribe en una colección de destino dentro de la misma base de datos (o en otra base de datos a partir de MongoDB 4.4). Es la etapa más directa para persistir los resultados de un pipeline. Debe ser siempre la **última etapa** del pipeline, ya que no pasa documentos a etapas posteriores.

Un aspecto crítico de `$out` es que **reemplaza completamente** la colección destino si ya existe. Esto significa que todos los documentos previos en esa colección serán eliminados y reemplazados por los resultados del pipeline. Esta característica la hace ideal para generar colecciones de reportes precalculados que se actualizan periódicamente, pero requiere cuidado para no sobreescribir datos importantes por accidente.

Para conjuntos de datos muy grandes, la operación puede requerir espacio adicional en disco, lo cual se habilita con la opción `{ allowDiskUse: true }`.

```javascript
// $out escribe los resultados en una nueva colección
// IMPORTANTE: debe ser siempre la última etapa del pipeline
db.ventas.aggregate([
  {
    // Agrupa las ventas por mes y calcula el total
    $group: {
      _id: { mes: { $month: "$fecha" }, año: { $year: "$fecha" } },
      totalMensual: { $sum: "$monto" }
    }
  },
  {
    // Escribe los resultados en la colección "reporte_ventas_mensuales"
    // Si la colección existe, la reemplaza completamente
    $out: "reporte_ventas_mensuales"
  }
])

// Para operaciones que requieren más memoria que la disponible en RAM
db.coleccionGrande.aggregate(
  [ /* etapas del pipeline */ ],
  { allowDiskUse: true }  // Permite usar disco cuando la RAM es insuficiente
)
```

### Etapa $lookup

La etapa **`$lookup`** realiza una unión (join) entre la colección actual y otra colección de la misma base de datos, similar conceptualmente al `LEFT OUTER JOIN` de SQL. Permite enriquecer documentos con información proveniente de otras colecciones sin necesidad de múltiples consultas desde la capa de aplicación.

`$lookup` es computacionalmente costosa, ya que implica combinar datos de dos colecciones. Por eso, se recomienda aplicarla después de etapas de filtrado (`$match`) para reducir el número de documentos sobre los que se realiza la unión. Además, el pipeline de agregación tiene un **límite máximo de 1000 etapas** por ejecución, lo que en la práctica raramente se alcanza pero es relevante para pipelines muy complejos.

```javascript
// $lookup une la colección actual con otra colección
db.ordenes.aggregate([
  {
    $lookup: {
      from: "clientes",           // Colección de la que se traen los datos
      localField: "clienteId",    // Campo en la colección actual (ordenes)
      foreignField: "_id",        // Campo en la colección foránea (clientes)
      as: "datosCliente"          // Nombre del array resultante con los datos unidos
    }
  }
  // Resultado: cada orden ahora incluye el array "datosCliente"
  // con el documento completo del cliente correspondiente
])

// Ejemplo con $match previo para optimizar el rendimiento
db.ordenes.aggregate([
  {
    // Filtra primero para reducir documentos antes del $lookup costoso
    $match: { estado: "completada", año: 2026 }
  },
  {
    // Luego realiza la unión solo sobre los documentos filtrados
    $lookup: {
      from: "productos",
      localField: "productoId",
      foreignField: "_id",
      as: "detalleProducto"
    }
  }
])
```

## Consideraciones de Rendimiento

Al trabajar con el framework de agregación, es importante tener en cuenta las siguientes recomendaciones:

- **Filtrar temprano**: Usar `$match` al inicio del pipeline para reducir el volumen de documentos antes de etapas costosas como `$lookup` o `$unwind`.
- **Limitar el uso de `$unwind`**: Genera un alto volumen de documentos intermedios. Combinarla siempre con filtros previos cuando sea posible.
- **`$out` reemplaza colecciones**: Verificar que la colección destino no contenga datos importantes antes de usar esta etapa.
- **`allowDiskUse`**: Necesario para pipelines que procesan grandes volúmenes de datos que superan el límite de memoria RAM disponible.
- **Límite de etapas**: El pipeline admite un máximo de 1000 etapas por ejecución.
- **Índices**: Los índices en los campos usados por `$match`, `$sort` y `$lookup` mejoran significativamente el rendimiento del pipeline.

## Referencias y Fuentes

- [MongoDB Documentation - Aggregation Pipeline](https://www.mongodb.com/docs/manual/core/aggregation-pipeline/) - Documentación oficial sobre el pipeline de agregación y sus etapas
- [MongoDB Documentation - $set](https://www.mongodb.com/docs/manual/reference/operator/aggregation/set/) - Referencia oficial de la etapa `$set`
- [MongoDB Documentation - $group](https://www.mongodb.com/docs/manual/reference/operator/aggregation/group/) - Referencia oficial de la etapa `$group` con todos sus acumuladores
- [MongoDB Documentation - $unwind](https://www.mongodb.com/docs/manual/reference/operator/aggregation/unwind/) - Referencia oficial de la etapa `$unwind` y sus opciones
- [MongoDB Documentation - $out](https://www.mongodb.com/docs/manual/reference/operator/aggregation/out/) - Referencia oficial de la etapa `$out`
- [MongoDB Documentation - $lookup](https://www.mongodb.com/docs/manual/reference/operator/aggregation/lookup/) - Referencia oficial de la etapa `$lookup` con ejemplos de joins
- [MongoDB University - Aggregation Course](https://university.mongodb.com/) - Cursos oficiales de MongoDB sobre agregación y modelado de datos
- [MongoDB - Aggregation Pipeline Optimization](https://www.mongodb.com/docs/manual/core/aggregation-pipeline-optimization/) - Guía de optimización del rendimiento en pipelines de agregación
