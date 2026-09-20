# MongoDB y el Modelo de Documento

**Última actualización:** 19 de septiembre de 2026

Este documento cubre los conceptos fundamentales del modelo de documento de MongoDB, incluyendo el formato BSON, tipos de datos, relaciones entre entidades y estrategias de modelado de datos usando embedding y referencias. Abarca desde la estructura básica de documentos hasta patrones avanzados para gestionar relaciones complejas en bases de datos no relacionales.

## Índice

- [Conceptos Fundamentales](#conceptos-fundamentales)
  - [Documento](#documento)
  - [Formato BSON vs JSON](#formato-bson-vs-json)
  - [Tipos de Datos BSON](#tipos-de-datos-bson)
- [Gestión de Bases de Datos, Colecciones y Documentos en Atlas](#gestión-de-bases-de-datos-colecciones-y-documentos-en-atlas)
- [Relaciones entre Datos](#relaciones-entre-datos)
  - [Entidades](#entidades)
  - [Atributos](#atributos)
  - [Tipos de Relaciones](#tipos-de-relaciones)
- [Embedding y Referencias](#embedding-y-referencias)
  - [Embedding en MongoDB](#embedding-en-mongodb)
  - [Referencias en MongoDB](#referencias-en-mongodb)
  - [Relaciones Uno a Uno](#relaciones-uno-a-uno)
  - [Relaciones Uno a Muchos](#relaciones-uno-a-muchos)
  - [Relaciones Muchos a Muchos](#relaciones-muchos-a-muchos)
- [Referencias y Fuentes](#referencias-y-fuentes)

## Conceptos Fundamentales

### Documento

Un documento en MongoDB es una colección de objetos e información estructurada en formato clave-valor, muy similar a JSON. Este formato específico de MongoDB se denomina **BSON (Binary JSON)**, una representación binaria de JSON que optimiza el almacenamiento y la transmisión de datos. Los documentos constituyen la unidad fundamental de almacenamiento en MongoDB, permitiendo una estructura flexible y jerárquica que se adapta naturalmente a cómo los desarrolladores modelan sus datos en aplicaciones modernas.

**Características importantes de los documentos MongoDB:**

- **Límite de tamaño**: Los documentos tienen un tamaño máximo de 16 MB. Este límite está diseñado para garantizar un rendimiento óptimo y una distribución eficiente de datos en sistemas distribuidos. Si necesitas almacenar información más grande, MongoDB proporciona GridFS para manejar archivos binarios.

- **Pares clave-valor (Field-Value Pairs)**: Cada documento consiste en pares de clave-valor donde la clave es una cadena que identifica de manera única el campo dentro del documento, y el valor puede ser cualquier tipo de dato soportado por BSON.

- **Esquema flexible**: A diferencia de las bases de datos relacionales con esquemas rígidos, MongoDB permite que cada documento en una colección tenga su propia estructura única. Esto significa que documentos diferentes pueden tener campos completamente distintos, lo que facilita la evolución del modelo de datos sin necesidad de migraciones complejas.

- **Límite de anidación**: MongoDB soporta un máximo de 100 niveles de documentos anidados. Este límite existe para prevenir problemas de rendimiento y mantener la eficiencia en las operaciones de consulta. En la práctica, la mayoría de aplicaciones rara vez necesita más de 5 o 10 niveles de anidación.

### Formato BSON vs JSON

**JSON (JavaScript Object Notation)** es un formato de texto ligero y legible para humanos que se utiliza comúnmente para el intercambio de datos en aplicaciones web. JSON admite tipos de datos básicos como strings, números, booleanos, arrays y objetos anidados.

**BSON (Binary JSON)** es la extensión binaria de JSON que MongoDB utiliza para serializar y deserializar documentos. BSON proporciona ventajas significativas sobre JSON puro: es más compacto en términos de almacenamiento, más rápido de procesar, y soporta tipos de datos adicionales que JSON no incluye de forma nativa, como fechas, números de precisión específica (Int32, Int64, Double), identificadores únicos (ObjectId), expresiones regulares y datos binarios. Esta capacidad de representar tipos de datos más ricos hace que BSON sea ideal para aplicaciones que requieren precisión en el manejo de datos numéricos y temporales.

```json
// Ejemplo de JSON (formato de texto)
{
  "nombre": "Juan",
  "edad": 30,
  "activo": true
}

// Ejemplo de BSON (formato binario, mostrado como JSON para legibilidad)
// BSON agrega información de tipo adicional que se almacena internamente
{
  "nombre": "Juan",           // String en BSON
  "edad": 30,                 // Int32 o Int64 en BSON
  "activo": true,             // Boolean en BSON
  "fechaCreacion": ISODate("2026-09-19T10:30:00Z")  // Date en BSON
}
```

### Tipos de Datos BSON

Los documentos BSON aceptan diversos tipos de valores, cada uno diseñado para representar diferentes clases de información:

- **Booleanos**: Valores que pueden ser `true` o `false`. Se utilizan para representar condiciones binarias, estados activados/desactivados, o banderas lógicas.

- **Null**: Representa la ausencia de valor o valor indeterminado. Útil para indicar que un campo no tiene información disponible en ese momento.

- **Strings (Cadenas de caracteres)**: Secuencias de texto codificadas en UTF-8. MongoDB acepta tanto comillas dobles (`"`) como comillas simples (`'`) para definir strings, aunque la práctica recomendada es usar comillas dobles por consistencia.

- **Números (Number)**: MongoDB diferencia entre varios tipos numéricos:
  - **Int32**: Enteros de 32 bits, rango de -2,147,483,648 a 2,147,483,647
  - **Int64**: Enteros de 64 bits para números muy grandes
  - **Double**: Números de punto flotante con precisión de 64 bits para valores decimales
  - Esta diferenciación permite optimizar el almacenamiento según las necesidades de precisión

- **Objetos (Objects)**: Estructuras que contienen pares de clave-valor relacionados, representados con llaves `{}`. Los objetos pueden estar anidados, permitiendo crear estructuras jerárquicas complejas. Por ejemplo, un documento de usuario podría contener un objeto "dirección" con campos como calle, ciudad y código postal.

- **Arrays**: Colecciones de valores representadas con corchetes `[]`. Los arrays en MongoDB pueden contener cualquier tipo de dato BSON, incluyendo otros arrays u objetos. Son especialmente útiles para representar listas de elementos, como tags de una publicación, comentarios en un artículo, o ítems en un carrito de compras.

- **Dates (Fechas)**: Representan momentos específicos en el tiempo, almacenadas como milisegundos desde el 1 de enero de 1970 (epoch). MongoDB maneja automáticamente las conversiones de zona horaria y proporciona funciones para manipular fechas de manera eficiente. Las fechas son cruciales para registrar cuándo se crearon o modificaron documentos.

- **ObjectIds**: Un tipo de dato especial único para MongoDB, generado automáticamente como identificador principal de cada documento. Los ObjectIds tienen 12 bytes: 4 bytes de timestamp, 5 bytes de identificador de máquina, 3 bytes de contador de proceso, y 3 bytes de contador incremental. Esta estructura garantiza que ObjectIds sean globalmente únicos y contengan información temporal, permitiendo a MongoDB generar identificadores sin coordinar con una autoridad central. ObjectIds son más eficientes que strings UUID para su uso en MongoDB.

```javascript
// Ejemplo de documento BSON con múltiples tipos de datos
{
  "_id": ObjectId("507f1f77bcf86cd799439011"),      // ObjectId - identificador único
  "nombre": "María García",                          // String
  "edad": 28,                                        // Int32
  "sueldo": 45000.50,                               // Double
  "activo": true,                                    // Boolean
  "fechaContratacion": ISODate("2023-06-15T09:00:00Z"), // Date
  "departamento": "Ingeniería",                      // String
  "telefonos": ["+34 912 345 678", "+34 654 321 987"], // Array de Strings
  "direccion": {                                     // Object anidado
    "calle": "Calle Principal 42",
    "ciudad": "Madrid",
    "codigoPostal": "28001",
    "pais": "España"
  },
  "etiquetas": ["senior", "backend", "python"],      // Array de Strings
  "notas": null,                                      // Null
  "ultimaModificacion": ISODate("2026-09-19T14:30:00Z")
}
```

## Gestión de Bases de Datos, Colecciones y Documentos en Atlas

En esta sección, exploramos cómo utilizar la interfaz de MongoDB Atlas para gestionar bases de datos, colecciones y documentos. Atlas proporciona una interfaz gráfica intuitiva que facilita las operaciones de administración y desarrollo sin necesidad de usar herramientas de línea de comandos. A través de la consola web de Atlas, puedes navegar por la estructura de tu base de datos, visualizar documentos, ejecutar consultas, crear índices y realizar todas las operaciones de base de datos esenciales.

### Editor de Documentos en Atlas

La interfaz de Atlas incluye un editor visual que permite:
- Crear nuevos documentos manualmente
- Editar documentos existentes de manera interactiva
- Visualizar la estructura jerárquica de documentos complejos
- Insertar datos de prueba para desarrollo y testing
- Validar la estructura de documentos antes de guardar

## Relaciones entre Datos

Entender y modelar correctamente las relaciones entre datos es fundamental para diseñar bases de datos eficientes. En MongoDB, a diferencia de las bases de datos relacionales tradicionales que utilizan joins entre tablas, el modelado de relaciones se aborda principalmente a través de dos estrategias: embedding (incrustar documentos) y referencias.

### Entidades

Una **entidad** en el contexto de modelado de datos es un objeto del mundo real que tiene características distinguibles y existencia independiente. En el contexto de MongoDB, una entidad típicamente se representa como un documento completo dentro de una colección. Por ejemplo, en una aplicación de comercio electrónico, las entidades podrían ser: usuarios, productos, órdenes de compra y reseñas. Cada entidad debe tener una identidad única, representada típicamente por un campo `_id` que MongoDB genera automáticamente. Las entidades son los bloques fundamentales sobre los cuales se construye el modelo de datos, y sus relaciones determinan cómo se estructura la información.

### Atributos

Los **atributos** son características o propiedades específicas que describen a una entidad. Cada atributo captura una pieza particular de información sobre la entidad. Por ejemplo, para la entidad "Usuario", los atributos podrían incluir: nombre, email, fecha de registro, dirección, teléfono, estado de verificación, y preferencias de notificación. En MongoDB, los atributos se representan como campos (keys) dentro de un documento. Los atributos pueden tener diferentes tipos de datos (strings, números, fechas, booleanos, etc.) y pueden ser opcionales debido al esquema flexible de MongoDB. La selección cuidadosa de atributos es crucial para capturar la información necesaria sin sobrecargar el modelo con datos irrelevantes.

### Tipos de Relaciones

Las relaciones entre entidades describen cómo interactúan y se conectan diferentes entidades en tu sistema. MongoDB soporta principalmente tres patrones de relación:

#### Relación Uno a Uno

Una relación **uno a uno (one-to-one)** existe cuando una entidad del tipo A está asociada con exactamente una entidad del tipo B, y viceversa. Por ejemplo, en una aplicación de usuarios, cada usuario tiene exactamente un perfil personal, y cada perfil pertenece a exactamente un usuario. En MongoDB, las relaciones uno a uno generalmente se modelan usando **embedding**, donde el documento del perfil se incluye directamente dentro del documento del usuario, creando una estructura única y cohesiva. Esta aproximación es eficiente porque los datos relacionados se almacenan juntos, reduciendo la necesidad de consultas múltiples.

```javascript
// Ejemplo de relación uno a uno con embedding
// Documento de usuario con perfil embebido
{
  "_id": ObjectId("507f1f77bcf86cd799439011"),
  "nombre": "Carlos López",
  "email": "carlos@example.com",
  // El perfil está embebido dentro del documento del usuario
  "perfil": {
    "bio": "Desarrollador apasionado por MongoDB",
    "avatar": "https://example.com/avatars/carlos.jpg",
    "website": "https://carloslopez.dev",
    "fechaCreacionPerfil": ISODate("2023-01-15T10:30:00Z")
  }
}
```

#### Relación Uno a Muchos

Una relación **uno a muchos (one-to-many)** ocurre cuando una entidad del tipo A se relaciona con múltiples entidades del tipo B, pero cada entidad del tipo B está asociada con solo una entidad del tipo A. Por ejemplo, un usuario puede tener múltiples direcciones de envío, pero cada dirección pertenece a un único usuario. En MongoDB, las relaciones uno a muchos pueden modelarse de dos maneras:

1. **Con embedding (para pocos documentos relacionados)**: Incluir un array de objetos embebidos dentro del documento principal. Esta aproximación es eficiente cuando el número de documentos relacionados es pequeño y relativamente estable.

2. **Con referencias (para muchos documentos relacionados)**: Almacenar referencias (ObjectIds) al documento relacionado. Esta aproximación es preferible cuando el número de documentos relacionados es grande o variable, para evitar que el documento padre exceda el límite de 16 MB.

```javascript
// Opción 1: Relación uno a muchos con embedding (para pocos elementos)
// Documento de usuario con direcciones embebidas
{
  "_id": ObjectId("507f1f77bcf86cd799439011"),
  "nombre": "María García",
  "email": "maria@example.com",
  // Array de direcciones embebido en el documento
  "direcciones": [
    {
      "tipo": "residencia",
      "calle": "Avenida Principal 100",
      "ciudad": "Barcelona",
      "codigoPostal": "08002",
      "predeterminada": true
    },
    {
      "tipo": "trabajo",
      "calle": "Carrera 50 #10-20",
      "ciudad": "Madrid",
      "codigoPostal": "28001",
      "predeterminada": false
    }
  ]
}

// Opción 2: Relación uno a muchos con referencias (para muchos elementos)
// Colección usuarios
{
  "_id": ObjectId("507f1f77bcf86cd799439011"),
  "nombre": "Juan Pérez",
  "email": "juan@example.com"
}

// Colección direcciones
{
  "_id": ObjectId("507f1f77bcf86cd799439012"),
  "usuarioId": ObjectId("507f1f77bcf86cd799439011"),  // Referencia al usuario
  "tipo": "residencia",
  "calle": "Plaza Mayor 5",
  "ciudad": "Valencia",
  "codigoPostal": "46001"
}
```

#### Relación Muchos a Muchos

Una relación **muchos a muchos (many-to-many)** ocurre cuando múltiples entidades del tipo A se relacionan con múltiples entidades del tipo B. Por ejemplo, en una aplicación de películas, muchas películas pueden tener múltiples actores, y cada actor puede aparecer en múltiples películas. En MongoDB, las relaciones muchos a muchos se modelan típicamente usando **referencias**. 

Existen dos enfoques comunes:

1. **Arrays de referencias en el lado de la relación más consultada**: Almacenar un array de referencias ObjectIds. Por ejemplo, si consultas frecuentemente "¿qué actores están en esta película?", almacenarías un array de actorIds en el documento de la película.

2. **Documentos de unión (lookup collections)**: Para relaciones muy complejas, crear una colección separada que represente la relación en sí, útil cuando la relación tiene atributos adicionales (como la fecha en que un actor fue contratado para una película).

```javascript
// Opción 1: Relación muchos a muchos con arrays de referencias
// Colección películas
{
  "_id": ObjectId("507f1f77bcf86cd799439011"),
  "titulo": "Inception",
  "año": 2010,
  "director": "Christopher Nolan",
  // Array de referencias a actores
  "actorIds": [
    ObjectId("507f1f77bcf86cd799439020"),
    ObjectId("507f1f77bcf86cd799439021"),
    ObjectId("507f1f77bcf86cd799439022")
  ]
}

// Colección actores
{
  "_id": ObjectId("507f1f77bcf86cd799439020"),
  "nombre": "Leonardo DiCaprio",
  "fechaNacimiento": ISODate("1974-11-11T00:00:00Z"),
  "nacionalidad": "Estadounidense"
}

// Opción 2: Usando una colección de unión para atributos de relación
// Colección películas_actores (junction collection)
{
  "_id": ObjectId("507f1f77bcf86cd799439030"),
  "peliculaId": ObjectId("507f1f77bcf86cd799439011"),
  "actorId": ObjectId("507f1f77bcf86cd799439020"),
  "personaje": "Cobb",
  "rol": "protagonista",
  "fechaContratacion": ISODate("2009-06-01T00:00:00Z")
}
```

## Embedding y Referencias

### Embedding en MongoDB

El **embedding** es la práctica de incrustar documentos relacionados directamente dentro de otro documento. Cuando utilizas embedding, toda la información relacionada se almacena en un único documento, creando una estructura jerárquica. Las ventajas de embedding incluyen:

- **Rendimiento mejorado**: Las lecturas son más rápidas porque toda la información se encuentra en un solo lugar, eliminando la necesidad de consultas adicionales (joins).
- **Transacciones atómicas**: Modificar un documento embebido y su padre es una operación atómica, garantizando consistencia.
- **Simplicidad**: El código es más directo porque no necesitas combinar múltiples documentos.

Las desventajas incluyen:

- **Límite de tamaño de documento**: Si hay muchos documentos relacionados, podrías superar el límite de 16 MB.
- **Flexibilidad de actualización**: Si necesitas actualizar solo documentos embebidos sin tocar el padre, debes usar operadores especiales.
- **Duplicación de datos**: Si los datos embebidos se necesitan en múltiples lugares, resultará en duplicación.

```javascript
// Ejemplo: Embedding de comentarios en un artículo de blog
{
  "_id": ObjectId("507f1f77bcf86cd799439011"),
  "titulo": "Introducción a MongoDB",
  "autor": "Software Engineer",
  "contenido": "MongoDB es una base de datos...",
  "fechaPublicacion": ISODate("2026-09-19T10:00:00Z"),
  // Los comentarios están embebidos directamente
  "comentarios": [
    {
      "_id": ObjectId("507f1f77bcf86cd799439012"),
      "usuario": "usuario1",
      "texto": "Excelente artículo, muy instructivo",
      "fecha": ISODate("2026-09-19T12:30:00Z"),
      "likes": 5
    },
    {
      "_id": ObjectId("507f1f77bcf86cd799439013"),
      "usuario": "usuario2",
      "texto": "Gracias por la explicación clara",
      "fecha": ISODate("2026-09-19T14:15:00Z"),
      "likes": 3
    }
  ]
}
```

### Referencias en MongoDB

Las **referencias** son conexiones entre documentos usando ObjectIds, similar a las claves foráneas en bases de datos relacionales. Con referencias, documentos relacionados se almacenan en colecciones separadas, y se conectan mediante identificadores. Las ventajas incluyen:

- **Flexibilidad**: Cada colección puede evolucionar de manera independiente.
- **Gestión de tamaño**: No hay riesgo de exceder el límite de 16 MB en un documento.
- **Reutilización de datos**: Un documento puede ser referenciado desde múltiples lugares sin duplicación.
- **Actualización centralizada**: Cambiar un documento se refleja automáticamente en todas las referencias.

Las desventajas incluyen:

- **Rendimiento**: Requiere consultas adicionales o lookups para obtener datos relacionados.
- **Más complejo**: El código necesita manejar múltiples operaciones y posibles referencias faltantes.
- **Transacciones**: Actualizar múltiples colecciones requiere manejo cuidadoso para garantizar consistencia.

```javascript
// Ejemplo: Referencias entre colecciones

// Colección autores
{
  "_id": ObjectId("507f1f77bcf86cd799439011"),
  "nombre": "Jane Austen",
  "pais": "Reino Unido",
  "fechaNacimiento": ISODate("1775-12-16T00:00:00Z")
}

// Colección libros con referencia al autor
{
  "_id": ObjectId("507f1f77bcf86cd799439020"),
  "titulo": "Orgullo y Prejuicio",
  "autorId": ObjectId("507f1f77bcf86cd799439011"),  // Referencia al autor
  "año": 1813,
  "genero": "Romance",
  "paginas": 432
}

// Para obtener el autor de un libro, necesitas realizar una consulta adicional
// db.libros.findOne({_id: ...}) -> obtiene el libro con autorId
// db.autores.findOne({_id: ObjectId("507f1f77bcf86cd799439011")}) -> obtiene el autor
```

### Relaciones Uno a Uno

Las relaciones **uno a uno** entre entidades generalmente se modelan usando **embedding** en MongoDB. La razón es que embedding es más eficiente: almacena ambas entidades juntas, permitiendo lecturas rápidas y operaciones atómicas. Usar referencias para relaciones uno a uno sería innecesariamente complejo y resultaría en un rendimiento inferior.

```javascript
// Recomendación: Usar embedding para uno a uno
// Documento de empresa con dirección embebida
{
  "_id": ObjectId("507f1f77bcf86cd799439011"),
  "nombre": "TechCorp Inc.",
  "industria": "Tecnología",
  "empleados": 500,
  "direccion": {  // Embebido: relación uno a uno
    "calle": "Tech Avenue 123",
    "ciudad": "San Francisco",
    "estado": "California",
    "codigoPostal": "94105",
    "pais": "Estados Unidos"
  },
  "sitioWeb": "https://techcorp.com"
}
```

### Relaciones Uno a Muchos

Las relaciones **uno a muchos** en MongoDB pueden modelarse de dos formas, dependiendo de la cantidad de documentos relacionados y los patrones de acceso:

**Embedding (para pocos elementos, típicamente menos de 100-1000):**
- Ventajas: Rendimiento rápido, operaciones atómicas
- Desventajas: Documento puede crecer sin límite

**Referencias (para muchos elementos o datos que cambian frecuentemente):**
- Ventajas: Flexibilidad, sin límite de tamaño
- Desventajas: Requiere consultas adicionales

```javascript
// Ejemplo práctico: Tienda online
// Usar embedding para opiniones de productos (generalmente pocos)
{
  "_id": ObjectId("507f1f77bcf86cd799439011"),
  "nombre": "Laptop Gaming",
  "precio": 1299.99,
  "stock": 50,
  "resenas": [  // Embebidas si hay pocas
    {
      "usuarioId": ObjectId("507f1f77bcf86cd799439020"),
      "calificacion": 5,
      "comentario": "Excelente producto",
      "fecha": ISODate("2026-09-19T10:00:00Z")
    }
  ]
}

// Usar referencias para órdenes de un cliente (pueden ser muchas)
// Colección clientes
{
  "_id": ObjectId("507f1f77bcf86cd799439011"),
  "nombre": "Carlos López",
  "email": "carlos@example.com"
}

// Colección ordenes
{
  "_id": ObjectId("507f1f77bcf86cd799439030"),
  "clienteId": ObjectId("507f1f77bcf86cd799439011"),  // Referencia al cliente
  "fecha": ISODate("2026-09-19T14:30:00Z"),
  "total": 1299.99
}
```

### Relaciones Muchos a Muchos

Las relaciones **muchos a muchos** se resuelven más efectivamente usando **referencias** en MongoDB. La estrategia típica es almacenar arrays de referencias ObjectIds en uno o ambos lados de la relación, dependiendo de los patrones de consulta más comunes.

Un ejemplo práctico es la relación entre películas y cines, donde muchas películas se proyectan en muchos cines, y cada cine proyecta muchas películas. Puedes modelar esto almacenando un array de cineIds en cada película, o un array de peliculaIds en cada cine, o incluso ambos si realizas consultas frecuentemente en ambas direcciones.

```javascript
// Ejemplo: Relación muchos a muchos entre películas y cines

// Colección películas
{
  "_id": ObjectId("507f1f77bcf86cd799439011"),
  "titulo": "Oppenheimer",
  "director": "Christopher Nolan",
  "año": 2023,
  // Array de cines donde se proyecta la película
  "cineIds": [
    ObjectId("507f1f77bcf86cd799439050"),
    ObjectId("507f1f77bcf86cd799439051"),
    ObjectId("507f1f77bcf86cd799439052")
  ]
}

// Colección cines
{
  "_id": ObjectId("507f1f77bcf86cd799439050"),
  "nombre": "Cine Central",
  "ciudad": "Madrid",
  "salas": 10,
  // Array de películas que se proyectan en este cine
  "peliculaIds": [
    ObjectId("507f1f77bcf86cd799439011"),
    ObjectId("507f1f77bcf86cd799439012"),
    ObjectId("507f1f77bcf86cd799439013")
  ]
}

// Para consultas complejas donde necesitas información de ambos lados,
// considera una colección de unión adicional:
// Colección proyecciones (relación explícita)
{
  "_id": ObjectId("507f1f77bcf86cd799439060"),
  "peliculaId": ObjectId("507f1f77bcf86cd799439011"),
  "cineId": ObjectId("507f1f77bcf86cd799439050"),
  "horarios": ["14:00", "17:30", "20:00", "22:30"],
  "formato": "IMAX 3D",
  "precioEntrada": 12.50,
  "asientosDisponibles": 45
}
```

## Referencias y Fuentes

- [MongoDB Documentation - Documents](https://www.mongodb.com/docs/manual/documents/) - Documentación oficial sobre la estructura y características de documentos en MongoDB
- [MongoDB Documentation - BSON Types](https://www.mongodb.com/docs/manual/reference/bson-types/) - Referencia completa de tipos de datos BSON soportados
- [MongoDB Documentation - Data Modeling](https://www.mongodb.com/docs/manual/data-modeling/) - Guía oficial de estrategias de modelado de datos en MongoDB
- [MongoDB Documentation - Schema Validation](https://www.mongodb.com/docs/manual/core/schema-validation/) - Documentación sobre validación de esquemas en MongoDB
- [MongoDB University - Data Modeling Patterns](https://university.mongodb.com/) - Cursos y materiales educativos sobre patrones de modelado
- [MongoDB Blog - Flexible Data Modeling](https://www.mongodb.com/developer/article/data-modeling-strategies/) - Artículos técnicos sobre estrategias de modelado flexible

